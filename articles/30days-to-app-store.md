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

## 30日でいちばん時間を食ったもの

<!-- TODO(jordi): ¿fue el parser de IA, el calendario, o el auth?
     Rellenar con lo que realmente te comió los días. Este es el apartado que
     más se lee de todo el artículo. -->

## 捨てたもの

<!-- TODO(jordi): lista de lo que decidiste NO hacer para llegar al 29.
     Ej.: sync de agenda al backend (sigue apagado), monetización,
     Android en la v1. -->

## ローカルファーストにした理由

現場の予定は端末の中だけに持っています。バックエンドには上げていません（`ENABLE_EVENT_SYNC` は今もオフ）。

<!-- TODO(jordi): explicar el trade-off. Lo bueno: cero latencia, funciona sin
     cuenta, nada que sincronizar. Lo malo: reinstalar la app borra la agenda
     del usuario. Contarlo honestamente — es el apartado que da credibilidad. -->

## やってよかったこと / やらなくてよかったこと

<!-- TODO(jordi): 3 y 3. -->

## おわりに

リリースしてからの話——10日で70人にどう届いたか——は別の記事に書きます。
