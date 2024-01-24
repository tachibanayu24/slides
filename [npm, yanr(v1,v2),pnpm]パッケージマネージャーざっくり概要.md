---
marp: true
theme: default
class: invert
paginate: true
footer: Written by [@tachibanayu24](https://twitter.com/tachibanayu24)
---

# [npm, yarn(v1,v2),pnpm]<br />パッケージマネージャーざっくり概要

<style scoped>
section { 
    font-size: 28px; 
}
</style>

2024.01.24
tatti ([@tachibanayu24](https://twitter.com/tachibanayu24))

---

## 👋 はじめに

- 話すこと
  - 各パッケージマネージャーの概要やパフォーマンス、実運用上の懸念
- 話さないこと
  - 詳細なアルゴリズムや思想
  - bun

---

## npm の特徴

- 同じに npm, Inc.(2020 年に GitHub により買収され現在 MS 管理)が提供するパッケージマネージャー
- Node.js によって開発されたパッケージマネージャーのパイオニア
- 実は Node Package Manager の略語ではない(なので全て小文字表記が正しい)
  - `npm is not an acronym` の recursive acronymic abbreviation(?)
  - [FAQ on Branding](https://github.com/npm/cli)

---

## yarn classic(v1) の特徴

- Facebook と Google を中心として開発
- npm に当初存在していた課題を解決するために始まった当時としては革新的なサードパーティ npm
- `yarn why` (なぜそのモジュールがインストールされているのかを分析) など独自機能がある
- 現在では yarn の独自機能は npm に取り込まれており差分はずいぶん小さくなった
  - lockfile によるバージョン固定
  - パフォーマンス改善
  - workspaces
  - `yarn upgrade-interactive` (対話的更新)
- 現在はメンテナンスモード

---

## yarn berry(v2) の特徴

- 2020 年に正式リリース(pnpm より若い)
- 独自路線とも言える様々な機能を同梱
  - Yarn Plug'n'Play (PnP)
    - node_modules を作成せずに、依存関係のマッピングファイルを利用してディスク効率を高める
    - `.yarn/cache` に圧縮した各パッケージを配置して git に stage することで install が不要に

---

## pnpm の特徴

- Performant npm
- yarn と同様に npm のサードパーティ
- ディスク効率やパフォーマンスに優れる
  - package の hoisting の代わりに symbolic link を利用
  - @see [Motivation](https://pnpm.io/motivation)

---

## パフォーマンス比較

<style scoped>
li { 
    font-size: 24px; 
}
</style>

以下の条件による計測です。それぞれ現実世界でどのようなシチュエーションに対応するかも示します(ほぼ起こり得ない状況は割愛)。
対象は npm, pnpm, yarn classic, yarn PnP です。

- clean install: まっさらな状態で install するとき
- with lockfile: CI サーバーで install されるとき
- with node_modules: lockfile を削除して更新したいとき
- with cache, lockfile: git clone 後最初の install など
- with cache, lockfile, node_modules: 2 回目以降の install
- update: package.json を変更して install を実行

---

## パフォーマンス比較

<style scoped>
*:not(h2) { 
    font-size: 20px; 
}

em {
    color: green;
}
</style>

| Action  | Cache | Lockfile | node_modules | npm    | pnpm   | Yarn | Yarn PnP |
| ------- | ----- | -------- | ------------ | ------ | ------ | ---- | -------- |
| install |       |          |              | 27s    | 8.8s   | 7.7s | _3.7s_   |
| install | ✔     | ✔        | ✔            | 1.5s   | _1.1s_ | 4.8s | n/a      |
| install | ✔     | ✔        |              | 7.3s   | 2.7s   | 5.3s | _1.4s_   |
| install | ✔     |          |              | 11.4s  | 6.1s   | 7.6s | _3.1s_   |
| install |       | ✔        |              | 12.2s  | 5.5s   | 5.4s | _1.4s_   |
| install | ✔     |          | ✔            | 1.7s   | _2.3s_ | 7.1s | n/a      |
| install |       | ✔        | ✔            | 1.5s   | _1.1s_ | 4.8s | n/a      |
| install |       |          | ✔            | _1.7s_ | 5.5s   | 6.9s | n/a      |
| update  | n/a   | n/a      | n/a          | 6.4s   | 4s     | 5.8s | _3.3s_   |

[Benchmarks of JavaScript Package Managers](https://github.com/pnpm/benchmarks-of-javascript-package-managers)より

---

## 実運用上起こり得る懸念

- npm/yarn classic
  - 遅い
- yarn berry
  - 依存関係をコミットする
  - 他パッケージマネージャーと互換性が低い
- pnpm
  - 特定のライブラリが非対応だったりするかも

---

## 👋 おわり

---

# 参考

- [pnpm/pnpm.io/tree/main/benchmarks](https://github.com/pnpm/pnpm.io/tree/main/benchmarks)
- [Choosing the Right Package Manager: NPM, Yarn, or PNPM?](https://hackernoon.com/choosing-the-right-package-manager-npm-yarn-or-pnpm)
- [npm と yarn と pnpm の違い 2021](https://zenn.dev/hibikine/articles/27621a7f95e761)
