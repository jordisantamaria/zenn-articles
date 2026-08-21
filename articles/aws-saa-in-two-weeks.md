---
title: "TODO: 2週間でAWS SAAを取った話（仮題 — 候補は notes/aws-saa-in-two-weeks.md）"
emoji: "☁️"
type: "tech"
topics: ["aws", "資格", "terraform", "個人開発", "saa"]
published: false
---

<!-- ⚠️ ESQUELETO — NO PUBLICAR TAL CUAL.
     Zenn IMPRIME los comentarios HTML, así que antes de poner published: true
     hay que borrar este bloque y todas las líneas «> ES:» / «> TODO:».
     El guion va en español a propósito: es imposible publicarlo por error sin verlo.
     Datos verificados y pendientes: notes/aws-saa-in-two-weeks.md -->

> ES: **Tesis del artículo.** El repo no es el souvenir del estudio: *es* el estudio.
> Escribir el material que un principiante necesitaría fue el método, y el examen
> salió de ahí. El bonus es que ese material sigue trabajando seis meses después,
> mientras el certificado se queda quieto.
>
> **Objetivo secundario, no disfrazado:** que el lector abra el repo. Enlace arriba
> (はじめに) y abajo (おわりに), no en cada sección.

## はじめに

> ES: Abrir con el resultado en la primera frase — es lo que funcionó en 30days.
> Los tres datos: fecha del examen, score, y «el repo decía 10 semanas, lo hice en 2».
> Y decir ya aquí que el repo es público, con el enlace.
>
> TODO: fecha exacta del examen y score (no los tengo verificados). Mirar el correo
> de Credly / AWS Certification.
> TODO: enlazar el badge de Credly si quieres que se vea la credencial.

https://github.com/jordisantamaria/aws-solutions-architect-lab

## 「2週間」の中身：2月13日から3月1日まで

> ES: Los datos reales del repo (verificados, ver notes):
>   - 2026-02-13 primer commit (scaffold del repo de estudio)
>   - 2026-02-18, 02-22, 02-23, 02-26, 02-27 → commits de 弱点 (weak concepts)
>   - 2026-03-01 último commit de contenido de estudio
> O sea: 17 días naturales. Si vas a titular «2週間», dilo aquí con la fecha exacta
> en vez de redondear en el título y que lo descubra el lector — el mismo criterio
> del 30days (los 30 días eran 30 días de verdad).
>
> TODO: ¿cuántas horas/día? Si eran 2h después del trabajo, ese dato es el que hace
> el artículo creíble y reproducible. Sin él, «2 semanas» suena a que no trabajabas.
> TODO: ¿partías de cero en AWS o ya lo tocabas en Cierpa? Decirlo. Un lector que
> parte de cero necesita saber que tú no partías de cero (si es el caso).

## やったこと：教材を「読む」のではなく「書く」

> ES: El núcleo. Explicar el método:
>   1. Leer el exam guide oficial y convertir cada dominio en una carpeta de docs/
>   2. Escribir la explicación como si se la contaras a alguien → si no sale, no lo sabes
>   3. Cada área con un lab de Terraform que la levanta de verdad
>   4. Los fallos de los simulacros vuelven al repo como «weak concepts»
> Es el bucle: examen simulado → fallo → escribir el concepto → siguiente.
> Los commits del 2/22 al 2/27 son literalmente ese bucle («update conceptos debiles»).

### リポジトリの構成

> ES: Cifras verificadas: 94 ficheros, 14 áreas de docs, 10 labs de Terraform,
> exam-prep con cheat sheets + decision trees + preguntas de práctica.

```
docs/           → 14の分野（IAM, VPC, EC2, S3, RDS, ...）
labs/           → Terraformで実際に建てる10個のラボ
exam-prep/      → チートシート・判断フローチャート・練習問題
```

> ES: Poner UNA tabla con los 10 labs y qué servicio prueba cada uno. Y elegir UNO
> para contarlo en detalle — el 04-three-tier-app (ALB + ECS + Aurora + ElastiCache)
> es el que más se parece a una pregunta de examen real.
>
> TODO: sacar del repo el `main.tf` del lab que elijas y pegar 15-20 líneas, no más.

## Terraformで建てると、選択肢問題が「思い出す問題」じゃなくなる

> ES: El argumento que diferencia este artículo de los otros 500 「SAA合格しました」
> de Zenn. El examen SAA es de escenarios: te dan una arquitectura y cuatro opciones.
> Si solo memorizaste tablas, comparas cadenas de texto. Si levantaste el NAT Gateway
> y te cobraron por él, la opción cara la reconoces sin leerla entera.
> Ejemplo concreto: TODO — elegir 1 pregunta de examen (o de tus 練習問題) donde el
> lab te dio la respuesta directamente. Sin ejemplo esta sección es una opinión.

### かかったお金

> ES: Sección corta pero es la que la gente busca en Google.
> TODO: coste real. Examen: 150 USD (verificar el precio en yenes que pagaste).
> TODO: factura de AWS de febrero — los labs con NAT Gateway y Aurora no son Free
> Tier. Si te llegó un susto, ese susto es contenido.
> TODO: ¿pagaste algún curso/simulacro (Udemy, TechStock, AWS Skill Builder)? Decirlo.
> El artículo pierde credibilidad si parece que solo con el repo se aprueba y luego
> resulta que hiciste 500 preguntas de pago.

## AIに教材を書かせた話

> ES: Esto hay que decirlo, y decirlo tú antes de que lo pregunten. Si los docs
> los generó Claude a partir del exam guide, el artículo NO se sostiene como
> 「2週間で書いた94ファイル」 sin explicar cómo.
> El matiz que salva —y que es verdad— es el orden: la IA escribe el borrador del
> concepto, tú lo corriges cuando fallas el simulacro. Lo que fija el conocimiento
> no es escribirlo, es descubrir que lo escrito estaba mal.
>
> TODO: confirmar qué parte fue generada y qué parte escribiste tú. Si fue todo IA,
> el artículo cambia de eje: pasa a ser 「AIに教材を作らせて2週間で受かった話」, que
> además es MEJOR tema para Zenn hoy. Decidir esto antes de redactar — cambia el
> título, los topics y el enganche con el artículo #4.

## リポジトリは、資格より長く働いてくれる

> ES: El cierre y el motivo real de publicar esto. Datos verificados (ver notes):
>   - ⭐ 1 → 7 entre junio y agosto. NO fue un pico: llegaron de una en una
>     (6/6, 6/14, 7/26, 8/5, 8/11, 8/16, 8/18). 2 forks.
>   - 218 visitas / 39 únicos en 14 días, 125 de ellas desde Google.
>   - Nunca lo publiqué en ningún sitio. Todo el tráfico es búsqueda.
> El punto: **el 3/11 lo traduje al inglés** (commits «Translate all remaining
> content», «Rename Spanish filenames»). Estaba en español y no lo encontraba nadie.
> Esa decisión de un día es la que produjo el goteo de después.
>
> ES: Cuidado con el marco. «7 estrellas» no es un éxito que presumir, y presentarlo
> como tal se lee mal. El marco honesto es el que sí es interesante: un repo que
> nadie promocionó recibe visitas todas las semanas seis meses después, mientras
> el certificado no ha traído ni un solo clic. Esa asimetría es el artículo.

## おわりに

> ES: Cerrar con lo que el lector se lleva:
>   1. El repo es reutilizable — está en inglés, tiene los labs, y el roadmap de
>      10 semanas sigue ahí para quien quiera ir despacio.
>   2. Qué haría distinto (TODO: piénsalo — ¿los labs de multi-region te sobraron?)
> Enlace al repo otra vez, y enlace al artículo #4 si para entonces ya está fuera.
>
> TODO: decidir si mencionas 推しスキ aquí. A favor: es tu marca y el lector de
> 個人開発 puede saltar. En contra: este artículo entra por gente buscando SAA, que
> no viene por ti. Recomendación: una línea en プロフィール, no una sección.

https://github.com/jordisantamaria/aws-solutions-architect-lab
