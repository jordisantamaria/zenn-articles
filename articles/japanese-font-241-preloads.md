---
title: "TODO: 日本語フォントで241個のpreloadが入っていた（仮題 — 候補は notes/）"
emoji: "🔤"
type: "tech"
topics: ["nextjs", "パフォーマンス", "フォント", "個人開発", "webフロントエンド"]
published: false
---

<!-- ⚠️ ESQUELETO — NO PUBLICAR TAL CUAL.
     Zenn IMPRIME los comentarios HTML: borrar este bloque y todas las líneas
     «> ES:» antes de poner published: true.
     Datos medidos y fuentes: notes/japanese-font-241-preloads.md

     ARTÍCULO 3 DE 3. Los otros: serverless-to-server (dónde corre el servidor) y
     landing-off-the-server (qué no debe depender de él). Éste va de POR QUÉ la
     página pesaba lo que pesaba, y es el más vendible: le pasa a todo dev japonés
     que use next/font y casi ninguno lo sabe.

     OBJETIVO: autoridad técnica que acabe en encargos de optimización. Método por
     encima del resultado, los fallos contados, números medidos o nada, antes/después
     en el mismo entorno, decir lo que no se puede arreglar, y SIN CTA.

     TÍTULO — el lector tiene que reconocer SU problema o querer ir a comprobarlo:
       a) 日本語フォントで241個のpreloadが入っていた  ← el número imposible
       b) ページの9割がフォントだった  ← la proporción
       c) next/fontは日本語フォントを366個に分割する  ← el mecanismo
       d) フォントを絞ったらCSSが588KB→45KBになった  ← el resultado
     (a) es el mejor: es un número que no puede ser y obliga a ir a mirar el propio
     HTML. Evitar «Lighthouseで100を取る方法» — no se tomó, y perseguir el número es
     justo el error que este artículo critica. -->

> ES: **La ventaja que este artículo tiene sobre los otros dos: el lector puede
> comprobarlo en su web en treinta segundos.** Poner el comando pronto, casi al
> principio, para que vaya a mirar antes de terminar de leer:
>
> ```bash
> curl -s https://tu-sitio.com | grep -o 'as="font"' | wc -l
> ```
>
> Un artículo que hace que alguien encuentre el problema en su propio sitio genera
> más confianza que cualquier caso de estudio.

## はじめに — 数字が合わなかった

> ES: Entrar por la contradicción, no por la conclusión. El informe decía FCP 0,8 s y
> LCP 7,5 s. **Cuando el primer pintado va rápido y el mayor tarda diez veces más, no
> es el servidor** — el servidor ya había respondido.
>
> Y el dato que cierra la escena: la página pesaba 4,8 MB. Una landing estática, sin
> vídeo, con doce capturas de pantalla en WebP.

## 犯人の見つけ方

> ES: **Sección de método.** Es la que hace el trabajo para el objetivo del artículo:
> el lector se lleva un procedimiento aplicable mañana. Cuatro pasos, de lo barato a
> lo caro:
>
> 1. **La combinación que no cuadra.** FCP rápido + LCP lento = el problema está en lo
>    que se descarga después, no en quién responde.
> 2. **Agrupar `network-requests` por `resourceType`.** El JSON de Lighthouse lo trae;
>    son cuatro líneas de Python. Aquí salió: **Font, 242 peticiones, 4,2 MB**.
> 3. **Contar los preloads del HTML** — `grep -o 'as="font"' | wc -l` → **241**.
> 4. **Cruzar cada preload con el CSS de cada familia** para saber cuál es. 239 de los
>    241 eran de una sola fuente.
>
> ES: Ese paso 4 es el que convierte «las fuentes pesan» en «esta fuente, por esta
> bandera». Sin él, el diagnóstico se queda en una intuición.

## next/fontは日本語フォントを366個に分割する

> ES: El mecanismo, explicado de una vez y sin culpar a nadie — la decisión de Google
> Fonts es correcta para lo que resuelve.
>
> Una fuente japonesa tiene ~20.000 glifos y pesa 3,6 MB. No se puede mandar entera,
> así que se parte por rango unicode y cada trozo se declara con su `unicode-range`;
> el navegador pide solo los que contienen glifos que hay en la página. Es listo.
>
> | familia | `@font-face` |
> |---|---|
> | Zen Maru Gothic | **366** |
> | Noto Sans JP | **373** |
> | Archivo + Figtree (latín) | 24 |
>
> El problema es el **default**: con `preload` activado, Next mete un `<link>` por casi
> todos. Medido: **241 preloads, 4,2 MB — el 90% del peso de la página**, pedidos antes
> de pintar un píxel.
>
> ES: Detalle que hace el artículo honesto: Noto Sans JP ya llevaba `preload: false` de
> un arreglo anterior. **A alguien ya le había pasado esto y solo lo arregló a medias.**
> Contarlo así vale más que presentarse como quien lo vio a la primera.

## 1回目：`preload: false` — スコアは下がった

> ES: **La sección que más autoridad da del artículo.** El arreglo obvio funciona, y
> aun así el marcador empeora.
>
> | | 前 | 後 |
> |---|---|---|
> | リクエスト | 267 | 70 |
> | 重さ | 5,82 MB | 2,31 MB |
> | フォント | 242 / 4,1 MB | 45 / 591 KB |
> | LCP（シミュレーション） | 39,8 s | 14,2 s |
> | **スコア** | **62** | **60** |
>
> Quitó 4 MB de descargas y **bajó dos puntos**. El motivo: el cuello no eran los
> preloads sino los **544 KB de CSS** que declaran esos 740 `@font-face` — van en la
> ruta crítica y bloquean el render. Lighthouse lo llama «Reduce unused CSS» y son
> exactamente los dos ficheros de fuentes.
>
> ES: La lección transferible, que es lo que se lleva un cliente potencial:
> **バンドルを軽くしても、クリティカルパスが同じなら数字は動かない。**

## 2回目：使う文字だけに絞る

> ES: El giro. Google Fonts parte la fuente en trozos genéricos porque **no sabe qué
> texto vas a mostrar**. Aquí sí se sabe —el copy está escrito a mano y entra por un
> build—, así que se puede hacer al revés: un fichero por peso con los caracteres que
> aparecen de verdad.
>
> - **930 caracteres únicos** en todo el sitio: 738 del copy más los kana completos
>   como red de seguridad. De los ~20.000 de la fuente, el **3,7%**.
> - **3,6 MB → 140 KB** por peso.
> - CSS de fuentes **588 KB → 45 KB**. `@font-face` **764 → 15**.
>
> Y de paso salieron dos familias que nadie usaba: Archivo (cero referencias, solo
> declaraba su variable) y Noto Sans JP (último eslabón de un fallback que empieza por
> Zen Maru, o sea 373 `@font-face` para un caso que no ocurre).
>
> | | 元 | subset |
> |---|---|---|
> | **スコア** | 62 | **74** |
> | FCP | 5,3 s | **1,8 s** |
> | LCP | 39,8 s | **9,6 s** |
> | リクエスト | 267 | **26** |
> | 重さ | 5,82 MB | **1,62 MB** |
>
> Comparando capturas antes y después: **0,03% de píxeles** con diferencia
> perceptible. Es antialiasing.

## この方法が使えない場所

> ES: **La sección que separa «una receta» de «saber cuándo aplicarla»**, y va en
> medio, no al final, para que nadie se la salte.
>
> Subsetear solo vale si **conoces el texto al construir**. Un carácter que no esté en
> el subset se pinta con la fuente del sistema.
>
> - **Landing y blog: perfecto.** El texto entra por un build; el script regenera el
>   subset en cada uno.
> - **La app y la consola: no.** Los nombres de las 推し y los títulos de los 現場 salen
>   de la base de datos, y no hay forma de saber qué kanji va a escribir alguien. Ahí
>   los subsets de Google son lo correcto — y por eso ese lado se quedó como estaba.

## スクリプトの作り方

> ES: Corto y concreto. Tres decisiones que tienen su porqué:
>
> - **Los caracteres salen del CÓDIGO, no del HTML construido.** Del HTML sería más
>   exacto, pero crea un círculo: el build necesita la fuente y la fuente necesitaría
>   el build.
> - **Los kana completos van siempre**, aparezcan o no (180 glifos, unos KB). Evita el
>   fallo más silencioso: que alguien escriba una palabra nueva en hiragana y le salga
>   con otra tipografía sin que nadie lo note.
> - **Se corre a mano y la salida se commitea.** Así el despliegue no depende de
>   harfbuzz y el peso de cada fichero se ve en el diff de la PR, no en producción.
>
> Herramienta: `subset-font` (harfbuzz vía WASM, sin dependencias nativas).
> Licencia: la OFL exige incluir el fichero de licencia junto a los `.woff2`.

## 直らなかったこと

> ES: **Obligatoria, y de las que más confianza generan.** Tras todo lo anterior, el
> Performance de móvil **no pasa de 74**. Tres intentos más, cada uno medido contra la
> misma base en el mismo servidor:
>
> | | スコア | LCP |
> |---|---|---|
> | 何もしない | 74 | 9,5 s |
> | + LCP画像に`priority` | 74 | 9,8 s |
> | + PostHogを`requestIdleCallback`に | 74 | 9,8 s |
> | + posthog-jsを動的`import()` | **74** | **8,6 s** |
>
> El motivo es aritmética: Lighthouse móvil simula **1,6 Mbps** y la página pesa
> **1,6 MB**. **8,2 segundos solo de descarga.** Con ese suelo, ningún orden de carga
> cambia la nota.
>
> | | 重さ | % |
> |---|---|---|
> | **JavaScript** | **776 KB** | **46%** |
> | フォント | 440 KB | 26% |
> | 画像 | 228 KB | 14% |
>
> De esos 776 KB, **231 eran `posthog-js`** — el chunk más grande de la landing.
> Sacarlo del bundle inicial dejó el JS en 258 KB. El resto es el framework, y eso ya
> no es un ajuste: es cambiar de arquitectura.
>
> ES: La frase que se lleva el lector:
> **スコアが動かないときは、順番ではなくバイト数を見る。**

## LCPが4つある話

> ES: **Cierre técnico, y probablemente lo más útil del artículo para alguien que
> discute métricas con su equipo.** Los cuatro números son correctos y miden cosas
> distintas:
>
> | どこから | 値 | 何を測っているか |
> |---|---|---|
> | Chromeのトレース | **492 ms** | そのロードで実際に起きたこと |
> | DevTools Live Metrics | **0,71 s** | 実際のロード、スロットリングなし |
> | Lighthouse（モバイル） | **6〜9 s** | 4Gを想定したシミュレーション |
> | CrUX | データなし | **実ユーザー — Googleが見ているのはこれ** |
>
> Y la parte incómoda: con poco tráfico **CrUX no tiene volumen suficiente**, así que
> no se puede afirmar ninguna mejora de campo. Decirlo en vez de inventar un
> porcentaje.

## おわりに

> ES: Tres frases que aguantan, y ninguna es «optimizad vuestras fuentes»:
>
> 1. **デフォルトを疑う。** El problema no era una configuración mala: era la que viene
>    de fábrica, pensada para latín y aplicada a japonés.
> 2. **バンドルではなくクリティカルパスを見る。** 4 MB fuera y el score bajó dos puntos.
> 3. **直らないものは、なぜ直らないかまで測る。** Un suelo de 8,2 segundos de descarga
>    explica más que cualquier truco.
>
> ES: Enlace a los otros dos artículos al final, en una línea. Sin CTA — el artículo
> es la prueba, y el enlace vive en el perfil.
