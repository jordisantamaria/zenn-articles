---
title: "TODO: ハーネスエンジニアリングを、ひとりで試したら（仮題 — 候補は notes/）"
emoji: "🐴"
type: "idea"
topics: ["claudecode", "aiエージェント", "githubactions", "個人開発", "開発生産性"]
published: false
---

<!-- ⚠️ ESQUELETO — NO PUBLICAR TAL CUAL.
     Zenn IMPRIME los comentarios HTML: borrar este bloque y todas las líneas
     «> ES:» / «> TODO:» antes de poner published: true.
     Datos, fuentes y el aviso de confidencialidad: notes/harness-engineering-solo.md
     ⚠️⚠️ LEE EL AVISO DE CONFIDENCIALIDAD DE LAS NOTAS ANTES DE ESCRIBIR UNA LÍNEA. -->

> ES: **El problema de este artículo, y hay que resolverlo antes de escribirlo.**
> El tema ya está muy bien cubierto en japonés. El artículo de referencia
> (aicon_kato, 21 agentes, pipeline entero) es exhaustivo y va a salir en cualquier
> búsqueda tuya. Escribir 「ハーネスエンジニアリングとは何か」 genérico es competir
> de frente y perder.
>
> **Tu ángulo, que nadie más tiene:** estás en los dos lados a la vez. De día, dentro
> de una empresa japonesa que está montando el harness de verdad. De noche, un
> 個人開発者 con un mini-harness de una persona. La pregunta que sale de ahí y que
> nadie ha escrito: **¿qué parte del harness corporativo sirve cuando eres uno solo,
> y cuál es puro coste?**
>
> Y la segunda mitad, que también falta en el material japonés: las **desventajas**,
> contadas por alguien que las ha pagado. Casi todo lo publicado son casos de éxito.

## はじめに

> ES: Entrar por la escena concreta, no por la definición. Algo así:
> «この半年で、二つのことが同時に起きました。仕事のチームでは、Issueを立てると
> エージェントが実装してPRまで持ってくる仕組みが動き始めました。個人開発では、
> 私はいまだにセッションを8つ手で開いています。」
> Y la pregunta del artículo en una línea: 個人開発にハーネスはどこまで必要か。
>
> TODO: verificar que la escena de trabajo se puede contar así. Ver el aviso de
> confidencialidad en notes/.

## ハーネスエンジニアリングとは（最短で）

> ES: Definición corta y con fuentes, sin extenderse — el lector que quiera el
> tratado va al artículo de aicon_kato, y tú lo enlazas honestamente.
> - Mitchell Hashimoto: 「エージェントのミスを検出・防止するフィードバックループ」
> - OpenAI: 「エージェントが最初から正しく動ける環境全体の設計」
> - El origen de la palabra: 馬具 — no le das más fuerza al caballo, la diriges.
> - Y el punto que sí conviene subrayar porque conecta con TU artículo anterior:
>   **ya estabas haciendo harness sin llamarlo así.** Un pre-commit hook, un linter
>   con mensaje de error útil, un CLAUDE.md: eso ya es harness. Lo nuevo es
>   rediseñarlo asumiendo que el lector de esos mensajes es un agente, no tú.
>
> ES: Enlazar aquí, y de verdad, no de paso:
>   - https://zenn.dev/aicon_kato/articles/harness-engineering-startup
>   - https://ai.acsim.app/articles/harness-engineering-2026
> Enlazar al que te gana en profundidad da credibilidad; esconderlo se nota.

## なぜいま出てきたのか：ボトルネックが移動したから

> ES: El dato que explica por qué el harness aparece ahora y no en 2024. Del DORA
> 2025 (≈5,000 respuestas), vía el artículo de Acsim — **cítalo desde el DORA, no
> desde el blog, y verifica los números en la fuente primaria** (ver TODO en notes):
>   - PRs mergeados: +98%
>   - tiempo de code review: +91%
>   - tamaño de los PRs: +154%
> O sea: escribir dejó de ser el cuello de botella y ahora lo son revisar, integrar
> y desplegar. El harness no es «más IA», es mover la automatización a ese tramo.
>
> ES: Y aquí engancha tu experiencia personal, que es la misma curva a escala 1:
> en el artículo anterior contaste que con 8 sesiones el cuello de botella pasó a
> ser tu tiempo de 確認. Es literalmente el DORA en una persona. Ese paralelo es
> tuyo y vale más que cualquier gráfico.

## 会社で起きていること

> ES: ⚠️ SECCIÓN DE RIESGO. Ver el aviso de confidencialidad en notes/.
> Lo que se puede contar sin permiso explícito: la FORMA general (Issue → labels →
> agentes → PR → review), que ya es pública en los artículos de referencia.
> Lo que NO se cuenta sin permiso: nombres de repos, workflows internos, capturas,
> métricas de la empresa, quién lo montó, nombres de clientes o producto.
>
> Recomendación fuerte: escribe esta sección en abstracto («日本の受託・自社開発の
> 現場でも導入が始まっています」) y pide permiso por escrito si quieres concretar.
> Un artículo de Zenn con detalles internos de un cliente es un problema que no
> compensa por unas cuantas いいね.
>
> TODO: preguntar a Ueno-san si puedes citar el sistema, y con qué nivel de detalle.
> Si dice que sí, esta sección se convierte en lo mejor del artículo. Si no hay
> respuesta, se escribe sin ella y el artículo sigue funcionando.

## ひとりでどこまでやったか

> ES: La parte 100% tuya y sin riesgo. Tu harness de una persona, hoy:
>   - `/bg`, `/solve`, `/continue`: una skill que coge un ticket (Jira o Notion),
>     crea el worktree, lanza un sub-Claude autónomo, y acaba en un draft PR.
>     Es exactamente la forma del harness corporativo, sin GitHub Actions.
>   - Los hooks de git que disparan skills solas (`post-commit` → `/release-draft`).
>   - `/ux-gate` como gate determinista antes de que el trabajo te llegue.
>   - Los PreToolUse que **deniegan** lo irreversible.
>
> ES: El contraste que hace el artículo interesante: el pipeline de la empresa vive
> en GitHub Actions porque hay varias personas y hay que VER quién hace qué. El tuyo
> vive en tu máquina porque el único que necesita verlo eres tú. **Misma idea,
> distinto soporte, y el motivo del soporte no es técnico sino organizativo.**
>
> TODO: recuperar la definición real de `/bg` y `/solve` (están en el repo de
> trabajo, no en `~/.claude`). Si son del cliente, describe el patrón sin pegar
> el código.

## 個人開発でやる価値がある部分／ない部分

> ES: **El corazón del artículo.** Una tabla, y que sea honesta — que haya varias
> filas en «no compensa», porque si todo compensa el artículo es publicidad.
> Borrador para que lo revises al redactar:
>
> | pieza | equipo | uno solo |
> |---|---|---|
> | Contexto en el repo (CLAUDE.md, docs/) | imprescindible | **imprescindible** — el tú de dentro de 3 meses ya es otra persona |
> | Gates deterministas (lint, tipos, pre-commit) | imprescindible | **imprescindible** — es lo más barato de montar y lo que más devuelve |
> | Worktree/aislamiento por tarea | útil | **imprescindible si vas en paralelo** |
> | Review automática por sub-agentes | imprescindible | útil — pero tú puedes mirar el diff de verdad, ellos no |
> | Todo en GitHub, visible | imprescindible | **no compensa** — la visibilidad es para OTROS; solo estás tú |
> | Labels + Actions como bus de agentes | imprescindible | **no compensa** — es infraestructura para coordinar personas |
> | Entornos preview por PR | imprescindible | depende — en móvil el coste de build es real (EAS no es gratis) |
> | 21 agentes con roles | tiene sentido | **no** — el coste de mantener 21 prompts te come el tiempo que ahorras |
>
> ES: La conclusión que sale de la tabla y que es tu tesis:
> **個人開発でハーネスに価値があるのは「強制」の部分だけで、「可視化」の部分はほぼ
> 要らない。** Porque la visibilidad existe para que otras personas sepan qué pasa,
> y tú ya lo sabes. Es una frase que se puede citar y que no está escrita en ningún
> otro sitio.

## デメリット — 実際に払ったコスト

> ES: La sección que hace que el artículo se guarde. Todo lo de aquí es vivido, con
> fecha (ver notes). No teorices, cuenta.
>
> 1. **Construir el harness es tiempo en el que no sale producto.** El caso de
>    referencia paró el desarrollo un mes entero y dedicó 2 semanas solo al bucle
>    de Issues. Es honesto decir que ese coste existe y que a escala de una persona
>    puede no amortizarse nunca.
> 2. **Los techos que no dependen de ti.** El límite semanal de la suscripción y la
>    RAM. Ya los contaste en el artículo anterior → aquí una línea y enlace, pero
>    con el giro nuevo: **el harness los empeora**, porque un pipeline autónomo
>    consume tokens aunque tú estés durmiendo.
> 3. **Los gates dan falsos positivos.** Ejemplo real y perfecto: escribiendo el
>    esqueleto de estos mismos artículos, el hook que impide publicar a producción
>    bloqueó la escritura del artículo — porque el TEXTO citaba el comando
>    prohibido. Una pared que mira patrones no distingue ejecutar de mencionar.
>    Es la mejor ilustración posible de que el harness también te frena a ti.
> 4. **Un `develop` roto bloquea a todos los agentes a la vez.** Pasó el 2026-08-20:
>    el pre-commit typechequea el proyecto entero, así que un commit incompleto
>    dejó tres sesiones sin poder commitear. **Cuanto más fuerte es el gate, más
>    caro es su fallo** — y eso es exactamente lo que no cuentan los casos de éxito.
> 5. **El harness hay que mantenerlo, y caduca.** Anthropic borra piezas del suyo
>    cada vez que sale un modelo nuevo. Lo que hoy es andamiaje, mañana es lastre.
>
> TODO: si tienes algún caso donde el pipeline autónomo produjo un PR que parecía
> bien y estaba mal, ESE es el ejemplo que le falta a todo el género. Piénsalo.

## どこから始めるか

> ES: Cerrar accionable, ordenado por retorno para una persona sola. Mi propuesta:
>   1. Un gate determinista que no se pueda saltar (pre-commit con tipos y lint).
>   2. Escribir en el repo el contexto que solo está en tu cabeza — y sobre todo el
>      POR QUÉ de cada regla.
>   3. Aislar cada tarea (worktree) para poder ir en paralelo sin romper nada.
>   4. Solo entonces, automatizar el disparo (hooks, o un `/bg` propio).
> El orden importa: automatizar el disparo antes de tener gates es multiplicar la
> velocidad a la que produces cosas que nadie ha verificado.
>
> ES: Y la última idea, la que cierra la serie entera:
> **ハーネスの本質は自動化ではなく、「その場の訂正」を「環境の改善」に変えること。**
> Cuando el agente falla, la pregunta no es cómo lo arreglo, es qué le faltaba al
> entorno. Eso vale igual para 21 agentes que para uno solo.
>
> TODO: enlazar al artículo de Claude Code (`claude-code-speed-quality`), que es el
> «cómo» concreto de los puntos 1-3 de esta lista.
