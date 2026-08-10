# zenn-articles

Artículos de [zenn.dev/jordisantamaria](https://zenn.dev/jordisantamaria), servidos
por Zenn Connect desde este repo.

## Cómo se trabaja

```bash
npx zenn new:article      # crea articles/<slug>.md con el frontmatter
npx zenn preview          # http://localhost:8000, se ve como se publicará
```

Un artículo nace con `published: false` y **solo sale publicado al cambiar esa
línea a `true` y hacer push**. Por eso se puede empujar a medias sin miedo: hasta
que no se cambia la bandera, en Zenn no lo ve nadie.

El plan editorial, los datos verificados y los pendientes de cada artículo viven en
Notion, en las subpáginas de 📱 OshiSuki.
