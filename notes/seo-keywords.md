# Registro SEO — keyword objetivo por artículo

Medido con `gsuggest` (sugerencias reales de Google JP). **Una sugerencia que aparece
= hay gente tecleándola.** Si no sale nada, esa frase no la busca nadie.

Herramientas (en `~/.local/bin/`):

- `gsuggest "prefijo"` — ¿alguien busca esto? (`-e` expande con a-z / あ-わ)
- `gserp "query"` — abre la SERP JP despersonalizada para comprobar posición **a mano**
- `gtrends "a" "b"` — ¿crece o decrece? (Trends JP, 5 años)

Lo que **no** se puede saber: qué query trajo cada visita. Zenn no da fuente y sin
Search Console sobre zenn.dev no hay forma. El volumen absoluto solo lo da Google
Keyword Planner (cuenta de Google Ads gratis, sin campaña → rangos tipo 100〜1000).

## Estado

| Artículo | Keyword objetivo | Evidencia (sugerencias) | Estado |
|---|---|---|---|
| `30days-to-app-store` | — (tarjeta de LinkedIn, sin query) | `アプリ リリース後` → **0** | publicado 2026-08-11 |
| `after-launch-is-not-a-break` | `app store 審査 長い` / `時間` | 4 (`長い`, `時間`, `土日`, `厳しい`) | por publicar |
| `build-in-public-distribution` | sin medir | — | por medir |
| `claude-code-speed-quality` | **`claude code 並列`** | **10** (`やり方`,`実行`,`起動`,`エージェント`,`開発`…) | **la mejor del backlog** |
| `harness-engineering-solo` | **`ハーネスエンジニアリング`** | **10** (`とは`,`ベストプラクティス`,`本`,`claude`…) | término emergente |
| `aws-saa-in-two-weeks` | `aws saa 勉強時間` | 5 (`未経験`, `clf`, `sap`…) | `aws saa 2週間` solo se devuelve a sí misma |

## Nicho vacío (medido 2026-08-21) — no titular por aquí

`アプリ リリース後` · `android版 リリース` · `expo 個人開発` · `React Native 個人開発`
(esta última solo se devuelve a sí misma)

Coherente con que el artículo 1 fuese **primero en los tags reactnative / expo /
googleplay / buildinpublic toda una semana y aun así hiciera 117 visitas**: esos
topics no tienen gente.

## Comprobación de posición

Cada 2 semanas, `gserp "<keyword>"` en incógnito y anotar el puesto:

| Fecha | Keyword | Puesto | Visitas del artículo |
|---|---|---|---|
| 2026-08-21 | `30日でアプリを出す` | 1 | 117 (casi todas de X) |
