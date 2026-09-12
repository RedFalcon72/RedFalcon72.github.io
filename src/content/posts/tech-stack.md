---
title: "Astro v5 + Tailwind CSS v4で作る、静的で高速な個人開発ポータル"
date: 2026-09-12
category: "Dev"
tags: ["Astro", "TailwindCSS", "JavaScript", "フロントエンド"]
---

個人開発で作ったものや技術的な学びをまとめるため、自作のデベロッパーポータル「RedFalcon72 Portal」を構築した。

既存のブログサービスを利用する方法もあるが、今回は「自分で必要なものを考え、実際に作って完成させる」ことを目的として、サイトの基盤から自分で構築することにした。

この記事では、サイトの構築に採用した Astro v5 と Tailwind CSS v4 の選定理由に加えて、実装中に特に試行錯誤したコードブロックのカスタマイズについて紹介する。

---

## 技術構成

今回のサイトでは、主に以下の技術を使用している。

- Astro v5
- Tailwind CSS v4
- JavaScript
- Markdown / Content Collections
- Shiki

記事自体はMarkdownで管理し、Astroによって静的なHTMLへ変換している。

---

## Astro v5を選んだ理由

### 必要なJavaScriptだけを使う

サイトを作るうえで特に重視したのが、できるだけ余計なJavaScriptを送らないことだった。

今回作っているのは、ブログ記事や開発したものを紹介する個人ポータルなので、ページを表示するために大量のJavaScriptを実行する必要はない。

そこで採用したのがAstroだった。

Astroでは、基本的にHTMLを中心としたページを生成し、必要な部分だけにJavaScriptを追加できる。

そのため、動的な処理が必要ないページでは、クライアント側へ送るJavaScriptを最小限に抑えられる。

個人ブログのようなコンテンツ中心のサイトとは相性がよく、表示速度とシンプルな構成を両立できると考えた。

### Content Collections

記事管理にはAstroのContent Collectionsを利用している。

記事をMarkdownで作成し、Frontmatterにタイトルや日付、カテゴリー、タグなどを記述する。

例えば、この記事も以下のようなFrontmatterから管理している。

```yaml
title: "Astro v5 + Tailwind CSS v4で作る、静的で高速な個人開発ポータル"
date: 2026-09-12
category: "Dev"
tags: ["Astro", "TailwindCSS", "JavaScript", "フロントエンド"]
```

記事が増えていった場合でも、メタデータの形式を統一して管理できるため、個人開発でも扱いやすい。

---

## Tailwind CSS v4を採用

スタイリングにはTailwind CSS v4を使用した。

今回のサイトでは、ページごとに大量のCSSを書くよりも、コンポーネントやHTMLの近くでスタイルを管理したかったため、ユーティリティファーストのTailwind CSSを選択した。

### Viteとの統合

Tailwind CSS v4では、`@tailwindcss/vite`を利用してViteと統合している。

これによって、開発中のHMRも含めて軽快にスタイルを反映できる。

### Markdownのスタイリング

Markdownから生成されたHTMLには、Tailwind CSS Typographyの`prose`を利用している。

例えば、

```html
<div class="prose prose-invert prose-emerald">
  <!-- Markdownから生成された記事 -->
</div>
```

のように指定することで、見出し、段落、リスト、コードなどの記事向けのスタイルをまとめて適用できる。

Markdownの記事を増やしていくことを考えると、個別にHTML要素のCSSを書く必要が少ない点はかなり便利だった。

---

## コードブロックをどうするか

今回の実装で特に調整したのが、Markdownの記事内に表示するコードブロックだった。

Astroでは、コードブロックのシンタックスハイライトにShikiを利用できる。

Shikiはコードをトークン単位でハイライトし、テーマに応じたスタイル付きのHTMLを生成する。そのため、単純にコードを表示するだけなら十分きれいに仕上がる。

しかし、実際にブログとして使うことを考えると、

- 何の言語のコードなのか分かりにくい
- コードをコピーするボタンがない
- コードブロック全体のデザインを調整したい

といった部分が気になった。

そこで、Shikiによるハイライトを利用しつつ、コードブロックの外側に独自のUIを追加することにした。

---

## コードブロックのスタイルを調整する

コードブロックを独自デザインにする際に問題になったのが、Shikiによって生成される`style`属性だった。

今回は、`pre`要素に設定されている既存の`style`属性を一度削除し、その後、JavaScriptから必要なスタイルを直接設定する方法にした。

```js
pre.removeAttribute('style');

pre.style.margin = '0';
pre.style.padding = '1rem';
pre.style.background = 'black';
pre.style.overflowX = 'auto';
pre.style.width = '100%';
pre.style.boxSizing = 'border-box';
```

`removeAttribute('style')`で既存のスタイルを取り除いたあと、`pre.style`を使ってコードブロックに必要なスタイルを設定している。

例えば、`margin`と`padding`を指定して余白を整え、`background`で背景を黒にする。また、`overflowX = 'auto'`によって、コードが横に長い場合でも横スクロールできるようにしている。

さらに、`width = '100%'`と`boxSizing = 'border-box'`を指定することで、コードブロックが親要素の幅に収まるようにしている。

ここではTailwind CSSのクラスを追加してスタイルを変更するのではなく、コードブロックをJavaScriptで動的に加工する処理の中で、インラインスタイルを直接設定する方式を採用した。

この方法なら、コードブロックの生成後に必要なスタイルをまとめて設定でき、独自のデザインに合わせて細かく調整できる。

---

## `<pre>`をラッパーで拡張する

コードブロックそのもののスタイルだけではなく、言語名やコピー機能などのUIも追加したかった。

そこで、ページ読み込み後にJavaScriptで`<pre>`要素を取得し、外側に独自のラッパー`div`を追加する方式にした。

イメージとしては、

```html
<div class="code-block">
  <div class="code-header">
    <span>JavaScript</span>
    <button>Copy</button>
  </div>

  <pre>
    <code>...</code>
  </pre>
</div>
```

という構造にする。

こうすることで、コード部分とヘッダー部分を分けて管理できる。

シンタックスハイライトされたコード自体はそのまま利用しながら、その外側に言語名やコピー機能などのUIを追加できるのがこの方式のメリットだった。

---

## まとめ

今回のサイトでは、単に「ブログを作る」のではなく、個人開発で長く使えることを意識して構成を決めた。

Astroによってコンテンツ中心のサイトを静的に生成し、Tailwind CSSでスタイルを管理することで、比較的シンプルな構成にまとめることができた。

また、コードブロックについては、Shikiが生成するスタイルをそのまま利用するのではなく、`pre.removeAttribute('style')`で一度既存のスタイルを外し、`pre.style`で必要なスタイルを直接設定する方法を採用した。

そのうえで`pre`をラッパーで囲み、言語名やコピー機能などを追加することで、単純なコード表示からブログ向けのコードブロックへ拡張した。

今回の実装を通して、既存の仕組みをそのまま使うだけではなく、生成されるHTMLやスタイルを確認し、自分の目的に合わせて必要な部分を作り直すことの重要性を学んだ。
