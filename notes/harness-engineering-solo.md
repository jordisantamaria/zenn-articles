# Notas — harness-engineering-solo

No van dentro del `.md`: **Zenn imprime los comentarios HTML.** El guion inline del
esqueleto está en español a propósito; hay que borrarlo entero antes de publicar.

Estado: **esqueleto**, sin redactar. Creado el 2026-08-21.

## ⚠️ Confidencialidad — leer antes de escribir

Este es el único artículo de la serie que toca **material de un cliente**. El
harness de Cierpa se anunció internamente en Slack (`#dev-team_questionnaire_new_infrastructure`,
Shinichi Ueno, 2026-07-03: issue → harness → Q&A → corrección → rama → PR →
reviewer asignado al autor del issue). Eso es comunicación interna, no material
publicable.

Regla para redactar:

- ✅ Se puede: la **forma general** del patrón (Issue → labels → agentes → PR),
  que ya es pública en los artículos de referencia de abajo.
- ❌ No, sin permiso por escrito: nombre del cliente, del producto, de los repos,
  de los workflows, capturas, métricas internas, o quién lo montó.
- Si quieres concretar, pregunta a Ueno-san **antes**, no después. Y si no hay
  respuesta, el artículo funciona igual sin esa sección — está diseñado así.

Coste/beneficio: unas cuantas いいね no compensan un problema con un cliente cuyo
contrato es tu visado.

## Títulos candidatos

1. `ハーネスエンジニアリングを、ひとりでやってみた — 会社の21体と、個人開発の私`
   → el contraste es el gancho, y es tuyo.
2. `個人開発にハーネスはどこまで必要か。「強制」は要る、「可視化」は要らない`
   → el más afilado, y la tesis va en el título.
3. `AIエージェント自律開発のデメリットを、実際に払った人間が書きます`
   → si decides que el eje sean las desventajas. Es el hueco más vacío del tema.

## Fuentes públicas (verificadas 2026-08-21)

- **aicon_kato — 21 agentes, pipeline completo**
  https://zenn.dev/aicon_kato/articles/harness-engineering-startup
  Lo que aporta: proceso descompuesto en pasos, labels como bus de agentes,
  tabla de los 21 workflows, CI/CD, entornos preview con Terraform, scoring
  periódico. Paró el desarrollo **todo febrero** para montarlo, y las **2 primeras
  semanas** no produjeron nada. Ese dato es el más útil para tu sección de costes.
  ⚠️ Es el artículo con el que compites. Enlázalo y no intentes cubrir lo mismo.
- **Acsim — ハーネスエンジニアリング**
  https://ai.acsim.app/articles/harness-engineering-2026
  De aquí salen: las dos definiciones (Hashimoto = bucle de detección de fallos;
  OpenAI = diseño del entorno entero), el origen de la palabra (馬具), «Blueprints»
  de Stripe (「モデルがシステムを動かすのではない。システムがモデルを動かす」),
  el Ralph Wiggum loop, el 80% de Stripe, y que Anthropic borra piezas del harness
  con cada modelo nuevo.
- **DORA 2025**: PRs +98%, review time +91%, tamaño de PR +154%.
  ⚠️ Estos números los tomé del artículo de Acsim, **no de la fuente primaria**.
  Verificar en el informe DORA antes de citarlos. Un número mal atribuido en un
  artículo sobre rigor es el peor sitio para tenerlo.

## Tu propio material (sin riesgo, todo tuyo)

- Skills tipo harness ya funcionando: `/bg` (crea el worktree y lanza un sub-Claude
  en background), `/solve` (issue/ticket → rama → draft PR → limpieza del worktree),
  `/continue`, `/bg-status`, `/task-tree`. Es la misma forma que el pipeline
  corporativo, sin GitHub Actions.
  ⚠️ Viven en el repo de trabajo, no en `~/.claude`. Si son propiedad del cliente,
  describe el patrón sin pegar el código.
- Harness propio en 推しスキ: `.husky/post-commit` → `/release-draft` (una skill
  disparada por un hook de git, sin que nadie la invoque), `/ux-gate`, los
  PreToolUse que deniegan lo irreversible.
- Plugin `ralph-loop` instalado — el mismo Ralph Wiggum del artículo de Acsim.

## Costes vividos, con fecha (la sección que distingue el artículo)

| fecha | qué | por qué importa |
|---|---|---|
| 2026-08-04 | RAM 32→64GB | el paralelismo topa con la máquina, y eso solo se resuelve con dinero |
| 2026-08-05 | límite **semanal** del plan de 100 USD | y para también el trabajo, no solo el hobby. Un pipeline autónomo lo quema aunque duermas |
| 2026-08-19 | `git add` de una sesión se lo llevó otra en su commit | el aislamiento no es opcional |
| 2026-08-20 | `develop` roto → 3 sesiones sin poder commitear | **cuanto más fuerte el gate, más caro su fallo** |
| 2026-08-21 | el hook anti-publicación bloqueó la escritura de este propio artículo (el texto citaba el comando) | los gates por patrón no distinguen ejecutar de mencionar |

El último es el mejor ejemplo del género y salió solo, escribiendo esto.

## Pendientes que bloquean la redacción

- [ ] **Permiso para hablar del sistema del cliente** (o decidir escribirlo en
      abstracto, que es la opción por defecto).
- [ ] Verificar los números del DORA en la fuente primaria.
- [ ] Recuperar la definición real de `/bg` y `/solve` para describirlas bien.
- [ ] Buscar el ejemplo que le falta al género entero: **un PR autónomo que parecía
      correcto y estaba mal.** Si lo tienes, es el centro del artículo.
- [ ] Decidir si este es el artículo 5 o si adelanta al de Claude Code. Ver abajo.

## Orden en la serie

Este debería ir **después** de `claude-code-speed-quality`, porque aquel monta las
piezas (worktree, hooks, gates) y este pregunta qué pasa cuando el disparo también
se automatiza. Publicarlo antes obliga a explicar dos veces lo mismo.
