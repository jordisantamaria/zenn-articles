# Free tiers verificados — infra de oshisuki

Todo verificado contra las **páginas oficiales** el **2026-08-27**. Cada fila lleva su
fuente. Si un dato no está aquí, no está verificado: no lo publiques sin comprobarlo.

> ⚠️ La página de *pricing* y la *landing* de un producto **se contradicen a menudo**.
> Pasó con Better Stack (ver abajo). Cuando no coincidan, gana la que describe el plan
> gratuito de forma explícita, y se cita esa.

> ⚠️ **La negrita japonesa se rompe justo antes de un `$`.** `4社は**$12から$30**に収まります`
> sale con los asteriscos literales: por la regla *left-flanking* de CommonMark, un `**`
> pegado a un signo de puntuación (`$`, `%`, `」`) y precedido de un carácter normal no
> abre énfasis. Lo mismo al cerrar: `**96%**になっていました` tampoco funciona.
> La salida es meter el kanji dentro (`**4社とも$12〜$30**`) o dejar un espacio.
> **Comprobación**: no basta con `npx zenn preview`, porque la página es un shell de Next.
> Hay que pedir el HTML ya renderizado y buscar asteriscos sueltos:
>
> ```bash
> curl -s "http://localhost:8000/api/articles/<slug>" \
>   | python3 -c "import json,sys,re; h=json.load(sys.stdin)['article']['bodyHtml']; \
>       print(len(re.findall(r'[*]', re.sub(r'<pre.*?</pre>|<code.*?</code>','',h,flags=re.S))))"
> ```

## Hosting de la aplicación

| | Mínimo siempre-encendido | €/mes | Tokio | Config |
|---|---|---|---|---|
| Vercel Hobby | **imposible** (solo funciones) | $0 (4 h CPU) | ○ `hnd1` | dashboard |
| Vercel Pro | **imposible** | $20/persona | ○ | dashboard |
| Cloudflare Workers | **imposible** | $5 | global | `wrangler.toml` |
| Render Free | **✗ duerme a los 15 min** | $0 | **✗** | `render.yaml` |
| Render Starter | 0.5 CPU / 512 MB | ~$7 | **✗** | `render.yaml` |
| Railway Hobby | por uso | $5 (incl. $5 crédito) | ✗ | dashboard |
| **Fly.io** | `shared-cpu-1x` 512 MB | **$3.32** | **○ `nrt`** | `fly.toml` |
| AWS ECS Fargate + ALB | sí | **≈ $27** (ALB 17,74 + task 9,00) | ○ | Terraform / CDK |

- Fly.io: 256 MB $2.02 · 512 MB $3.32 · 1 GB $5.92 · 2 GB $11.11. Egress APAC $0.04/GB.
  <https://fly.io/docs/about/pricing/>
- **Fly.io ya no tiene free tier.** Los planes (3× `shared-cpu-1x` 256 MB gratis) se
  descontinuaron el **2024-10-07**; solo se honran en cuentas anteriores. Las nuevas
  reciben un trial de "2 hours of machine runtime or 7 days, whichever comes first".
  ⚠️ **Jordi tiene cuenta legacy**, así que su factura NO es replicable por un lector.
  <https://fly.io/docs/about/discontinued-plans/> · <https://fly.io/docs/about/free-trial/>
- Render Free duerme: "spins down … 15 minutes without … traffic" y tarda "about one
  minute" en despertar. <https://render.com/docs/free>
- Render regiones: Oregon / Ohio / Virginia / Frankfurt / Singapore. **Sin Japón.**
  <https://render.com/docs/regions>
- Railway: memoria $0.0139/GB-h, CPU $0.0278/vCPU-h, egress $0.05/GB.
  <https://railway.com/pricing>
- Cloudflare Workers free: 100.000 req/día y **10 ms de CPU por invocación** (no llega
  para Next.js + Prisma + auth). De pago $5/mes.
  <https://developers.cloudflare.com/workers/platform/pricing/>
- **AWS en Tokio (`ap-northeast-1`), verificado contra la Price List API el 2026-08-27.**
  Las páginas de precios de AWS son JS y no sirven para verificar; la fuente buena es
  `https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/<SERVICIO>/current/ap-northeast-1/index.json`.
  - **ALB: $0.0243/hora** (`APN1-LoadBalancerUsage`, "per Application LoadBalancer-hour")
    → **$17,74/mes con tráfico cero**. Ojo: **us-east-1 son $0.0225**, y citar ese precio
    en una fila que dice «Tokio ○» es el error que tenía el artículo.
  - LCU: **$0.008/LCU-hora** (`APN1-LCUUsage`). A 1,7 M req/mes ≈ 0,65 req/s es despreciable.
  - **Fargate ARM: $0.04045/vCPU-h + $0.00442/GB-h** (`APN1-Fargate-ARM-*`).
    x86: $0.05056 y $0.00553. La tarea mínima (0,25 vCPU / 0,5 GB, ARM) = **$9,00/mes**.
  - Mínimo real de entrada: **$26,74/mes ≈ 8× Fly.io**.
  - 3er año (2 tareas de 0,5 vCPU / 1 GB ARM + ALB) ≈ **$55/mes**.
  - El tipo de lanzamiento EC2 **no lo abarata**: pagas la instancia 24 h y el ALB igual.
  - ⚠️ **NO verificado**: si AWS cobra las IPv4 públicas ($0.005/h) de un ALB
    internet-facing. El anuncio oficial no lo dice de forma explícita y el resto de
    fuentes son blogs, así que se ha dejado FUERA del artículo.
    <https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/>
- Vercel Active CPU por región: `iad1` $0.128/h · **`hnd1` $0.202/h (+58%)** ·
  `kix1` $0.202 · `sin1` $0.160 · `icn1` $0.169.

## Base de datos

Los cuatro ejes que importan para producción: **rendimiento, precio, recuperación
(backups) y si se puede consultar desde una IA.** No se entra en SQL vs NoSQL.

Criterio de la comparación: Postgres gestionado + gratis o <$10/mes + sin montar VPC.

| | Región más cercana a Japón | Free: rendimiento y parada | Precio | **Backups** | MCP |
|---|---|---|---|---|---|
| **Neon** | **Singapur** (105 ms medidos) | 0.5 GB / 0.25 CU fijo, cero a los **5 min** | $0 → $0.106/CU-h | **PITR 6 h** (pago: 1 día → 7 → 30) | **oficial** |
| Supabase | **Tokio** | CPU compartida / 500 MB, **pausa a la semana** | $0 → **$25/mes** | **NINGUNO en free** | **oficial** |
| Fly MPG | **Tokio `nrt`** | sin free tier | **$38/mes** | automáticos incluidos | CLI |

⚠️ **Neon NO es "sin Japón" a secas: tiene Singapur, a 105 ms.** Escribirlo como ✗ en una
columna "¿Japón?" oculta el dato del que va todo el capítulo 4.

Descartados, con motivo:

- **AWS RDS / Aurora** — el free tier es **temporal**: "up to 6 months" para cuentas
  creadas después del 2025-07-15, 12 meses para las anteriores; luego pay-as-you-go.
  Un free tier con fecha de caducidad no sirve de base para un producto que sigue vivo.
  Además arrastra VPC y security groups. <https://aws.amazon.com/rds/free/>
- **PlanetScale** — ya no tiene free tier; el más barato es **PS-5 non-HA a $5/mes**.
  Ahora ofrece **Postgres**, no solo MySQL. <https://planetscale.com/pricing>
- **Railway Postgres** — cabe en el mismo plan de $5, pero sin región japonesa y con
  condiciones de backup más flojas.

- ⚠️ **Supabase SÍ tiene backups — pero solo de pago.** Por plan: Free **nada**;
  Pro ($25/mes) diarios con **7 días**; Team **14 días**; Enterprise **30 días**. PITR
  es add-on de pago en todos (y al activarlo **dejan de hacerse los diarios**).
  Su doc para el free: "We recommend that free tier plan projects regularly export their
  data using the Supabase CLI `db dump` command and maintain off-site backups".
  No escribir "Supabase no tiene backups" a secas: es falso.
  <https://supabase.com/docs/guides/platform/backups>
- Neon PITR: **6 horas en Free**, 1 día en planes de pago, configurable hasta 7 días
  (Launch) o 30 (Scale). <https://neon.com/docs/introduction/point-in-time-restore>
- MCP oficial de Neon: hosted en `mcp.neon.tech`, hace SQL, ramas, comparar esquemas y
  cambios de esquema vía rama temporal. <https://neon.com/docs/ai/neon-mcp-server>
- MCP oficial de Supabase: hosted en `mcp.supabase.com/mcp`, SQL, Edge Functions, tipos
  TypeScript, advisors. <https://supabase.com/docs/guides/getting-started/mcp>
- Fly MPG incluye "automatic backups, high availability, monitoring, scaling, 24/7
  support, and encryption" — ventana concreta **sin verificar** (la URL
  /docs/mpg/backup-restore/ da 404).

- Neon regiones (8, **sin Japón**): US×3, Frankfurt, London, **Singapore**, Sydney,
  São Paulo. Y **la región no se puede cambiar**: "You cannot change the region for an
  existing project." <https://neon.com/docs/introduction/regions>
- Neon de pago (Launch): $0.106/CU-hora + $0.35/GB-mes, sin cuota fija.
  <https://neon.com/pricing>
- Supabase Free: 500 MB, se pausa tras 1 semana de inactividad, máx. 2 proyectos
  activos. Pro "from $25/month". <https://supabase.com/pricing>
- **Fly Managed Postgres SÍ tiene Tokio (`nrt`)**, pero el plan más barato es
  **$38/mes** (Basic, shared-2x, 1 GB) + $0.28/GB de almacenamiento. Por eso no se usa
  para arrancar: es 10× el coste de toda la infra actual. <https://fly.io/docs/mpg/>
- Regiones con MPG: ams, dfw, fra, iad, lax, lhr, **nrt**, ord, sin, sjc, syd, yyz.
  <https://fly.io/docs/reference/regions/>

## Observabilidad

| Capa | Servicio | Free |
|---|---|---|
| Logs | **Axiom** | **500 GB/mes, 30 días** |
| Logs (alt.) | Grafana Cloud | 50 GB/mes, 14 días |
| Logs (alt.) | Better Stack Telemetry | 3 GB/mes, **3 días** |
| Producto + errores | PostHog | 1 M eventos/mes |
| Uptime | **Better Stack** | 10 monitores + **10 heartbeats** + status page |
| Uptime (alt.) | UptimeRobot | 50 monitores |

- Axiom Personal: "500 GB / mo data loading", "30-day retention", sin tarjeta.
  Siguiente tier: $25/mes + uso. <https://axiom.co/pricing>
- **⚠️ Better Stack: el free son checks de 3 MINUTOS, no de 30 segundos.**
  La landing es la fuente clara: "Get 10 monitors, 10 heartbeats and a status page with
  **3-minute checks** totally free". Los **30-second checks son de pago**. La página de
  /pricing dice "up to 30 seconds check frequency" sin aclarar que es del plan pagado, y
  por eso se coló el dato equivocado una vez. <https://betterstack.com/uptime>
- Better Stack logs de pago: Nano $45/mes (40 GB, 30 días). <https://betterstack.com/pricing>
- UptimeRobot free: 50 monitores, intervalo de **5 min**. Solo: $9/mes con 60 s.
  <https://uptimerobot.com/pricing/>
- Grafana Cloud free: logs 50 GB/mes con 14 días; métricas 10k series; trazas 50 GB.
  <https://grafana.com/pricing/>
- Vercel Hobby retiene logs **1 hora**, y Observability solo deja mirar **12 h atrás**
  (24 h pide Pro; 3–30 días pide Observability Plus). Verificado en el panel el
  2026-08-27: por eso el 42,3% de cold starts del 22 de agosto **ya no es recuperable**.

## Medidas propias (producción, 2026-08-22)

| | Antes (Vercel `iad1`) | Después (Fly `sin`) |
|---|---|---|
| Query a la BD | 1.541 ms | **35 ms** |
| Petición completa | ~1,9 s | ~0,35 s |
| Cold starts | 42,3 % | 0 |
| Active CPU P75 | 593 ms | — |

- RTT Japón → Neon Singapur: **105 ms**.
- Reparto del cupo del team (30 días, visto el 2026-08-27): oshisuki **2 h 40 m = 74,1 %**.
- Cronología: correo del 75 % el **21 ago 19:46** → 96 % a la mañana siguiente (11 h) →
  aviso de mantenimiento **22 ago 8:03** → terminado **11:01** (2 h 58 min).

## Pendiente de verificar

- [ ] La configuración real de Better Stack de Jordi (qué endpoints vigila, a qué
      intervalo, cómo avisa). Requiere login: **no accesible sin él**.
- [ ] Si alguna vez ha saltado una alerta real antes de que la viera un usuario.
