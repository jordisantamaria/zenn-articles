---
title: "TODO: アプリのAPIは引っ越せない（仮題 — 候補は notes/landing-off-the-server.md）"
emoji: "🪧"
type: "tech"
topics: ["cloudflare", "nextjs", "個人開発", "seo", "パフォーマンス"]
published: false
---

<!-- ⚠️ ESQUELETO — NO PUBLICAR TAL CUAL.
     Zenn IMPRIME los comentarios HTML: borrar este bloque y todas las líneas
     «> ES:» antes de poner published: true.
     Datos medidos y fuentes: notes/landing-off-the-server.md

     ARTÍCULO 2 DE 2. Este es «qué NO debe depender del servidor».
     El otro —dónde corre el servidor— es articles/serverless-to-server.md.
     Publicar éste DESPUÉS y con unos días de separación: son dos ideas y merecen
     dos lecturas.

     TÍTULO — el lector objetivo tiene una landing lenta o mal indexada y busca a
     quién preguntar. Por el problema, y por el SUYO, no por la herramienta:
       a) アプリのAPIは引っ越せない ― 1つのドメイン、2つのオリジン  ← la restricción
       b) ランディングページが、サーバーと一緒に落ちる  ← el problema
       c) Googlebotに5xxを見せ続けると、インデックスから消える  ← la consecuencia
     Evitar «Cloudflare Pagesにデプロイした話»: describe la herramienta, y de eso hay
     cientos. Lo que no está escrito es (a). -->

> ES: **Por qué esto no es «desplegué Next en Cloudflare Pages» más.**
> Tutoriales de eso hay cientos. Lo que no está escrito es la restricción:
>
> **La API no se podía mover de dominio.** La app publicada en App Store y Google
> Play lleva `https://oshisuki.com` clavado en el binario y no se puede actualizar
> hacia atrás. Quien tenga la versión de julio seguirá pidiendo ahí para siempre.
>
> Así que no es «mover la web a otro sitio». Es **partir un dominio entre dos
> orígenes**, y eso solo se decide en el borde. Ése es el artículo.

## なぜ速くするのか — 「速い＝いいこと」では記事にならない

> ES: **La sección que decide si el artículo vale algo.** Sin ella esto es un tutorial
> más de optimización; con ella es una decisión de negocio explicada. Va ARRIBA, antes
> de cualquier detalle técnico.
>
> La landing de un producto no existe para ser bonita: existe para convertir una visita
> en una instalación. Tres motivos concretos, y ninguno es «me gusta que vaya rápido»:
>
> **1. SEO — es de donde tiene que venir el crecimiento.**
> El LCP es factor de ranking directo (Core Web Vitals). Y hay algo peor que rankear
> mal: Googlebot que se encuentra 5xx sostenidos **recorta el presupuesto de rastreo**
> y acaba desindexando. Una landing que comparte servidor con la API se cae cuando se
> cae la API — y el rastreador no vuelve a intentarlo como lo haría una persona.
>
> **2. CRO — cada segundo se lleva instalaciones.**
> Toda la landing existe para un solo evento: pulsar el botón de la tienda. Quien llega
> y espera tres segundos a que aparezca el hero no llega a ese botón. No hay forma de
> medir cuántos se pierden —los que se van no dejan rastro—, así que **hay que decirlo
> como lo que es: un coste invisible.** No inventar un porcentaje.
>
> **3. La campaña — el tráfico de un post se quema una sola vez.**
> Éste es el que más duele y el que menos se escribe. El tráfico no viene de Google
> todavía: viene de X, de posts que cuestan una tarde escribir. Ese tráfico llega
> **concentrado en veinte minutos** y no se repite. Si la landing tarda en abrir
> justo entonces, el trabajo del post se ha ido — y no hay reintento.
>
> ES: La frase que resume las tres y que puede abrir el artículo:
> **ランディングページが遅いのは、UXの問題じゃなくて集客の問題。**

## なぜサーバーから出したのか

> ES: Aquí ya sí el motivo técnico, que ahora se lee como consecuencia de lo de arriba
> y no como capricho: un servidor de aplicación se cae —se despliega mal, se queda sin
> memoria, la base no responde—. Eso es asumible para la app: el usuario reintenta.
>
> No lo es para el contenido que sostiene los tres puntos anteriores. Ahí quien pasa
> mientras está caído es un rastreador que no reintenta igual, o alguien que venía de
> un post y no va a volver.

## 分ける前に気づいたこと：ルートレイアウトの動的API

> ES: Sección corta y muy rentable — es diagnóstico gratis que casi nadie hace.
>
> Antes de separar nada, la landing YA se renderizaba en el servidor en cada visita.
> La causa estaba en el layout raíz: `getLocale()` acaba llamando a `cookies()`. **Una
> API dinámica en el layout raíz vuelve dinámico todo lo que cuelga de él** — hasta
> la página de política de privacidad, que es texto que no cambia nunca.
>
> Se arregla bajando el provider de i18n a los grupos de rutas que traducen algo.
>
> La lección concreta: mirar la salida de `next build` y buscar `ƒ` donde debería
> haber `○`. Antes de cambiar de infraestructura, comprobar que no te estás
> renderizando el sitio entero por una cookie.

## 1つのドメイン、2つのオリジン

> ES: El corazón. La tabla:
>
> | | どこ | 落ちたら |
> |---|---|---|
> | ランディング・規約類 | Cloudflare Pages | Cloudflareが落ちたとき |
> | API・メールの着地ページ | Fly.io | Flyが落ちたとき |
>
> El reparto lo hace un Worker en la ruta `oshisuki.com/*`. La propiedad que lo hace
> seguro: **lo que va a Fly se reenvía con `fetch(request)` tal cual** — mismo host,
> mismas cabeceras, mismo cuerpo. Para la app publicada no cambia nada: ni el `Host`
> que lee el middleware, ni el dominio de las cookies de better-auth, ni el streaming.
>
> ES: Y el detalle de diseño que merece explicarse, porque es al revés de lo que se
> espera: en el middleware de la app se enumera **lo público**; en el Worker se
> enumera **lo de Fly**, y el resto va a Pages. El default apunta al lado que crece —
> la página nueva del blog se publica cada semana, la ruta nueva de la app casi nunca.

## `/_next/` は両方が要求する

> ES: **El detalle técnico que no está escrito en ningún sitio y que rompe el
> despliegue si no lo ves.**
>
> Los dos Next sirven sus assets bajo el mismo prefijo. Los nombres llevan hash, así
> que las rutas concretas no colisionan nunca; **lo que colisiona es el prefijo**.
>
>   - `/_next/*` → Pages: `/reset-password` (que sirve Fly) se queda sin estilos.
>   - `/_next/*` → Fly: la landing entera se queda sin JS ni CSS. Catastrófico.
>
> Se resuelve preguntando: Pages primero y, si devuelve 404, Fly. La petición de más
> solo ocurre al abrir esas dos páginas. La alternativa (`assetPrefix`) obliga a CORS
> y a que cada entorno lo configure bien.

## `images.unoptimized` は「最適化しない」ではない

> ES: `output: "export"` lo obliga: no hay servidor que redimensione al vuelo, así
> que `next/image` sirve el fichero tal cual. La optimización no desaparece — **se
> mueve antes del build**, a un script que se corre a mano y cuyo resultado se
> commitea.
>
> Ventajas de que no vaya en el build: el despliegue no depende de sharp, dos
> ejecuciones no dan bytes distintos, y el peso de cada imagen se ve en el diff de la
> PR en vez de descubrirse en producción.
>
> Medido: los dos personajes del hero eran PNG de 997 px pintados a 320 CSS px.
> **1,7 MB → 104 KB.**

## 送信済みのメールは書き換えられない

> ES: **El mejor detalle del artículo. Solo aparece haciéndolo.**
>
> El correo de bienvenida enlaza `oshisuki.com/chibi-wota.png`. Esos correos están en
> bandejas de gente real. Dejar los assets solo en WebP —que Outlook no pinta— o
> cambiarles el nombre habría vaciado de imágenes cada bienvenida ya enviada.
>
> Solución: los PNG siguen con su nombre de siempre, recomprimidos al tamaño al que
> el correo los pinta. La web usa los WebP.
>
> Generalizable en una línea: **メールから参照しているアセットは公開APIと同じ。
> リネームできない。**

## ついでに直った（そして直さないと危なかった）こと

> ES: Dos cosas que salieron al activar el proxy, y conviene separarlas porque una es
> seguridad y la otra es un efecto secundario:
>
> 1. **SSL/TLSが`Full`だった。** Cifra el tramo Cloudflare↔origen pero **no valida el
>    certificado**: acepta el de cualquiera que se ponga en medio. Con el DNS en gris
>    no aplicaba a nada; en cuanto metes a Cloudflare en el camino, sí. `Full (strict)`.
> 2. **El TLS pasó a terminar en Tokio** (`cf-ray: …-NRT`) en vez de Singapur. Eso no
>    se buscaba y **también afecta a la API**, que no tiene nada que ver con Pages.
>    Decirlo separado o el lector atribuye a Pages algo que es del proxy.

## 結果

> ES: La tabla:
>
> | | 前 | 後 |
> |---|---|---|
> | `/` | 371 ms | **111 ms** |
> | `/faq` | ~380 ms | **85 ms** |
> | ヒーローの画像 | 1.7 MB | **104 KB** |
>
> Y la honestidad que hace falta: **no hay números de SEO todavía.** El sitio es
> pequeño, CrUX no tiene volumen suficiente para dar datos de campo, y decir «mejoró
> el ranking» sería inventarlo. Lo que sí se puede afirmar es lo estructural: la
> landing ya no puede caerse con el servidor de aplicación.

## 壊れたもの：CIが、なくなったページを3分待っていた

> ES: Sección de cierre técnico. Es corta y se lee sola porque el mensaje de error
> miente:
>
> ```
> [WebServer] ✓ Ready in 142ms
> Error: Timed out waiting 180000ms from config.webServer.
> ```
>
> Playwright sondea una URL hasta que responda <400 antes de arrancar los tests, y esa
> URL era `/`. Al mudarse la landing, la app dejó de tener raíz: 404 para siempre. El
> mensaje dice «el servidor no levanta» justo debajo de un log que dice que levantó en
> 142 ms.
>
> Y el arreglo obvio también era trampa: `/api/health` parece la elección correcta
> —existe justo para decir «estoy vivo»— pero hace `SELECT 1`, así que devuelve 503
> cuando la base duerme, y Neon duerme por inactividad. Habría cambiado un timeout de
> tres minutos por el mismo timeout de tres minutos. Va a `/login`.
>
> ES: Y el hallazgo colateral, que da una frase memorable: al tocar el workflow salió
> un paso que filtraba por un paquete inexistente. **pnpm sale con 0 cuando el filtro
> no encuentra nada**, así que llevaba tiempo en verde sin ejecutar un solo test.
> 何も検証しないチェックは、無いより悪い。カバレッジがあるように見えるから。

## 直らなかったこと、そしてなぜ直らないのか

> ES: **La sección que más autoridad da del artículo, y la que casi nadie escribe.**
> Demuestra que se entiende el sistema entero en vez de aplicar recetas. Y es honesta:
> el número se quedó donde se quedó.
>
> Tras sacar la landing del servidor, recortar la tipografía y arreglar el LCP, el
> **Performance de móvil no se movió de 74**. Se probaron tres cosas por separado,
> midiendo cada una contra la misma base en el mismo servidor:
>
> | | Score | LCP |
> |---|---|---|
> | 何もしない | 74 | 9.5 s |
> | + `priority` | 74 | 9.8 s |
> | + PostHogをidleに | 74 | 9.8 s |
> | + 動的import | **74** | **8.6 s** |
>
> El motivo, y es aritmética: Lighthouse móvil simula **1,6 Mbps** y la página pesa
> **1,6 MB**. Son **8,2 segundos solo de descarga**. Con ese suelo, ningún orden de
> carga cambia la nota — solo quitar bytes la cambia.
>
> | | 重さ | % |
> |---|---|---|
> | **JavaScript** | **776 KB** | **46%** |
> | フォント | 440 KB | 26% |
> | 画像 | 228 KB | 14% |
>
> ES: La conclusión transferible, que es lo que se lleva el lector:
> **スコアが動かないときは、順番ではなくバイト数を見る。**
>
> Y el matiz que separa medir de creer: el LCP **real** del trace es de **492 ms**.
> Los 8,6 s son la simulación de una red muy lenta. Los dos números son correctos y
> miden cosas distintas — explicar eso vale más para el lector que subir a 90.

## 試して、外れたこと

> ES: **Cuatro intentos fallidos, todos medidos.** Van juntos porque cada uno enseña
> algo distinto sobre diagnosticar, y porque contar solo los aciertos hace que parezca
> suerte.
>
> **1. `priority` en la imagen del LCP.** Es objetivamente correcto —`next/image` la
> marcaba `loading="lazy"`— pero **no movió el número**: su preload compite con el CSS
> y las fuentes, y se compensa. Se queda igualmente, porque en una red real sí ayuda.
> Lección: *un arreglo puede ser correcto y no salir en el marcador.*
>
> **2. `aria-hidden` para eximir del contraste.** Dos textos de 9 px dentro de una
> ilustración: parecía que marcándolos como decorativos quedaban fuera del criterio.
> **axe los siguió midiendo, y con razón**: `aria-hidden` los saca del lector de
> pantalla, pero **el texto sigue viéndose**, y el criterio de contraste existe para
> quien tiene poca visión, no para quien no ve nada. La exención de WCAG es para texto
> dentro de una imagen de píxeles.
>
> **3. `/api/health` como sonda de Playwright.** Al mudarse la landing, el CI se quedó
> esperando tres minutos a `/`, que ya no existía. El arreglo evidente era apuntar a
> `/api/health` —el endpoint que existe justo para decir «estoy vivo»— y era una
> trampa: hace `SELECT 1`, así que devuelve 503 cuando la base duerme. Habría cambiado
> un timeout de tres minutos por el mismo timeout de tres minutos.
>
> **4. Oscurecer el color de marca para cumplir contraste.** Cumplía. Pero ese color
> también es FONDO de una cápsula, y oscurecido deja de leerse como rosa: se lee
> granate. Se revirtió. Lección: *un token de color no se cambia sin mirar en qué
> propiedad se usa*.

## おわりに

> ES: La lección transferible, que es la que justifica los dos artículos juntos:
>
> **アプリとコンテンツは、求められる可用性が違う。** La app puede caerse un rato: los
> usuarios reintentan. El contenido que sostiene el crecimiento por búsqueda, no —
> ahí quien pasa mientras está caído es un rastreador que no reintenta igual, y lo que
> se pierde no se recupera reiniciando.
>
> Separarlos no es sobre-ingeniería si el motivo es ése. Lo sería si fuera por
> rendimiento.
>
> ES: Enlace al artículo 1 en una línea.
