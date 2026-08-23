---
title: "TODO: 月1.5万リクエストでVercelの無料枠が尽きた（仮題 — 候補は notes/serverless-to-server.md）"
emoji: "🌏"
type: "tech"
topics: ["vercel", "flyio", "個人開発", "nextjs", "パフォーマンス"]
published: false
---

<!-- ⚠️ ESQUELETO — NO PUBLICAR TAL CUAL.
     Zenn IMPRIME los comentarios HTML: borrar este bloque y todas las líneas
     «> ES:» antes de poner published: true.
     Datos medidos y fuentes: notes/serverless-to-server.md

     ARTÍCULO 1 DE 2. Este es «dónde corre el servidor».
     El otro —sacar la landing del servidor— es articles/landing-off-the-server.md.
     Se partieron porque juntos eran dos tesis compitiendo en 6.000 caracteres.

     TÍTULO — candidatos, todos apuntan al problema y no a la solución:
       a) 月1.5万リクエストでVercelの無料枠が尽きた  ← el número contraintuitivo
       b) アクセスの42%が、サーバーが起きるのを待っていた  ← el hallazgo
       c) ユーザーが少ないほど、サーバーレスは高くつく  ← la tesis en una línea
       d) サーバーは、ユーザーの近くではなくDBの近くに置く  ← el giro
     (a) para el titular y (c) como primera frase funciona bien: el número engancha,
     la tesis retiene. Evitar «VercelからFlyへ移行した話»: describe el trámite, no el
     problema, y el lector de Zenn busca el problema. -->

> ES: **Por qué este artículo no es «Vercel vs Fly» más.**
> De esos hay cientos y casi todos comparan features. Éste tiene cuatro cosas que
> no están escritas en japonés:
>
> 1. **El free tier se agotó con 14.500 requests al mes.** Casi sin tráfico. Rompe
>    la intuición: no se agota por éxito, se agota por arquitectura mal encajada.
> 2. **El ángulo japonés**: si tu producto es para Japón, el default de Vercel
>    (`iad1`, Washington) cuesta dinero *y* velocidad. Y el remedio obvio —mover a
>    Tokio— es la región **más cara** de Vercel: +58% de Active CPU.
> 3. **El servidor acabó en Singapur, no en Tokio.** Es lo mejor del artículo y va
>    explicado abajo.
> 4. **La contraintuición del pooler**: serverless escala *peor* contra Postgres
>    que un servidor persistente, no mejor.
>
> La tesis honesta: **se compraron las desventajas del serverless sin poder usar su
> ventaja**, porque la elasticidad está detrás del paywall. No «Vercel es malo».

## はじめに

> ES: Entrar por la escena, no por la conclusión. La escena es literal y buena: un
> correo de Vercel un viernes por la noche diciendo «has usado el 75% de tu free
> tier», y al abrir el dashboard a la mañana siguiente el número ya era 96%. Con
> una app publicada en las tiendas y usuarias reales usándola ese mismo sábado.
>
> La pregunta del artículo en una línea:
> 「ユーザーがほとんどいないのに、なぜ無料枠を使い切ったのか？」

## なぜ速くするのか — 無料枠の話ではない

> ES: **La sección que le da sentido a todo lo demás, y va antes del primer dato
> técnico.** Sin ella el artículo es «se me agotó el free tier y migré», que es una
> anécdota. Con ella es «por qué la latencia decide si una app sobrevive».
>
> El correo de Vercel fue el disparador, no el motivo. El motivo es dónde y cómo se
> usa esta app:
>
> **1. Se abre de pie, en el 現場, con una mano.**
> 推しスキ no se consulta en un escritorio: se abre en el hueco entre actuaciones para
> mirar a qué hora sale tu 推し, con el móvil en una mano y la ペンライト en la otra, en
> un recinto lleno donde la cobertura ya es mala de por sí. Dos segundos ahí no se
> leen como «va lento»: se leen como **«está rota»**, y guardas el móvil.
>
> **2. Lo que se pierde no es una sesión, es la costumbre.**
> Una app de agenda vive de que la abras ANTES de cada 現場. Si las tres primeras veces
> tarda, no hay una cuarta — y sin esa costumbre no hay producto, porque nadie va a
> abrir a mano una app que le cuesta más que mirar el post de X original.
>
> ES: Ésta es la frase que sostiene el artículo:
> **リテンションは機能じゃなくて、開くまでの時間で決まる。**
>
> **3. Y el 42% no es una media: es una ruleta.**
> Con casi la mitad de las peticiones pagando el arranque completo, no hay una
> «experiencia lenta» uniforme a la que acostumbrarse. Unas veces abre al instante y
> otras tarda dos segundos, sin patrón. **Una app impredecible se siente peor que una
> lenta**, porque no puedes anticiparla — y eso es exactamente lo que produce el
> serverless con tráfico disperso.
>
> ES: Cerrar la sección conectando con el free tier, para que no parezca que se
> ignora: el correo de Vercel solo puso fecha a algo que ya había que hacer. Sin él,
> la migración habría pasado igual — más tarde y con más usuarios sufriéndolo.

## 状況：アプリはリリース済み、トラフィックはほぼゼロ

> ES: Contexto mínimo para que las cifras signifiquen algo. 推し活アプリ publicada en
> App Store y Google Play; backend Next.js (App Router) + Hono + Prisma +
> better-auth sobre Neon. Todo en el free tier de Vercel.
>   - 12時間で241インボケーション（月1.5万弱）
>   - Fluid Active CPU: 4時間の上限に対して3時間50分

## 犯人の見つけ方 — 推測しない

> ES: **Sección de método, y es la que trabaja para el objetivo del artículo.** Un
> lector con una app lenta se lleva de aquí un procedimiento que puede aplicar mañana,
> y de paso saca la conclusión de que quien lo escribe sabe diagnosticar.
>
> El orden importa, y va de lo barato a lo caro:
>
> 1. **Observability → Functions, no el dashboard de Usage.** El total no dice nada;
>    lo que dice algo es **Active CPU P75 por ruta** y el **% de cold start por ruta**.
>    Ahí salió que el 20% de las invocaciones eran bots escaneando `/_not-found` y
>    `/robots.txt` — coste puro que nadie mira.
> 2. **Dibujar el mapa físico antes de tocar nada**: dónde está el usuario, dónde la
>    función, dónde la base. Tres líneas en una tabla. En este caso el mapa era el
>    diagnóstico entero.
> 3. **Contar los viajes, no las distancias.** Una petición son 1 viaje usuario↔servidor
>    y N viajes servidor↔base. Ese multiplicador es lo que casi nadie calcula antes de
>    elegir región, y es lo que decide.
> 4. **Medir la cadena en frío de verdad**, con `curl -w "%{time_total}"`, en vez de
>    sumar estimaciones. La diferencia entre un artículo creíble y uno que suena a
>    marketing es ésa.
>
> ES: Y decir explícitamente qué NO sirve: mirar el bundle, optimizar imágenes,
> añadir caché. Nada de eso toca el problema cuando el coste está en el arranque.
> Empezar por ahí es el error más común.

## 犯人はトラフィックではなかった

> ES: El corazón del artículo. El desglose de Observability:
>   - Active CPU P75: **593ms／リクエスト**
>   - コールドスタート率: **42.3%**
>
> Con tráfico bajo *y disperso*, Fluid no puede mantener instancias calientes. Casi
> la mitad de las peticiones pagan el arranque completo de Next.js + Prisma +
> better-auth. **Cuanto menos tráfico tienes, peor es el coste por petición.**
>
> Y el método, que es lo que el lector se lleva: no adivines, mira Active CPU P75 y
> el % de cold start POR RUTA. Ahí salió también que **~20% de las invocaciones eran
> bots** escaneando `/_not-found` y `/robots.txt`.

## 地図を描いたら、原因が見えた

> ES: La tabla que hace clic:
>
> | | 場所 |
> |---|---|
> | ユーザー | 日本 |
> | Vercel Functions | `iad1`（ワシントンDC）— **デフォルトのまま** |
> | Neon | `ap-southeast-1`（シンガポール） |
>
> Y la cadena en frío, sumada paso a paso:
>   1. 関数のコールドスタート（ワシントン）
>   2. 東京 → ワシントン
>   3. Neonの起動（5分でsuspendする）
>   4. ワシントン → シンガポール × クエリ数
>
> Remate: **Neonも寝ていた。** El doble cold start es lo que nadie mide, y explica
> la diferencia entre «lento» y «dos segundos».

## 東京に移せば解決、ではなかった

> ES: El giro que da valor al artículo para el lector japonés. Mover a Tokio EN
> VERCEL arregla la latencia pero **sube la tarifa un 58%**: `hnd1` es la región más
> cara del rate card. La tabla de regiones está en notes/.
>
> Y el problema de fondo, el que decide: **el suelo**. El uso real valía $1,07/月;
> el plan Pro cuesta $20/月. El problema no es el precio unitario, es el mínimo.

## サーバーはユーザーの近くではなく、DBの近くに置く

> ES: **La mejor sección del artículo, y la que justifica escribirlo.** Es
> contraintuitiva y está medida.
>
> El plan era Tokio. Acabó en **Singapur**, y a propósito: Neon **no tiene región en
> Japón**. La base estaba y sigue en `ap-southeast-1`.
>
> Hacer el cálculo delante del lector, que es lo que casi nadie hace antes de elegir
> región:
>   - ユーザー ↔ サーバー: **1往復**／リクエスト
>   - サーバー ↔ DB: **N往復**／リクエスト（クエリの数だけ）
>
> Una pantalla hace varias queries. Poner el servidor en Tokio ahorra un viaje y
> paga Tokio↔Singapur multiplicado por cada query. Poniéndolo **al lado de la base**,
> el multiplicador desaparece y solo queda Japón↔Singapur una vez.
>
> El número que lo cierra, medido: la query pasó de **1.541 ms a 35 ms**. No es que
> Fly sea rápido — es que la query dejó de cruzar el océano.
>
> La frase que se lleva el lector: **遅いのはユーザーとの距離ではなく、サーバーと
> 「おしゃべりな依存先」との距離 × おしゃべりの回数。**

## Postgresの話：サーバーレスのほうが弱い

> ES: La sección que más gente citará.
>   - Neon fijado en 0.25 CU (min = max), sin autoescalado
>   - `pooler_enabled: false`
>
> Cada lambda abre su propia conexión → un pico da un connection storm. Un servidor
> persistente mantiene un pool compartido y encola. **Ante un pico real, el servidor
> único es MÁS robusto que las lambdas.**
>
> ES: Ser justo aquí o el artículo pierde credibilidad: con el pooler activado el
> problema se mitiga mucho. No es «serverless es malo», es «el default sin pooler te
> traiciona».

## 移行先を選ぶ：Fly / Railway / AWS

> ES: Comparación corta y honesta, con el criterio explícito:
>   - **Railway**: アジアはシンガポールのみ → （結果的には問題なかったが）当時は除外
>   - **AWS**: Lightsail/EC2は安いが、VPC・ALB・ACM・CI/CDの運用が個人には重い。しかも
>     NeonはAWS上でも別アカウントなので、同リージョンでも私設ネットワークにはならない
>   - **Fly**: `fly deploy`だけで済む、月$3〜5
>
> ES: No escondas lo que se pierde: los preview deployments por PR de Vercel son
> buenísimos y hay que reconstruirlos, o dejarlos en Vercel — que es lo que se hizo.

## 移行の実際

> ES: Los puntos que dolieron de verdad. Cada uno es un párrafo corto:
>   - `output: "standalone"` + **`outputFileTracingRoot`** en monorepo pnpm. Sin el
>     segundo, el standalone sale sin los `packages/*`: construye, arranca, y revienta
>     en el primer import.
>   - Prisma en Docker: **Debian y no Alpine**. Los engines para musl dan guerra y el
>     ahorro no compensa depurarlo el día del cutover.
>   - **`prisma generate` no arranca sin `DATABASE_URL`**, aunque no se conecte a
>     ninguna base: `prisma.config.ts` la resuelve al cargarse. Una URL de mentira en
>     el stage de build lo desbloquea.
>   - **Las `NEXT_PUBLIC_*` se hornean en el build, no en runtime** → build args, no
>     secrets. Es el error que no falla ruidosamente: se despliega apuntando al sitio
>     equivocado y no lo cuenta.
>   - `release_command` para `prisma migrate deploy`: si falla, Fly aborta y la
>     versión anterior sigue sirviendo.
>   - crons de `vercel.json` → GitHub Actions con `CRON_SECRET`.
>   - `fly logs` es un caño en vivo sin memoria → hace falta un sink externo.

## ログの仕組みが、デプロイを落とした

> ES: **La mejor anécdota, y merece sección propia.** Es el tipo de fallo que solo se
> cuenta si lo has vivido, y es lo que hace que un artículo se lea entero.
>
> El transport a Axiom tenía un handler `beforeExit` para no perder los últimos logs.
> `beforeExit` se emite cuando el event loop se vacía — o sea, justo cuando el proceso
> ya se va. El fetch revivía el loop, el proceso salía igualmente, el socket moría a
> media petición y undici emitía un 'error' fuera del `await`: excepción no capturada.
>
> Mató el `release_command`, o sea `prisma migrate deploy`. Fly lo leyó como migración
> fallida y abortó el despliegue. **El sistema de observabilidad tirando la aplicación
> que observa.**
>
> Dos lecciones concretas: (1) `SIGTERM` sí, `beforeExit` no; (2) en un fetch de
> apagado hace falta `.catch()` PEGADO a la promesa además del try/catch, porque el
> error llega fuera del await.

## 引っ越したら、壊れていたものが3つ見つかった

> ES: Sección corta y honesta. Mover de sitio obliga a mirarlo todo:
>   - **R2 estaba roto en producción**: las `S3_*` solo existían en Preview, así que
>     el proxy de imágenes devolvía 500. Nadie lo había notado.
>   - **El logo del login era un 404**: `NEXT_PUBLIC_MARKETING_URL` apuntaba a un
>     dominio `.vercel.app` muerto.
>   - **La base iba 18 días por delante del código**: migraciones aplicadas desde
>     `develop` sin que `main` las tuviera.
>
> La lección: una migración es una auditoría con fecha límite.

## 結果

> ES: La tabla, con las cifras medidas (todas en notes/):
>
> | | 前 | 後 |
> |---|---|---|
> | DBクエリ | 1,541 ms | **35 ms** |
> | リクエスト全体 | ~1.9 s | ~0.35 s |
> | コールドスタート | 42.3% | **0** |
> | 費用 | $20/月（下限） | ~$3.5/月 |
>
> ES: Y lo que de verdad importa de esa tabla, dicho explícitamente para volver al
> principio: **la línea que cuenta no es la de los milisegundos, es la del 42% → 0.**
> Antes la app abría rápido o tardaba dos segundos según le tocara; ahora abre igual
> siempre. Lo que se ganó no es velocidad — es que sea **predecible**, que es lo que
> hace que alguien la abra sin pensarlo antes del 現場 siguiente.
>
> ⚠️ ES: Y la honestidad que toca: **no hay dato de retención todavía.** La migración
> es de hace nada y el volumen es pequeño; decir «mejoró la retención un X%» sería
> inventarlo. Lo que se puede afirmar es lo medido —la latencia y la varianza— y el
> razonamiento de por qué eso importa en esta app. Decirlo así vale más que un número
> falso, y en los comentarios lo agradecen.
>
> Y el matiz que hay que incluir o el artículo es deshonesto: **Vercel mejora con la
> escala.** Los 593 ms/invocación son consecuencia del 42% de cold starts; con tráfico
> denso las instancias se reutilizan y eso baja mucho. La regla de tres sobrestima el
> coste de Vercel a escala alta.

## 試して、外れたこと

> ES: **Sección obligatoria, y de las que más autoridad dan.** Contar solo lo que
> funcionó hace que parezca suerte; contar lo que se descartó y por qué demuestra que
> hubo método. Tres, todas medidas:
>
> **1. 「東京に移せばいい」** — la respuesta obvia y la equivocada. Arregla la latencia
> del usuario y multiplica por N la de la base, además de ser la región más cara de
> Vercel. Se descartó calculando, no probando.
>
> **2. 「Proもう払えばいい」** — también razonable, y también no: el problema no era el
> precio unitario ($1,07 de uso real) sino que existe un suelo de $20. Pagar habría
> tapado el síntoma dejando intactos los cold starts, que eran lo que se sentía.
>
> **3. El sistema de logs, que tumbó un despliegue.** Contado en su sección. Es el
> mejor ejemplo de que una herramienta de observabilidad mal metida hace más daño que
> no tenerla.
>
> ES: Cerrar con lo que esto le enseña al lector sobre su propio proyecto: **antes de
> mover nada, comprobar cuál de las tres respuestas obvias es la suya — y por qué no
> lo es.**

## おわりに

> ES: Cerrar con la lección transferible, no con «migré y ya». Tres frases que
> aguantan:
>   1. **無料枠は成功で尽きるとは限らない。** アーキテクチャのミスマッチでも尽きる。
>   2. トラフィックが少なく、ユーザーが一つの国に集中しているなら、**常時起動の
>      サーバーのほうが速くて安い**。サーバーレスが輝くのは、その逆の形のとき。
>   3. リージョンは「ユーザーの近く」ではなく「一番おしゃべりな依存先の近く」で選ぶ。
>
> Y la nota de humildad que lo hace creíble: esto es reversible. Si el perfil cambia,
> volver a serverless es un fin de semana. No es una guerra santa, es elegir la forma
> que encaja con la carga de HOY.
>
> ES: Enlace al artículo 2 al final, en una línea: la landing salió del servidor
> después, y ésa es otra historia.
