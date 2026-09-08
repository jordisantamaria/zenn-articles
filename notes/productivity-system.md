# Notas — artículo de productividad general (no-AI)

Volcado de los conceptos que va dando Jordi. **Fuente de verdad de las ideas**; el
artículo se redacta en `articles/productivity-system.md`.

---

## LA TESIS (lo que quiere que se lleve el lector)

Este artículo es **las cosas principales que hago yo**, no una guía exhaustiva de todo
lo que se puede hacer para ser productivo. **Esa guía no existe en ningún sitio.**

**Por qué es imposible una guía que lo resuelva todo:** porque *todo* afecta a la
productividad. Si procrastinas. Si estás enfocado en la tarea adecuada o en otra. Tu
salud física. Tu salud mental. Si estás cansado, si estás estresado.

**Por tanto lo que necesitas no es leer una infinidad de artículos y libros.** Lo que
necesitas es, en tu día a día, **crear un tiempo para pararte a reflexionar** sobre
cuáles son las cosas que más afectan a *tu* rendimiento, explorar soluciones que
resuelvan *tu* problema, y tomar acción para resolverlas.

**Eso se logra vía journaling.**

> Nota: es la misma forma que la tesis del artículo de Claude Code («no hay fórmula, lo
> construyes con lo que te pasa a ti»). Consistencia de marca, no repetición: allí es el
> setting, aquí es la vida. Cuidado con que no suene a calco.

---

## 0. La base de todo: los objetivos

Va **primero** aunque en la lista aparezca tarde, porque sostiene lo demás.

- ¿Productivo para qué? **Productivo = acercarte más rápido a tus objetivos.** Sin
  objetivos, la productividad no sirve de nada.
- Al tenerlos claros, puedes enfocarte en lo que realmente importa.
- **Y sin objetivos no te mantienes sano mentalmente**, así que la productividad se
  rompe en el primer paso: caes en depresión por no tener control sobre tu vida, y tu
  vida no va a donde quieres porque ni siquiera sabes dónde es.
- Dos horizontes:
  - **Visión de futuro a 5-10 años**: el estado final, la vida ideal.
  - **Objetivos semanales**: cuáles son las siguientes acciones que más te acercan.
- **Dónde reflexiona: en la sauna.** Tiempo sin pantallas, deliberado.

## 1. Pomodoro (no estricto)

No lo usa a rajatabla todos los días ni todo el tiempo. **Sirve para recuperar el foco y
para no procrastinar.**

- **El valor NO son los descansos: es la conciencia del tiempo.** «Esta tarea la había
  estimado en 1h y ya han pasado 3h y ni siquiera he empezado». «Ya han pasado 30 minutos
  y se acabó el ciclo, ¿y no he hecho más que scrollear en X?».
- **La recompensa del descanso sirve para arrancar.** Tarea que da pereza o que parece
  muy difícil: por ahora solo 25 minutos de foco real. **Casi siempre te das cuenta de que
  no era tan difícil.**
- Y luego te recompensas. **Si has logrado resultados de verdad, puedes estirar el
  descanso** — no 5 minutos, hasta 30. Nadie se va a quejar de que te tomes recompensas
  de «procrastinación» si obtienes resultados.

## 2. La herramienta: Super Productivity

**Super Productivity** (open source, Flatpak, v18.21.2). Confirmado por Jordi el 2026-09-08.

- Tareas del día **por proyecto**.
- **Estimación de tiempo** — lo que le hace ser más consciente del tiempo.
- Pomodoro **trackeando el tiempo de una tarea concreta**.

### Datos medidos de su instalación (backup del 2026-09-08)

| | |
|---|---|
| Periodo | 2026-04-21 → 2026-08-18 |
| Días con trabajo registrado | **48** |
| Horas trackeadas | **144,8** |
| Mediana por día activo | **2,4 h** |
| Tareas | 125, en 2 proyectos (Inbox y el cliente) |

**Lo que de verdad hace el mecanismo de «conciencia del tiempo»**, y es distinto de lo que
parecía: en su config, `defaultEstimate` está en **25 minutos** y
**`isNotifyWhenTimeEstimateExceeded: true`**. Es decir:

- **No estima tarea por tarea.** 116 de 119 tareas conservan los 25 minutos por defecto.
- Toda tarea nace valiendo **un pomodoro**, y la app **le avisa cuando lo pasa**.
- Y lo pasa mucho: la tarea mediana consumió **2,3 pomodoros** (58 min), **el 62% necesitó
  más de uno**, y la peor se comió **20** (25 min estimados → 8h17 en «Revisar excel»).

Ese aviso ES la mecánica que describe («ya han pasado 30 minutos y ¿qué he hecho?»), y el
62% es la prueba de que salta constantemente. **Ojo al redactar: NO decir que subestima
2,3× — no hay estimación propia, hay un default. Decir que la unidad es el pomodoro y que
el aviso salta en 6 de cada 10 tareas.**

Y confirma lo que él ya dice: **48 días activos en 4 meses** = pomodoro **no estricto**,
se enciende cuando hace falta. El tracking se detiene el 18/08 (hay que preguntarle si
dejó de usarlo o solo dejó de trackear).

## 3. Atajos de teclado

## 4. Bookmarks, en dos categorías distintas

1. **Material para el futuro**: cualquier cosa a la que pueda darle uso después. Posts de
   X para inspirarse al redactar posts, webs para inspiración de diseño, etc.
2. **Atajos a las acciones que más hace en su día a día.**

## 5. La revisión semanal

Una vez por semana repasa **todo lo que ha capturado**: imágenes descargadas, vídeos,
ficheros, favoritos, bookmarks, notas… y lo organiza.

El libro es **«Building a Second Brain» (Tiago Forte)**. Confirmado el 2026-09-08.

⚠️ **NO citar «Getting Things Done»: Jordi no lo ha leído.** Aunque la revisión semanal,
el inbox vacío y «resuélvelo ahora» se parezcan a GTD, aquí no son una referencia — son su
propio sistema. Atribuirlo a un libro que no ha leído sería falso, y además le quita el
mérito de haber llegado solo.

## 6. PARA, adaptado

Usa el método **PARA**, pero adaptado a su caso para que le resulte cómodo, añadiendo
accesos directos y tags. El reparto por herramienta:

| Qué | Dónde |
|---|---|
| Fuente principal | **Notion** |
| Ficheros | **Drive** |
| Imágenes | **cloud** |
| Enlaces | **Raindrop** |

## 7. Capturar todo — y resolver en el momento lo que se pueda

- Anotar todo lo que se te vaya ocurriendo durante el día.
- **Y todo lo que puedas resolver en el momento, resuélvelo en el momento** — por ejemplo
  preguntándole a la IA. Dos razones:
  - **Ya tienes todo el contexto**, así que cuesta mucha menos energía que rearrancarlo
    en el futuro.
  - **El 90% de lo que dejas para el futuro no llega a ocurrir nunca.** Lo que acabas
    teniendo es una lista infinita de cosas por hacer que solo genera estrés y la
    sensación de que nunca estás al día.

## 8. La bandeja de entrada siempre limpia

- Abre el correo **en cada pausa**, cada pocas horas.
- Lo que no es relevante: **desuscribirse** si se puede, y **archivarlo todo**.
- **Nunca se elimina** — nunca se sabe si algún día vas a querer rescatarlo.
- Así en la bandeja solo quedan **2 o 3 cosas que requieren acción**. Cada vez que abre
  el correo, le recuerdan que siguen pendientes.
- Al tomar la acción, **archivar en el momento**. Y esa acción, tomarla lo antes posible,
  idealmente ya.

## 9. El calendario

Todo lo que tiene que ocurrir en un momento concreto **va siempre al calendario**. Y si
de verdad no puedes fallar en esa fecha, **alarma en el móvil**.

## 10. Sprints por proyecto, y el foco del día

- Lista de tareas basada en **sprints por proyecto**.
- **Proyecto = algo que quieres lograr.** Lo ideal es que sea alcanzable **en 3 meses**.
- Al final o al principio de cada día, **definir qué tareas vas a hacer ese día**. Solo
  las que de verdad crees que vas a poder terminar, y quizá un poco más por si da tiempo.
- **Eso es tu foco del día. Todo lo demás, por tanto, es procrastinación.**
- Solo añades cosas si son **urgentes de verdad**, o si es un «esto se me ocurre ahora y
  lo resuelvo rápido» que no merece una tarea nueva.

## 11. Los dos roadmaps

- **Roadmap de tareas a 3 meses** (el horizonte del proyecto).
- **Roadmap de «visión de futuro» a 5-10 años**: dónde quiere llegar, su vida ideal.

---

## Decidido sobre la publicación (medido el 2026-09-08)

**Sí va en Zenn**, pero **NO como los anteriores**:

- `type: idea`, no `tech`. Los que funcionan en este terreno son casi todos `idea`.
- Tags: **`仕事術`, `タスク管理`, `notion`, `生産性`**.
  - ❌ `ライフハック` (78 artículos, se hunde tras el primero) y `習慣` (23, muerto).

Rendimiento histórico de la temática en Zenn:

| tag | artículos | mejores |
|---|---|---|
| `notion` | 874 | 451 / 440 / 433 / 325 |
| `仕事術` | 139 | 362 / 212 / 204 / 194 |
| `タスク管理` | 212 | 240 / 194 / 98 |
| `生産性` | 580 | 337 / 319 / 315 / 269 |

Referencias del género que ya funcionaron, y por qué importan:

- 「1年目エンジニアがバリューを出すためにした工夫、**結果が出たモノのみ**具体的にまとめてみる」 — **362♥**.
  El 「結果が出たモノのみ」 es exactamente el encuadre de Jordi: lo que hago yo, no lo exhaustivo.
- 「AIを5本同時に走らせても、俺の脳みそは1個しかない」 — **315♥**. Productividad con marco de ingeniero.
- 「【タスク管理術】Notionで全ての仕事を管理する方法を徹底解説」 — **194♥**.

**Lo que hay que evitar**: que se lea como un artículo de autoayuda. El género premia
「具体的に」 y 「結果が出たモノのみ」 — concreto y personal, no teoría general.

## Pendiente de Jordi

1. ~~El nombre de la herramienta~~ → **Super Productivity** ✅
2. ~~El libro~~ → **Building a Second Brain** ✅ (GTD no lo ha leído: no citarlo)
3. ~~Evidencia medible~~ → **144,8 h en 48 días; el aviso de pomodoro excedido salta en el
   62% de las tareas** ✅
4. ¿Sigue usando Super Productivity? El tracking se corta el 2026-08-18.
5. ¿Se pueden dar los nombres de proyecto? Ahora mismo son «Inbox» y el cliente — **el
   nombre del cliente no puede salir en el artículo**.
