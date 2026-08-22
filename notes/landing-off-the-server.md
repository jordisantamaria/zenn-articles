# notes — landing-off-the-server

Artículo 2 de 2. El primero es `notes/serverless-to-server.md` (Vercel → Fly).
Se partieron porque son dos tesis distintas: aquél va de **dónde corre el servidor**,
éste de **qué NO debe depender de él**.

Todo medido el 2026-08-22 sobre `oshisuki` en producción.

## ⚠️ Aviso antes de publicar

- No publicar IDs de cuenta de Cloudflare, tokens ni nombres de secrets.
- El nombre del proyecto de Pages y las rutas públicas sí son publicables (están en
  el sitio, cualquiera las ve).

## La tesis

`oshisuki.com` es la base de crecimiento por SEO — blog incluido. Googlebot que se
encuentra **5xx sostenidos recorta el presupuesto de rastreo** y acaba desindexando.
La landing no puede caerse con el servidor.

Pero la API **no se puede mudar a otro dominio**: la app publicada en las tiendas
lleva `https://oshisuki.com` clavado en el binario, y no se puede actualizar hacia
atrás. Quien tenga la versión de julio seguirá pidiendo ahí para siempre.

**Un dominio con dos orígenes solo se resuelve en el borde.** Esa es la restricción
que hace el artículo distinto de los cientos de «monté mi landing en Pages».

## El hallazgo previo: una API dinámica contamina todo el árbol

Antes de separar nada, la landing ya se renderizaba en el servidor **en cada visita**,
y la causa estaba en el layout raíz: `getLocale()` acaba en `cookies()`. Una API
dinámica en el layout raíz vuelve dinámico TODO lo que cuelga de él — hasta
`/privacy`, que es texto legal que no cambia nunca.

Se arregló bajando `NextIntlClientProvider` a los grupos que sí traducen algo. El
grupo de marketing no usa i18n: su copy está escrito a mano en japonés.

**Publicable:** mirar la salida de `next build` y buscar `ƒ` donde debería haber `○`.
Es diagnóstico gratis y casi nadie lo hace.

## Cifras

| | Antes | Después |
|---|---|---|
| `/` | 371 ms | **111 ms** |
| `/faq` | ~380 ms | **85 ms** |
| Assets del hero | 1,7 MB (PNG) | **104 KB** (WebP) |

Build: 611 ficheros, 18 MB. De esos, **13 MB son 495 `.woff2`** — `next/font` parte
el japonés en cientos de subsets. (Dato colateral bueno: Googlebot los pedía uno a
uno como si fueran páginas, y 71 acabaron en el informe de indexación. Se arregla con
`Disallow: /_next/static/media/*.woff2$`.)

## El japonés y `next/font`: sección propia, y quizá el mejor material del artículo

Salió después, mirando por qué Lighthouse daba **LCP 7,5 s en móvil con un FCP de
0,8 s** — una combinación que no cuadra.

`next/font` parte una fuente japonesa en cientos de subsets por rango unicode:

| Familia | `@font-face` |
|---|---|
| Zen Maru Gothic | **366** |
| Noto Sans JP | **373** |
| Archivo + Figtree | 24 |

Con `preload` activado (**el default**) Next mete un `<link rel="preload">` por casi
todos. Medido en producción: **241 preloads, 4,2 MB** — el 90% del peso de la página,
pedido antes de pintar un píxel. Noto Sans JP ya llevaba `preload: false`; a Zen Maru
Gothic se le había pasado.

Los dos builds, Lighthouse móvil, mismo servidor:

| | antes | después |
|---|---|---|
| Peticiones | 267 | **70** |
| Peso | 5,82 MB | **2,31 MB** |
| Fuentes | 242 / 4,1 MB | **45 / 591 KB** |
| LCP (simulado) | 39,8 s | **14,2 s** |

**Y el score de Lighthouse NO subió** (60 contra 62). Eso es lo interesante y lo que
hace la sección honesta: el cuello que queda son los **544 KB de CSS** que declaran
esos 740 `@font-face`, en la ruta crítica y bloqueando el render. Una bandera no lo
arregla; pide subsetear la fuente a los glifos que la página usa.

### El aviso que vale la sección entera

**El LCP de Lighthouse no es una medición.** En el trace, el LCP real ocurre a los
**298 ms**; los 7,5 s son la *simulación* de 4G lento, que penaliza sobre todo el
NÚMERO de peticiones compitiendo. Y las Live Metrics de DevTools sobre una carga real
daban **0,71 s**.

Tres números para lo mismo, y los tres correctos en su contexto. Explicar eso es más
útil para el lector que cualquier «cómo subí mi score a 100»:
  - **campo (CrUX / Live metrics)** — lo que ven los usuarios; es lo que Google usa
  - **laboratorio simulado (Lighthouse / PSI)** — comparable entre sitios, no real
  - **trace** — lo que pasó de verdad en esa carga

⚠️ Y la honestidad que toca: con el volumen actual **CrUX no tiene datos suficientes**,
así que no se puede afirmar ninguna mejora de campo. Decirlo.

## El efecto secundario que no se buscaba

Activar el proxy naranja movió el fin del TLS a **Tokio** (`cf-ray: …-NRT`) en vez de
Singapur. Eso explica parte de la mejora y **también afecta a la API**, que no tiene
nada que ver con Pages. Conviene separarlo en el artículo o el lector atribuye a
Pages algo que es del proxy.

Y de paso salió que el modo SSL estaba en **Full**, no en **Full (strict)**: cifra
pero no valida el certificado del origen, o sea que acepta el de cualquiera que se
ponga en medio. Con el DNS en gris no aplicaba a nada; en cuanto metes a Cloudflare
en el camino, sí.

## El reparto va al revés que el middleware, a propósito

- En el middleware de la app se enumera **lo público**, porque la ruta que se olvide
  acabaría servida en el dominio de marketing.
- En el Worker se enumera **lo de Fly**, y todo lo demás va a Pages, porque la ruta
  que se olvide acabaría en Pages.

El default apunta al lado que crece: la página nueva del blog se publica cada semana
y la ruta nueva de la app casi nunca.

A Fly van cuatro cosas: `/api/*`, `/image-proxy/*`, `/reset-password` y `/verify`.
Las dos últimas son el aterrizaje de los correos de better-auth.

## El detalle técnico que nadie escribe: `/_next/` lo piden los dos

Los dos Next sirven sus assets bajo el mismo prefijo. Los nombres llevan hash, así
que **las rutas concretas no colisionan; lo que colisiona es el prefijo**.

Si `/_next/*` va a Pages, `/reset-password` (que sirve Fly) se queda sin estilos. Si
va a Fly, la landing entera se queda sin JS y sin CSS — catastrófico.

Se resuelve preguntando: Pages primero y, si devuelve 404, Fly. Cuesta una petición
de más solo al abrir esas dos páginas. Las alternativas (`assetPrefix`) obligan a
CORS y a que cada entorno lo configure bien.

## `images.unoptimized` no es «servir sin optimizar»

`output: "export"` lo obliga: no hay servidor que redimensione al vuelo. La
optimización se mueve **antes del build**, a un script que se corre a mano y cuyo
resultado se commitea. Así el despliegue no depende de sharp y el peso de cada imagen
se ve en el diff de la PR.

## Los correos ya enviados no se pueden reescribir

**El mejor detalle del artículo, y es de los que solo aparecen haciéndolo.**

El correo de bienvenida enlaza `oshisuki.com/chibi-wota.png`. Esos correos están en
bandejas de gente real. Dejar los assets solo en WebP —que Outlook no pinta— o
cambiarles el nombre habría vaciado de imágenes cada bienvenida ya enviada.

Solución: los PNG siguen ahí con su nombre de siempre, recomprimidos a 300 px (el
correo los pinta a 150). La web usa los WebP.

**Generalizable:** los assets enlazados desde correos son API pública. No se renombran.

## Lo que rompió, y por qué costó verlo

El CI se puso en rojo con un mensaje que apuntaba al sitio equivocado:

```
[WebServer] ✓ Ready in 142ms
Error: Timed out waiting 180000ms from config.webServer.
```

Playwright sondea una URL hasta que responda <400 antes de arrancar los tests, y esa
URL era `/`. Al mudarse la landing, la app dejó de tener raíz: 404 para siempre.
El mensaje dice «el servidor no levanta» y el log de justo encima dice que levantó
en 142 ms.

Y el arreglo obvio también era trampa: `/api/health` parece la elección correcta
—existe justo para eso— pero hace `SELECT 1`, así que da 503 cuando la base duerme.
Neon duerme por inactividad. Habría cambiado un timeout de tres minutos por el mismo
timeout de tres minutos. Va a `/login`.

## Un check que llevaba tiempo mintiendo

Al tocar el workflow salió que había un paso `Run Playwright tests for Marketing app`
filtrando por un paquete que no existe en el workspace. **pnpm sale con 0 cuando el
filtro no encuentra nada**, así que llevaba tiempo saliendo en verde sin ejecutar un
solo test.

Publicable en una línea: un check que no comprueba nada es peor que no tenerlo,
porque parece cobertura.

## Ángulos que NO usar

- «Cloudflare Pages es mejor que Vercel/Fly». No va de eso. Va de que **el contenido
  y la aplicación tienen requisitos de disponibilidad distintos**.
- Tutorial de «cómo desplegar Next en Pages». Está escrito mil veces. Lo que no está
  escrito es el reparto de un dominio entre dos orígenes con un binario publicado de
  por medio.
- Prometer números de SEO. Todavía no hay datos: el sitio es pequeño y CrUX no tiene
  volumen suficiente. Decir «mejoró el ranking» sería inventarlo.
