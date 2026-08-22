# notes — serverless-to-server

Datos medidos el **2026-08-22 (mañana, JST)**, antes de migrar. Todo esto sale de
mediciones reales sobre `oshisuki` en producción, no de estimaciones.

## ⚠️ Aviso antes de publicar

- **No publicar identificadores de infraestructura**: IDs de proyecto Neon
  (`plain-leaf-…`), hosts de endpoint (`ep-…neon.tech`), IDs de equipo Vercel,
  nombres de secrets. Las regiones y los productos sí son publicables.
- **No dar el número exacto de usuarios registrados.** El artículo funciona con
  «リクエスト数», que es el dato técnico relevante, y no obliga a exponer tracción.
- La captura del email de Vercel se puede publicar **recortando el nombre del team**.

## Cifras del problema (medidas)

Fuente: dashboard de Vercel → Usage, y Observability → Functions del proyecto.

| Métrica | Valor |
|---|---|
| Fluid Active CPU consumido | **3h 50m / 4h** (96%) |
| Reparto del cupo del team | oshisuki 2h54m (75,5%) · paopaoanime 55m33s (24,5%) |
| Invocaciones (12 h, prod) | **241** → ≈14.500/mes |
| Active CPU P75 | **593 ms** por invocación |
| Cold start | **42,3%** |
| CPU Throttle P75 | 9,8% |
| Memoria | media 294 MB sobre 2 GB provisionados |
| Ritmo de consumo | ~7-9 min de CPU/día, plano (pico único el 9/8, ~18 min) |

Desglose por ruta en 12 h (all environments):
`/api/[[...rest]]` 140 inv → 2m · `/` 27 → 7s · `/admin` 6 → 4,51s ·
`/_not-found` **38** → 3,02s · `/login` 6 → 1,38s · `/robots.txt` **10** → 870ms

→ **~20% de las invocaciones son bots** escaneando (`/_not-found` + `/robots.txt`).
Dato usable en el artículo: en un proyecto sin tráfico, el ruido de bots es un
porcentaje enorme del coste.

## Topología (la tabla del artículo)

| Pieza | Región |
|---|---|
| Usuarios | Japón |
| Vercel Functions | `iad1` (Washington DC) — el **default**, nunca se cambió |
| Neon | `aws-ap-southeast-1` (Singapur) |

Estado de Neon medido: `compute_size` 0.25, `autoscaling_limit_min_cu` =
`max_cu` = **0.25** (sin autoescalado), `pooler_enabled: **false**`,
`suspend_timeout_seconds: 0` (= default 5 min), estado en el momento de medir:
`idle` / suspendida.

Latencia estimada de la cadena en frío (declararlo como estimación, no medición):
cold start función ~300-800 ms + Tokio→Washington ~150 ms + wake de Neon ~500 ms +
Washington→Singapur ~220 ms × nº de queries.
→ **TODO: medirlo de verdad** con `curl -w` antes del cutover, para poder dar un
número real en vez de una suma de estimaciones. Es la diferencia entre un artículo
creíble y uno que suena a marketing.

## Precios (oficiales, verificados 2026-08-22)

Fluid compute, Active CPU por hora / memoria por GB-hora:

| Región | Active CPU | Provisioned Memory |
|---|---|---|
| Washington `iad1` | $0,128 | $0,0106 |
| **Tokio `hnd1`** | **$0,202** | **$0,0167** |
| Osaka `kix1` | $0,202 | $0,0167 |
| Singapur `sin1` | $0,160 | $0,0133 |
| Seúl `icn1` | $0,169 | $0,0140 |

Tokio es **+58% sobre Washington** en CPU. Ése es el dato que hace el artículo útil
para un lector japonés.
Fuente: https://vercel.com/docs/functions/usage-and-pricing

- Invocaciones: $0,60/millón · Hobby incluye 4 h CPU, 360 GB-h, 1M invocaciones.
- Pro: $20/mes por miembro, con $20 de crédito flexible que absorbe el uso.

**Coste real del uso actual en Tokio**: CPU 2,9 h × $0,202 = $0,59 · memoria
28 GB-h × $0,0167 = $0,47 · invocaciones $0,01 → **≈$1,07/mes**. Contra $20 de
suelo. Ése es el argumento, y hay que darlo con los números delante.

Fly (verificado en https://fly.io/docs/about/pricing/):
`shared-cpu-1x` 256MB $2,02 · 512MB $3,32 · 1GB $5,92 al mes.
Egress Asia-Pacífico $0,04/GB (transferencia actual: 4,15 GB/mes ≈ $0,17).

## Tabla de escalado (para la sección de precios)

| Escala | Vercel Pro (Tokio) | Fly `nrt` |
|---|---|---|
| Hoy | $20 (uso real $1) | ~$4 |
| ×10 | $20 (uso $11) | ~$4 |
| ×30 | ~$32 | ~$6 |
| ×100 | ~$60-100 | ~$12 |

⚠️ Matiz honesto que **hay que incluir** o el artículo es deshonesto: Vercel mejora
con la escala. Los 593 ms/invocación son consecuencia del 42% de cold starts; con
tráfico denso las instancias se reutilizan y eso baja mucho. La regla de tres
sobrestima el coste de Vercel a escala alta.

## Medido DESPUÉS (2026-08-22, migración hecha)

| Métrica | Antes | Después |
|---|---|---|
| Query a la base | **1.541 ms** | **35 ms** (~40×) |
| Petición completa | ~1,9 s | ~0,35 s |
| Cold starts | 42,3 % | **0** |
| Coste | $20/mes de suelo (Pro) | ~$3,50/mes |
| Imagen Docker final | — | 94 MB |

Máquina real: `shared-cpu-1x`, 512 MB, región `sin`. Los 512 MB bastaron (la media
medida antes era 294 MB). Autoescalado 1→2 con `soft_limit = 80` requests.

## ⚠️ EL ÁNGULO CAMBIÓ: acabó en Singapur, NO en Tokio

Esto es lo más importante de las notas, y lo que hace que el artículo valga la pena:
**la intuición «pon el servidor cerca del usuario» es incorrecta aquí.**

Neon **no tiene región en Japón**. La base estaba —y sigue— en `ap-southeast-1`
(Singapur). Con el servidor en Tokio:

- usuario ↔ servidor: **1 RTT** por petición
- servidor ↔ base: **N RTT** por petición, uno por query

Una pantalla de la app hace varias queries. Poner el servidor en Tokio para ahorrar
un RTT y pagar Tokio↔Singapur multiplicado por cada query sale perdiendo. Con el
servidor **al lado de la base**, ese multiplicador desaparece y solo queda el RTT
Japón↔Singapur una vez.

Es lo que explica el 1.541 ms → 35 ms: no es que Fly sea rápido, es que la query
dejó de cruzar el océano.

**Corolario publicable:** la latencia que importa no es la del usuario al servidor.
Es la del servidor a su dependencia más chismosa, multiplicada por lo chismosa que
sea. Y ese cálculo casi nadie lo hace antes de elegir región.

## Lo que salió mal (y es lo que la gente quiere leer)

1. **`prisma generate` no arranca sin `DATABASE_URL`.** `prisma.config.ts` la resuelve
   al cargarse, así que el build de Docker moría antes de generar nada — y `generate`
   no se conecta a ninguna base. Se arregla con una URL de mentira en el stage de build.
2. **El sistema de logs tumbó un despliegue.** Un handler `beforeExit` que hacía flush
   a Axiom: el evento se emite cuando el event loop se vacía, o sea cuando el proceso
   ya se va; el fetch revivía el loop, el socket moría a media petición y undici tiraba
   una excepción no capturada. Mató el `release_command` (`prisma migrate deploy`), y
   Fly lo leyó como migración fallida. **El sistema de observabilidad tirando la app**
   es la mejor anécdota del artículo.
3. **Los códigos ANSI llegaban a Axiom**: `--> GET /api/health \x1b[32m200\x1b[0m`.
   Buscar `200` no encontraba nada, que es justo lo que quieres buscar cuando algo falla.

## Tres bugs que la migración destapó sin buscarlos

Vale la pena una sección: mover de sitio obliga a mirar todo, y ahí aparecen cosas.

- **R2 estaba roto en producción.** Las `S3_*` solo existían en el entorno Preview, así
  que `image-proxy` devolvía 500 en prod. Nadie lo había notado.
- **`NEXT_PUBLIC_MARKETING_URL` apuntaba a un dominio muerto** (`…vercel.app`), o sea
  404 en el logo del login.
- **La base iba 18 días por delante del código**: migraciones aplicadas desde `develop`
  a la base de producción sin que `main` las tuviera.

## El artículo se partió en dos

Este cubre **dónde corre el servidor**. La segunda mitad —sacar la landing del
servidor a Cloudflare Pages— es una tesis distinta y va en
`notes/landing-off-the-server.md`. Juntos salían 6.000 caracteres y dos ideas
compitiendo.

## Ángulos que NO usar

- «Vercel es caro / malo». Es falso y se nota: el problema es el encaje, y el
  artículo pierde credibilidad en el primer comentario.
- Comparativa genérica de features Vercel vs Fly. Ya está escrita mil veces.
- Presentarlo como decisión irreversible. Lo bueno del ángulo es que es reversible
  y lo dices tú mismo.
- **«Migré a Tokio».** No se hizo, y decirlo destruiría el ángulo bueno: se eligió
  Singapur *a propósito*, contra la intuición, porque la base manda sobre el usuario.
