# Notas — aws-saa-in-two-weeks

No van dentro del `.md`: **Zenn no oculta los comentarios HTML, los imprime**.
El esqueleto del artículo sí lleva guion inline (en español, para que sea imposible
publicarlo por accidente), pero hay que **borrarlo entero** antes de `published: true`.

Estado: **esqueleto**, sin redactar. Creado el 2026-08-21.

## Títulos candidatos

1. `2週間でAWS SAAに受かった。教材を読むのをやめて、教材を書きました`
   → el método es el gancho. Es el que recomiendo.
2. `AWS SAA対策リポジトリを公開したら、宣伝ゼロで半年間Googleから人が来続けている`
   → el gancho es el repo/SEO, no el certificado. Mejor si el eje acaba siendo la
     cola larga, y compite con muchos menos artículos.
3. `AIに教材を書かせて、2週間でAWS SAAに受かった話`
   → **el más fuerte para Zenn 2026** si la respuesta al TODO de「AIに教材を書かせた話」
     es que los docs los generó Claude. Decide eso ANTES de redactar.

## Datos verificados (2026-08-21, contra la API de GitHub)

Repo: https://github.com/jordisantamaria/aws-solutions-architect-lab

- creado: 2026-02-13 14:30 UTC (`Initial commit: AWS SAA-C03 study repository`)
- commits de estudio: 2/13, 2/18, 2/22, 2/23, 2/26 (x2), 2/27, 3/1 → **17 días naturales**
- 2026-03-11: tres commits de traducción ES→EN + borrado de las notas personales
  (`Remove personal study notes (weak-concepts)`) ← ojo, los weak-concepts YA NO
  están en el repo público; si los citas en el artículo, sácalos del historial.
- contenido hoy: **94 ficheros**, 14 carpetas en `docs/`, **10 labs** de Terraform
  (`labs/00-setup` … `labs/09-full-architecture`), `exam-prep/` con cheat-sheets,
  decision-trees y practice-questions.
- el README promete un roadmap de **10 semanas** (Phase 1-5). Sigue ahí. La tensión
  «prometía 10 semanas / lo hice en 2» es material del artículo, no un error a tapar.

### Estrellas y tráfico (no es un pico, es goteo)

| fecha | usuario |
|---|---|
| 2026-06-06 | paulolnobre |
| 2026-06-14 | fatema-maitham |
| 2026-07-26 | nhai10825 |
| 2026-08-05 | jayanth-anala |
| 2026-08-11 | MarouaneBouaricha |
| 2026-08-16 | UsmanKhalil25 |
| 2026-08-18 | sunt0x1 |

- 2 forks (2026-04-16, 2026-08-20)
- tráfico de los últimos 14 días: **218 visitas / 39 únicos**
- referrers: Google 125, github.com 27, DuckDuckGo 12, Brave 2, Yahoo 1
  → **el 100% del tráfico es búsqueda. Nunca se promocionó en ningún sitio.**

⚠️ «1 → 7 estrellas recientemente» es un redondeo: fueron 7 estrellas repartidas en
10 semanas, una a una. Contarlo como subidón se cae en cuanto alguien abre el
stargazers. El dato bueno no es el número, es que **entran todas las semanas sin
que nadie lo empuje**, y que empezó justo después de traducirlo al inglés.

## Pendientes que bloquean la redacción

- [ ] **Fecha del examen y score.** No están en ningún sitio que yo pueda leer.
- [ ] **Horas al día de estudio.** Sin esto, «2 semanas» no es reproducible.
- [ ] **¿Partías de cero en AWS?**
- [ ] **Coste real**: examen + factura de AWS de febrero + material de pago
      (Udemy / Skill Builder / simulacros). Si hubo material de pago, hay que decirlo.
- [ ] **¿Los docs los escribió Claude?** ← decide el eje del artículo entero.
- [ ] Elegir 1 lab para enseñar código y 1 pregunta de examen que el lab resolviera.

## Relación con la serie

- Si el eje acaba siendo「AIに教材を書かせた」, este artículo y el de Claude Code
  (`claude-code-speed-quality`) se enlazan mutuamente: uno es la IA aplicada a
  aprender, el otro a producir.
- Este es el único de la serie que **no** habla de 推しスキ. Entra por búsqueda de
  AWS, público distinto. No forzar el puente más allá de una línea en el perfil.
