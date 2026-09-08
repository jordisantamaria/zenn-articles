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

### Cómo la usa de verdad (corregido por Jordi el 2026-09-08)

- **No la usa siempre. Solo cuando le cuesta coger el foco.**
- **Las estimaciones no son estrictas**: son una referencia para darse cuenta de que el
  tiempo pasa. No son un objetivo que cumplir.
- **No hace falta ser extremadamente disciplinado para obtener resultados** — y eso es
  parte del mensaje del artículo, no una excusa.

⚠️ **NO usar los datos de Super Productivity como evidencia del artículo.** Los backups dan
144,8 h en 48 días entre abril y agosto, pero eso **no mide su productividad**: mide los
días que necesitó ayuda para concentrarse. Publicarlo sugeriría que trabaja 2,4 h al día,
que es falso. Lo mismo vale para el «el aviso salta en el 62% de las tareas»: sale de la
misma muestra sesgada.

Detalle de configuración que sí se puede contar, porque es el mecanismo y no una métrica:
toda tarea nace valiendo **un pomodoro** (`defaultEstimate` = 25 min) y la app **avisa
cuando lo pasa** (`isNotifyWhenTimeEstimateExceeded`). Ese aviso es el «ya han pasado 30
minutos, ¿y qué he hecho?».

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

---

## Regla sobre las cifras (fijada el 2026-09-08)

**Cada artículo se prueba con la evidencia de SU mecanismo, no con la cifra de commits.**

Los commits demuestran «entrego mucho código». **No demuestran** que PARA funcione, ni el
inbox vacío, ni el journaling. Usarlos como prueba de todo es el atajo que un lector
atento detecta — y en はてな alguien lo dice en los comentarios.

| artículo | su evidencia |
|---|---|
| Claude Code (setting) | commits/mes: 100 → 270 |
| Productividad general | **ninguna métrica de tracking** — ver abajo |
| Flujo diario con IA | (pendiente: ramas de tarea, tamaño de PR) |

Los commits pueden aparecer **una línea, como contexto de quién escribe** — nunca como la
prueba del artículo. Y son el activo más fuerte que tiene: gastarlo de adorno en cuatro
artículos lo devalúa.

**Excepción, y es este artículo: aquí no va ninguna métrica.** Exigirle un número medido
contradice su propia tesis — un artículo que empieza enseñando horas trackeadas dice «mira
qué disciplinado soy», y lo que dice el artículo es que no hace falta serlo. Lo que
sostiene esta pieza es **lo concreto que sea el sistema** y, como contexto, el resultado
(las 3.566 contribuciones). Nada más.

### Cifras verificadas (2026-09-08, ventana 2025-09-08 → 2026-09-08)

| dato | valor | cómo se comprueba |
|---|---|---|
| Contribuciones en GitHub | **3.566** | GraphQL `contributionCalendar.totalContributions`; es lo que enseña el gráfico del perfil |
| — de ellas, privadas | 3.185 | `restrictedContributionsCount` |
| Commits reales contados | **2.604** | `git log --all --no-merges --author` en los 23 repos locales |

**Cuenta todo: una PR o una review es trabajo igual que un commit.** Lo único que hay que
cuidar es la palabra.

- ✅ **「3,566 コントリビューション」** — entra todo, es lo que enseña el gráfico del perfil,
  y es como los devs japoneses lo cuentan (草).
- ❌ **「3,566 コミット」** — GitHub no desglosa las 3.185 privadas, así que le regala a
  cualquiera un «esos no son todos commits». Cuesta credibilidad y no aporta nada.

---

## Decisión abierta: el bloque de metas personales (cap. 1)

Está **escrito y dentro** del artículo, pero Jordi no lo tiene claro (2026-09-08).

El bloque es 「目標は、仕事の目標だけではありません」 con: 日本料理を作れるようになった /
初めて恋人ができた / 収入も大きく増えた / アプリをゼロから / 月15回以上ライブ.

- **A favor:** es el diferenciador del artículo. Todo el género mide producción de trabajo;
  este dice que si devuelves el tiempo ganado al trabajo no has ganado nada. Y el remate
  (「どれも、目標として書いてあったものです」) convierte la lista en prueba del capítulo 1.
- **En contra:** es lo único del artículo que **no se puede despublicar** de la cabeza de
  quien lo lea, va bajo su nombre real, y desplaza el registro hacia blog personal en una
  pieza que usa como portafolio.
- **Si se quita:** se pierde menos de lo que parece. La intro ya lleva 月15回以上ライブ, que
  sostiene el mismo argumento sin exponer nada.

**RESUELTO el 2026-09-08:** fuera la novia (privacidad) y fuera el salario (el artículo se
comparte con el cliente freelance, y publicar que sus ingresos subieron mucho le debilita
la próxima negociación de tarifa). Queda 日本料理 / アプリ / 月15回ライブ — dos de los tres
no tienen nada que ver con programar, así que el argumento del párrafo sigue en pie.

⚠️ **Descartada la variante «solo salario y la app»**: las dos son logros de trabajo y
dinero, así que el párrafo pasaría a demostrar lo contrario de lo que afirma. Lo que hace
funcionar el bloque es justamente 日本料理 y las ライブ — lo que no tiene nada que ver con
programar.

