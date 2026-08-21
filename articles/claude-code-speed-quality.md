---
title: "TODO: Claude Codeを8つ並列で回しても壊れない仕組みの話（仮題 — 候補は notes/）"
emoji: "⚙️"
type: "tech"
topics: ["claudecode", "個人開発", "ai", "git", "開発生産性"]
published: false
---

<!-- ⚠️ ESQUELETO — NO PUBLICAR TAL CUAL.
     Zenn IMPRIME los comentarios HTML: borrar este bloque y todas las líneas
     «> ES:» / «> TODO:» antes de poner published: true.
     Datos verificados y pendientes: notes/claude-code-speed-quality.md -->

> ES: **El hueco exacto de este artículo.** El artículo anterior
> (`after-launch-is-not-a-break`) ya contó QUÉ hiciste: 8 sesiones en paralelo,
> /rename, tú pasas de escribir a revisar, y los dos techos (límite semanal + RAM).
> **No repitas nada de eso.** Este artículo cuenta el CÓMO: qué infraestructura hace
> que ocho sesiones sobre el mismo repo no se destruyan entre ellas, y qué le pones
> alrededor para que la velocidad no baje la calidad.
>
> Titular en una frase: **「速さはモデルではなく、足場から来る」**.
> Enlaza al anterior en はじめに con una línea y sigue. No lo resumas.

## はじめに

> ES: Entrar por donde el lector ya está: todo el mundo ha probado Claude Code y
> todo el mundo va rápido los primeros días. La pregunta que nadie responde en
> japonés es qué pasa cuando el volumen sube de verdad.
> Datos verificados que puedes usar aquí (ver notes):
>   - 530 commits en 推しスキ entre el 2026-06-30 y el 2026-08-21 (53 días)
>   - hoy hay 57 skills en `~/.claude/skills/`
> Y decir el precio de entrada: **esto no lo montas el primer día. Cada pieza de
> abajo nació de una vez que algo se rompió.** Ese es el tono honesto y es el que
> hace que el artículo no se lea como 自慢.

## 並列で回すと、最初に壊れるのはコードではなくGitです

> ES: La primera parte, y la más útil: los tres modos en que 8 sesiones sobre UN
> mismo árbol se pisan. Los tres están vividos, con fecha (ver notes):
>
>   1. **El índice de git es compartido.** Tu `git add` se lo lleva la otra sesión
>      en SU commit. Pasó el 2026-08-19.
>   2. **El pre-commit corre `pnpm type-check` sobre todo el proyecto.** El refactor
>      a medias de otra sesión te bloquea el commit aunque tus ficheros estén bien.
>   3. **Dos sesiones editando el mismo fichero**: uno de los dos cambios se pierde
>      en silencio.
>
> Esto es lo que hay que enseñar con detalle, porque es contraintuitivo: la gente
> asume que el problema de la IA en paralelo es que escribe mal. El problema real
> es de concurrencia, y es un problema de git, no de modelo.
>
> TODO: si tienes un commit real donde se ve el fichero ajeno colado, enlázalo.
> Vale más que la explicación.

### 解決策：セッションごとにgit worktreeを持たせる

> ES: La solución, con el flujo real. Enseñar los 4 comandos tal cual:
>
> ```
> EnterWorktree                        # el nombre es la tarea: "cheki-crop"
> bash scripts/worktree-setup.sh       # enlaza node_modules, .env, expo-env.d.ts
> …editar y commitear ahí dentro…
> ExitWorktree (keep)
> bash scripts/worktree-land.sh <rama> # rebase + avance de develop
> ```
>
> Los dos scripts son cortos y son la mitad del valor del artículo:
>   - `worktree-setup.sh`: `node_modules` va **enlazado, no copiado** (8 copias de
>     node_modules de React Native no caben, y además `.env` y `expo-env.d.ts` están
>     gitignored, así que un worktree recién nacido no compila sin esto).
>   - `worktree-land.sh`: rebasa sobre `develop`, aborta al primer conflicto dejando
>     el worktree como estaba, y **se niega** a aterrizar si hay cambios sueltos.
>
> TODO: pegar los dos scripts (o los trozos que importan) desde
> `/mnt/data/sideprojects/oshisuki-mobile/scripts/`. Son públicos, el repo es tuyo.
> TODO: captura de `git worktree list` con 8 ramas vivas. Es la imagen del artículo.

### worktreeが解決しないこと：`develop`が壊れたとき

> ES: El matiz que hace que el lector confíe en ti. Los worktrees nacen de `develop`;
> si `develop` no compila, **ninguno** compila y nadie puede commitear en ninguna
> parte, porque el pre-commit typechequea el proyecto entero.
> Pasó el 2026-08-20: un commit se llevó un `pick-image.ts` que importaba un módulo
> y un tipo que se quedaron sin commitear → tres sesiones bloqueadas a la vez.
> La regla que salió de ahí: **commitea el conjunto que compila, no el fichero que
> tocaste.** Si tu cambio estrena un tipo, entra en el mismo commit que quien lo usa.

## 2つめ：CLAUDE.mdは説明書ではなく、契約書です

> ES: El giro conceptual del artículo. La mayoría de los CLAUDE.md japoneses que se
> ven por ahí son 「このプロジェクトはNext.jsです」— información que el modelo ya
> deduce leyendo el repo. Eso no cambia ningún comportamiento.
> El tuyo escribe **decisiones que el repo no puede contar**, y sobre todo el
> POR QUÉ, porque una regla sin motivo se salta en cuanto el caso no encaja.
> Ejemplo real y citable (está en el repo público): la sección de por qué worktree
> lleva los tres modos de pisarse Y la fecha en que pasó cada uno.
>
> TODO: pegar 15 líneas del CLAUDE.md real de 推しスキ. La sección
> 「main la mergea Jordi, nunca Claude」 es la mejor para enseñar, porque muestra
> los ✅/❌ explícitos.

## 3つめ：お願いではなく、フックで止める

> ES: El punto más práctico y el que menos se escribe. Una regla en CLAUDE.md es una
> petición; un hook es una pared. La distinción importa cuando corres en paralelo,
> porque no estás mirando.
> Los tuyos, reales (`~/.claude/hooks/`):
>   - `block-prod-merge.sh` → PreToolUse, **deniega** los comandos que publican
>     (el merge del PR a la rama de producción y el push directo a esa rama) en los
>     repos de oshisuki. El motivo: esa rama dispara el deploy del backend y es de la
>     que sale el binario de la tienda. Con usuarios reales, publicar dejó de ser una
>     operación de desarrollo.
>   - `ask-before-dev-server.sh` → evita que una sesión levante un segundo Metro en
>     el 8081. El móvil apunta al principal; un Metro extra solo choca de puerto.
>   - `.husky/pre-commit` → `tsc --noEmit` + `eslint --fix` sobre lo staged, y
>     bloquea los commits directos a la rama de producción.
>
> ES: La frase que resume la sección: **CLAUDE.mdは「やらないで」、フックは「できない」。**
> 並列で回すときに効くのは後者です。
>
> ES: Y una anécdota que salió sola escribiendo este mismo esqueleto, el 2026-08-21:
> el hook `block-prod-merge.sh` **bloqueó la escritura de este artículo**, porque el
> texto contenía el comando prohibido como ejemplo. Es el falso positivo perfecto
> para la sección: un PreToolUse que mira el comando por patrón no distingue entre
> ejecutarlo y escribir sobre él. Vale la pena contarlo — enseña el coste real de los
> hooks sin tener que teorizar, y de paso demuestra que la pared existe de verdad.
>
> TODO: pegar el hook `block-prod-merge.sh` entero si es corto — un PreToolUse real
> y funcionando es exactamente lo que un lector de Zenn se lleva y copia.

## 4つめ：速さは品質を下げます。ゲートを置かない限り

> ES: Aquí va la parte de 品質, que es lo que prometes en el título y lo que casi
> ningún artículo de Claude Code entrega. La tesis: generar más rápido no baja la
> calidad del código *generado*; baja la calidad de lo que **revisas**, porque el
> cuello de botella eres tú (ya lo dijiste en el artículo anterior — una línea de
> enlace, no lo repitas). La respuesta no es revisar más, es automatizar el criterio.
>
> Tu ejemplo real: la skill `/ux-gate` en `.claude/skills/` del repo móvil. Corre
> DESPUÉS de construir cualquier pantalla y ANTES de que llegue a ti, y puntúa
> contra criterios con umbral duro sacados de **bugs reales ya vividos**: teclado,
> safe area, los tres estados (vacío/cargando/error), errores tragados, a11y.
> El punto que lo hace interesante: **los criterios no salen de una guía de estilo,
> salen del historial de fallos del propio repo.** Cada bug que te llega al móvil se
> convierte en una línea del gate para que no vuelva.
>
> TODO: pegar 10 líneas de `ux-gate/SKILL.md` (los criterios con umbral).
> TODO: ¿tienes un caso donde el gate paró algo antes de llegarte? Si sí, es el
> ejemplo. Si no, dilo — «todavía no tengo métrica de cuántos paró» es honesto y
> mejor que inventar.

### 自動化のもうひとつの形：コミットが勝手に仕事をする

> ES: La pieza que más sorprende y que es tuya. `.husky/post-commit` y `post-merge`
> lanzan `scripts/release-draft-sync.sh` en background, que llama a una skill
> (`/release-draft`) que actualiza la descripción de la PR de release con lo que
> aportan los commits nuevos. O sea: **una skill de Claude Code disparada por un
> hook de git, sin que nadie la invoque.**
> Y las dos reglas de diseño que la hacen segura, que es lo que hay que contar:
>   1. **Solo añade.** Un `- [x]` no se desmarca, no se reescribe y no se borra:
>      es QA ya pasada en un dispositivo real. Perderla = repetirla sin saberlo.
>   2. **Nunca toca git ni el árbol de trabajo, y sale siempre en verde.** No puede
>      tumbarte un commit. Un automatismo que puede romperte el flujo se acaba
>      desactivando; uno que solo puede añadir, no.
>
> TODO: captura de la PR de release con el body generado solo. Muy visual.

## それでも自分でやること

> ES: Sección corta pero obligatoria para que el artículo no se lea como
> 「AIに全部やらせています」. Lo que NO delegas y por qué:
>   - la publicación a producción (lo decide una persona mirando el PR — de hecho
>     está escrito en el hook, no solo en la cabeza)
>   - probar en el móvil real
>   - decidir qué se construye
> TODO: ¿algo más? Piensa si hay decisiones de producto que ya delegas y no deberías.

## おわりに

> ES: Cerrar con la lista corta y accionable, ordenada por retorno para alguien que
> empieza mañana. Mi recomendación de orden:
>   1. worktree por sesión (sin esto, lo demás da igual)
>   2. CLAUDE.md con los POR QUÉ, no con la descripción del stack
>   3. un hook que deniegue lo irreversible
>   4. un gate de calidad hecho con tus propios bugs
> Y la frase de cierre: el modelo es el mismo para todos. Lo que no es igual es el
> andamiaje, y eso lo construyes tú una vez y te sirve en todos los proyectos.
>
> TODO: enlazar el artículo del harness cuando exista: este llega hasta «yo reviso»,
> y el siguiente pregunta qué pasa cuando ni eso.
