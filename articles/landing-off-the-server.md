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

     TÍTULO — candidatos, por el problema y no por la herramienta:
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

## なぜ分けたのか

> ES: El motivo NO es rendimiento, y conviene decirlo pronto para que no parezca
> optimización porque sí: `oshisuki.com` es la base de crecimiento por SEO, blog
> incluido. **Googlebotが5xxを見続けると、クロールバジェットが削られ、最後には
> インデックスから消える。**
>
> Un servidor de aplicación se cae: se despliega mal, se queda sin memoria, la base
> no responde. Eso es asumible para la app. No lo es para el contenido que tiene que
> estar ahí cuando pase el rastreador.

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
