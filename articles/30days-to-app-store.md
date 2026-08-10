---
title: "30日で個人開発アプリをApp Storeに出した。使った技術と、捨てたもの"
emoji: "🚀"
type: "tech"
topics: ["reactnative", "expo", "個人開発", "typescript"]
published: false
---

<!--
  BORRADOR. `published: false` hasta que esté terminado.
  Plan y TODOs en Notion:
  https://app.notion.com/p/3b8ea0f87143817fafabf4122c0e1cd4

  Datos verificados (no cambiar sin volver a comprobarlos):
  - primer commit de oshisuki-mobile: 2026-06-30 20:59 (096e229, scaffold Expo SDK 54)
  - publicación en App Store: 2026-07-29
  - stack: Expo SDK 54 / RN 0.81.5 / expo-router 6 / better-auth 1.6 /
    zustand 5 / TanStack Query 5 / zod 4 / MMKV 4 / FlashList 2
  - 73 ficheros de test en src/
-->

## はじめに

6月30日の夜に `create-expo-app` を叩いて、7月29日に「推しスキ」がApp Storeに並びました。ちょうど30日です。

作ったのは、地下アイドルのライブや特典会の予定を管理するアプリです。スペイン人の私が、日本語で、日本の推し活のために作りました。開発もデザインもマーケティングも一人です。

この記事は「30日で作る方法」ではありません。**実際に何を使って、何を捨てたか**の記録です。うまくいかなかったところも書きます。

## 技術スタック

```
Expo SDK 54 / React Native 0.81.5
expo-router 6      画面遷移
better-auth 1.6    認証（Google / Apple / メール）
zustand 5          クライアント状態
TanStack Query 5   サーバー状態
zod 4              スキーマ
MMKV 4             ローカル永続化
FlashList 2        リスト
Neon + Prisma      DB
Vercel             バックエンド
Claude API         告知画像・URLの解析
```

テンプレートは Obytes starter をそのまま使いました。ゼロから組む時間を、そのぶん機能に回すためです。

## 30日で何がどれだけかかったか

記憶ではなく、gitの履歴から出した数字です。30日で277コミットでした。

| 週 | コミット数 |
| --- | --- |
| 6/30〜 | 3 |
| 7/7〜 | 13 |
| 7/14〜 | 71 |
| 7/21〜 | **190** |

**最後の9日間に、全体の69%が入っています。**

<!-- TODO(jordi): ¿por qué? Dos lecturas posibles y solo tú sabes cuál es la buena:
     (a) las primeras semanas fueron de decidir qué construir, y una vez claro
         el producto la ejecución fue rápida;
     (b) te comiste el plazo y trabajaste como un loco la última semana.
     Si es (b), dilo. Es lo que va a hacer que este artículo se comparta. -->

コードの量で見るとこうなります。

| ディレクトリ | 変更行数 |
| --- | --- |
| `features/events` | 8,747 |
| `features/mypage` | 7,108 |
| `lib/oshi` | 6,292 |
| `features/home` | 4,561 |
| `components/ui` | 3,442 |
| `features/auth` | 2,733 |

いちばん上が「現場（ライブ）の予定」で、これはアプリの中心なので納得です。

<!-- TODO(jordi): lo interesante es el 2º. ¿Esperabas que mypage se llevara
     7.100 líneas? Si la respuesta es «no», ese es el aprendizaje del artículo:
     dónde se te fue el tiempo sin que lo hubieras planeado. -->

## 捨てたもの

<!-- TODO(jordi): lista de lo que decidiste NO hacer para llegar al 29.
     Candidatos que sé que son ciertos:
     - sync de la agenda al backend (ENABLE_EVENT_SYNC sigue apagado hoy)
     - monetización (Stripe está cableado en código pero sin usar)
     - Android (salió 9 días después, el 7 de agosto)
     Confirma cuáles fueron decisión consciente y cuáles simplemente no dieron
     tiempo — esa distinción es la parte honesta del apartado. -->

## ローカルファーストにした理由

現場の予定は端末の中だけに持っています。バックエンドには上げていません（`ENABLE_EVENT_SYNC` は今もオフ）。

<!-- TODO(jordi): explicar el trade-off. Lo bueno: cero latencia, funciona sin
     cuenta, nada que sincronizar. Lo malo: reinstalar la app borra la agenda
     del usuario. Contarlo honestamente — es el apartado que da credibilidad. -->

## やってよかったこと / やらなくてよかったこと

<!-- TODO(jordi): 3 y 3. -->

## おわりに

リリースしてからの話——10日で70人にどう届いたか——は別の記事に書きます。
