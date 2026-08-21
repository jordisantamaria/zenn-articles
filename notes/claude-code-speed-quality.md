# Notas — claude-code-speed-quality

No van dentro del `.md`: **Zenn imprime los comentarios HTML.** El guion inline del
esqueleto está en español a propósito; hay que borrarlo entero antes de publicar.

Estado: **esqueleto**, sin redactar. Creado el 2026-08-21.

## Títulos candidatos

1. `Claude Codeを8つ並列で回すと、最初に壊れるのはコードではなくGitでした`
   → el mejor. Concreto, contraintuitivo, y el problema es reconocible.
2. `速さはモデルではなく足場から来る — Claude Codeの周りに作った4つの仕組み`
   → más de tesis, menos clicable. Bueno si el anterior te parece negativo.
3. `AIに速く書かせるほど、品質は下がります。ゲートを置かない限り`
   → si decides que el eje sea 品質 en vez de 並列.

## El hueco: qué NO repetir

`after-launch-is-not-a-break.md` (todavía sin publicar) ya cubre en la sección
「どうやったか：Claude Codeを8つ並列で回す」:

- las 8 sesiones simultáneas y el bucle投げる→待たない→実機確認
- `/rename` para no perderse entre sesiones
- «dejé de escribir código y pasé a revisar»; el cuello de botella se mueve a 確認
- los dos techos: **límite semanal** del plan de 100 USD (8/5) y **RAM 32→64GB** (8/4)
- el tweet https://twitter.com/jordisantamar1a/status/2086286461981331520

**Este artículo empieza donde aquel acaba.** Aquel: qué hice. Este: qué infraestructura
lo sostiene. Si se solapan, el segundo sobra.

## Datos verificados (2026-08-21)

- 推しスキ móvil: **530 commits** desde el primer commit (2026-06-30) — 53 días.
- `~/.claude/skills/`: **57 skills**. `~/.claude/hooks/`: 3 (`block-prod-merge.sh`,
  `ask-before-dev-server.sh`, `ccws-map.sh`). `~/.claude/rules/`: 15 ficheros.
- `.claude/worktrees/` del repo móvil: 12 worktrees vivos ahora mismo
  (album-index, cheki-finish, feedback-*, posthog-cli-fix, store-review…).
- `scripts/`: `worktree-setup.sh`, `worktree-land.sh`, `release-draft-sync.sh`.
- `.husky/`: pre-commit, commit-msg, post-commit, post-merge.
- Skill de proyecto: `.claude/skills/ux-gate/SKILL.md`.

### Los incidentes que justifican cada pieza (están en el CLAUDE.md del repo)

| fecha | qué pasó | qué salió de ahí |
|---|---|---|
| 2026-08-19 | un `git add` se lo llevó otra sesión en su commit | commitear con `git commit -- <paths>` en un solo paso |
| 2026-08-20 | un commit dejó un `pick-image.ts` importando un módulo sin commitear → 3 sesiones bloqueadas | «commitea el conjunto que compila, no el fichero que tocaste» |
| 2026-08-20 | **nueve** sesiones a la vez sobre el mismo repo | worktree por sesión, obligatorio |
| 2026-08-21 | el hook `block-prod-merge.sh` bloqueó la escritura de ESTE artículo porque el texto citaba el comando prohibido | el falso positivo va en el artículo: enseña el coste de los hooks sin teorizar |

## Pendientes que bloquean la redacción

- [ ] Decidir cuánto código pegar. Cuatro scripts enteros lo convierten en un
      README; el artículo necesita 2 bloques de código buenos, no 6 mediocres.
- [ ] Captura de `git worktree list` con las 8-12 ramas vivas → imagen principal.
- [ ] Captura del body de la PR de release generado solo por el hook.
- [ ] ¿El `/ux-gate` ha parado algo real? Si no hay caso, decirlo en vez de sugerirlo.
- [ ] Revisar si algo del setup es específico de tu máquina y no reproducible
      (rutas absolutas, wezterm, `ccws`). Lo que no se pueda copiar, fuera.

## Riesgo del artículo

Es el que más fácil se lee como 自慢 («mira mi setup»). El antídoto está en la
estructura: **cada sección empieza por el fallo y acaba en la pieza**, nunca al
revés. Un lector que no monte nada de esto tiene que salir habiendo aprendido los
tres modos de pisarse en git — eso es valor aunque no copie una línea.

## 記事の芯（2026-08-21 に整理）

GTDの「2分ルール」（2分で終わるならメモせず今やる、David Allen『Getting Things Done』）
※Clean Codeではない。Clean Codeのほうは Boy Scout Rule（来たときより綺麗にして帰る）。

- 元のルール：**速いなら**今やる。遅いものはメモ＝バックログに積む。
- メモのコスト：1週間後にも残っている。そして再開時に文脈を思い出す時間がかかる。
- Claude Codeで変わること：**「速いなら」という条件が消える。** 2時間かかるタスクも、
  思いついた瞬間にセッションへ投げれば、バックログに積まずに片付く。
  だからセッションはどんどん増やしていい。
- ただし正直に書くべき反面：**ボトルネックは「書く時間」から「確認する時間」へ移る。**
  8つ同時に走っても、実機で触れるのは一度に1つ。生成が速くなっても検証は速くならない。
  → この記事の結論は「速くなった」ではなく「待ち時間がゼロになった」。

キーワード：`claude code 並列`（サジェスト10件 = 今回測ったなかで最大の需要）。
やり方 / 実行 / 起動 / エージェント / 作業 / 開発 / 実装 / 処理 / 化 — タイトルに入れる候補。
