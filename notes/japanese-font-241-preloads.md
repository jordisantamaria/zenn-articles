# notes — japanese-font-241-preloads

Artículo 3 de 3 sobre la optimización de `oshisuki.com`. Los otros dos son
`serverless-to-server.md` (dónde corre el servidor) y `landing-off-the-server.md`
(qué no debe depender de él). Éste va de **por qué la página pesaba lo que pesaba**.

Probablemente el más vendible de los tres: es un problema que tiene **todo dev japonés
que use `next/font`** y casi ninguno sabe que lo tiene.

Todo medido el 2026-08-22/23 sobre `oshisuki.com` en producción.

## ⚑ Para qué existe este artículo

**Objetivo: autoridad técnica que acabe en encargos de optimización.** Ver la cabecera
de `notes/serverless-to-server.md` — las seis reglas son las mismas. Método por encima
del resultado, los fallos contados, números medidos o nada, antes/después en el mismo
entorno, decir lo que no se puede arreglar, y sin CTA.

Éste tiene una ventaja sobre los otros dos: **el lector puede comprobarlo en su propia
web en treinta segundos** (contar los `<link rel="preload" as="font">` de su HTML). Un
artículo que hace que alguien vaya a mirar su propio sitio y encuentre el problema
genera más confianza que cualquier caso de estudio.

## El hallazgo

`next/font` parte una fuente japonesa en cientos de subsets por rango unicode:

| Familia | `@font-face` generados |
|---|---|
| Zen Maru Gothic | **366** |
| Noto Sans JP | **373** |
| Archivo + Figtree (latín) | 24 |

Con `preload` activado —**que es el default**— Next mete un `<link rel="preload">` por
casi todos. Medido en producción: **241 preloads, 4,2 MB**, o sea **el 90% del peso de
la página**, pedidos antes de pintar un píxel.

Noto Sans JP ya llevaba `preload: false` de un arreglo anterior. A Zen Maru Gothic se
le había pasado — y era la fuente del titular.

## Cómo se encuentra (la sección de método)

1. **La combinación que no cuadra**: FCP de 0,8 s y LCP de 7,5 s. Cuando el primer
   pintado es rápido y el mayor tarda diez veces más, no es un problema de servidor.
2. **`network-requests` agrupado por `resourceType`.** Una línea de Python sobre el
   JSON de Lighthouse. Ahí salió: Font 242 peticiones, 4,2 MB.
3. **Contar los preloads del HTML**: `grep -o 'as="font"' index.html | wc -l`. 241.
4. **Cruzar cada preload con el CSS de cada familia** para saber cuál es la culpable:
   239 de los 241 eran de Zen Maru Gothic.

Ese paso 4 es el que convierte «las fuentes pesan» en «esta fuente, por esta bandera».

## Las dos vueltas, y por qué la primera no bastó

**Vuelta 1 — `preload: false`.** Quita los 241 `<link>`. Resultado medido, los dos
builds en el mismo servidor:

| | antes | después |
|---|---|---|
| Peticiones | 267 | 70 |
| Peso | 5,82 MB | 2,31 MB |
| Fuentes | 242 / 4,1 MB | 45 / 591 KB |
| LCP (simulado) | 39,8 s | 14,2 s |
| **Score** | **62** | **60** |

**El score BAJÓ.** Y ahí está la lección del artículo: el cuello no eran los preloads
sino los **544 KB de CSS** que declaran esos 740 `@font-face`, que van en la ruta
crítica y bloquean el render. Una bandera no lo arregla.

**Vuelta 2 — subsetear.** El texto se conoce en el build (copy escrito a mano), así
que se puede hacer al revés que Google Fonts: un fichero por peso con los caracteres
que aparecen de verdad.

- **930 caracteres únicos** en todo el sitio (738 del copy + los kana completos como
  red de seguridad). De los ~20.000 de la fuente: el **3,7%**.
- **3,6 MB → 140 KB por peso.**
- CSS de fuentes: **588 KB → 45 KB**. `@font-face`: **764 → 15**.
- Y de paso se fueron dos familias enteras: Archivo (cero usos) y Noto Sans JP (último
  eslabón de un fallback que empieza por Zen Maru).

| | original | subset |
|---|---|---|
| **Score** | 62 | **74** |
| FCP | 5,3 s | **1,8 s** |
| LCP | 39,8 s | **9,6 s** |
| Peticiones | 267 | **26** |
| Peso | 5,82 MB | **1,62 MB** |

Comparando capturas: **0,03% de píxeles** con diferencia perceptible.

## El límite, y es lo que hace el artículo honesto

Subsetear **solo vale si conoces el texto al construir**. Un carácter que no esté se
pinta con la fuente del sistema.

- Landing y blog: perfecto, el texto entra por un build.
- **La app y la consola: no.** Los nombres de las 推し y los títulos de los 現場 salen
  de la base de datos. Ahí los subsets de Google siguen siendo lo correcto.

Decir esto es lo que separa «una receta» de «saber cuándo aplicarla».

## El script

Extrae los caracteres del **código fuente**, no del HTML construido: del HTML sería más
exacto pero el build necesita la fuente y la fuente necesitaría el build. Se corre a
mano y su salida se commitea — así el despliegue no depende de sharp/harfbuzz y el peso
de cada fichero se ve en el diff.

Mete los kana completos (180 glifos, unos KB) aunque no aparezcan hoy: evita el fallo
más silencioso de esto, que alguien escriba una palabra nueva en hiragana y le salga
con otra tipografía sin que nadie lo note.

Herramienta: `subset-font` (harfbuzz vía WASM, sin dependencias nativas).

## Lo que NO se arregló, con su aritmética

Tras todo esto, el **Performance de móvil sigue en 74**. Tres intentos más, medidos
contra la misma base:

| | Score | LCP |
|---|---|---|
| nada | 74 | 9,5 s |
| + `priority` en la imagen del LCP | 74 | 9,8 s |
| + PostHog en `requestIdleCallback` | 74 | 9,8 s |
| + `import()` dinámico de posthog-js | **74** | **8,6 s** |

Lighthouse móvil simula **1,6 Mbps** y la página pesa **1,6 MB**: **8,2 segundos solo
de descarga**. Con ese suelo, ningún orden de carga cambia la nota.

| | peso | % |
|---|---|---|
| **JavaScript** | **776 KB** | **46%** |
| Fuentes | 440 KB | 26% |
| Imágenes | 228 KB | 14% |
| HTML | 165 KB | 10% |
| CSS | 45 KB | 3% |

De esos 776 KB, **231 eran `posthog-js`** — el chunk más grande. Sacarlo del bundle
inicial bajó el JS a 258 KB en producción. El resto es el framework.

## Los tres LCP, y cuál importa

Dato que el artículo puede regalar y que ahorra discusiones:

| fuente | valor | qué es |
|---|---|---|
| trace de Chrome | **492 ms** | lo que pasó de verdad en esa carga |
| DevTools Live Metrics | **0,71 s** | una carga real, sin throttling |
| Lighthouse móvil | **6-9 s** | simulación de 4G lento |
| CrUX | *sin datos* | usuarios reales — **es lo que Google usa** |

Los cuatro son correctos y miden cosas distintas. Y con poco tráfico CrUX no tiene
volumen, así que **no se puede afirmar ninguna mejora de campo**. Decirlo.

## Ángulos que NO usar

- «Cómo llegué a 100 en Lighthouse». No se llegó, y perseguir el número es justo el
  error que el artículo critica.
- Tutorial genérico de optimización web. Lo específico —y lo que no está escrito en
  japonés— es el comportamiento de `next/font` con CJK.
- Prometer números de SEO o de conversión. No hay datos.
