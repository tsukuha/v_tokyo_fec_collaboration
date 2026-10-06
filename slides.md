---
theme: ./theme
background: ./img/alice-donovan-rouse-PFxSKx4kc5U-unsplash.jpg
title: Nuxt Custom Directives により堅牢な権限管理を実現する
titleTemplate: '%s - Nuxtカスタムディレクティブにより堅牢な権限管理を実現する'
info: Nuxt, カスタムディレクティブ, 権限管理, created by karacoro / からころ
author: karacoro
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/custom/#configure-fonts
fonts:
  sans: ['Yu Gothic', 'Noto Sans JP']
  serif: 'Noto Serif JP'
  mono: 'Fira Code'
  weights: '400,500,700'
  local: ['Yu Gothic']
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-up
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
presenter: true
htmlAttrs:
  lang: ja
---

#  Nuxt Custom Directives により堅牢な権限管理を実現する

### #frontend_unite, karacoro / からころ

<div class="abs-br m-6 mr-18 flex gap-2">
  <a href="https://x.com/karan_corons" target="_blank" alt="X" title="Open in X"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-x />
  </a>
</div>
<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/tsukuha" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<style>
.slidev-layout p {
  opacity: 0.7;
}
.h2 {
  opacity: 0.6;
}
.slidev-layout.cover h1 {
  font-size: 3.65rem !important;
  line-height: 1.4 !important;
}
</style>

---
layout: section-1
background: ./img/section-1.svg
---
# 本日のアジェンダ
1. はじめに
2. 権限管理の種類
3. カスタムディレクティブとは
4. カスタムディレクティブで権限管理を実現する
5. 堅牢なカスタムディレクティブを型で表現するために
6. おわりに

<div class="abs-br mr-2">
  <SlideCurrentNo class="counter" />
</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li {
  font-size: 24px;
}
</style>


---
transition: slide-up
layout: section-2
background: ./img/section-2.svg
---

# はじめに

<div v-click.fade-in class="antipattern">

> role の判定がコンポーネントの中に `v-if="user.role === 'admin'"` のように直接書かれている

</div>

<div v-click.fade-in class="antipattern">

> AI にお願いしたタスクで、同じロール判定のロジックがあちこちのコンポーネントにコピペされ続けている

</div>

<div v-click.fade-in class="antipattern">

> コンポーネントの権限チェックを書き忘れて想定外のユーザに見えてしまった

</div>

<p v-click.fade-in class="assertion assertion-solution">
こういった経験がある方も<br> 多いのではないでしょうか？
</p>

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter"/>
</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.antipattern blockquote {
  margin: 10px 0;
  padding: 10px 20px;
  border-left: 0px;
  background: rgba(98, 166, 255, 0.1);
  border-radius: 4px;
  font-size: 20px;
  line-height: 1.5;
  opacity: 0.85;
}
.assertion {
  padding: 16px 40px 0;
  text-align: center;
  font-weight: 700;
  font-size: 26px;
}
.fadein {
  position: absolute;
  top: 45%;
  left: 20%;
  color: rgb(255, 255, 255);
  opacity: 0;
  animation-name: fadein;
  animation-duration: 1.5s;
  animation-timing-function: ease-out;
  animation-fill-mode: forwards;
}
@keyframes fadein {
  0% {
     opacity: 0;
     transform: translateY(0);
  }
  100% {
     opacity: 1;
     transform: translateY(20px);
  }
}
.assertion-solution {
  color: #1c80ee;
  font-size: 24px;
}
h1 {
   background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
h2 {
  padding-bottom: 8px;
  font-size: 26px;
}
div, p, span, li, button, tr, td {
  font-size: 22px;
  line-height: 1.5;
}
</style>

---
transition: slide-up
layout: section-2
background: ./img/section-2.svg
---

# 権限管理の種類

<div class="description">
権限管理には代表的ないくつかのモデルがあり、<br>
サービスの特性や要件に応じて使い分けられています。
</div>

- **RBAC**（Role-Based Access Control / ロールベースアクセス制御）
  - ユーザーに「ロール」を割り当て、ロール単位で権限を管理する
  - ex. `admin` / `editor` / `viewer` など、役割ごとにできることを定義
- **ABAC**（Attribute-Based Access Control / 属性ベースアクセス制御）
  - ユーザー・リソース・環境などの「属性」を組み合わせて動的に権限を判定する
- **RuBAC**（Rule-Based Access Control / ルールベースアクセス制御）
  - 管理者が定めた「ルール」に従い、全ユーザーに一律で権限を適用する
  - ex. 「業務時間外はアクセス不可」「社内 IP 以外からは閲覧のみ」など
- **ACL**（Access Control List / アクセス制御リスト）
  - リソースひとつひとつに対して「誰が」「何をできるか」を個別に定義する

<p v-click.fade-in class="assertion assertion-banner"><span>今回はABACをベースに、<br>Nuxtのカスタムディレクティブで実現する方法を紹介します。</span></p>

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter"/>
</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.description {
  opacity: 0.8;
  padding-bottom: 16px;
}
.assertion {
  padding: 24px 40px 0;
  text-align: center;
  font-weight: 500;
  font-size: 24px;
}
.assertion-banner {
  position: absolute;
  z-index: 10000;
  inset: -1px;
  margin: 0;
  padding: 0 3.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  font-size: 32px;
  backdrop-filter: blur(2px);
  -webkit-backdrop-filter: blur(2px);
}
.assertion-banner::before {
  content: '';
  position: absolute;
  inset: 0;
  background:
    linear-gradient(rgba(0, 0, 0, 0.65), rgba(0, 0, 0, 0.65)),
    url('./img/alice-donovan-rouse-PFxSKx4kc5U-unsplash.jpg') center / cover no-repeat;
  opacity: 0.9;
}
.assertion-banner span {
  position: relative;
  min-width: 0;
  max-width: 100%;
  font-size: 1.8rem;
  text-align: center;
}
h1 {
   background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
h2 {
  padding-bottom: 8px;
  font-size: 26px;
}
div, p, span, li, button, tr, td {
  font-size: 22px;
  line-height: 1.5;
}
</style>

---
transition: slide-up
layout: section-2
background: ./img/section-2.svg
clicks: 4
---

# カスタムディレクティブとは

<div class="description">
カスタムディレクティブは、DOM 要素に対する処理を <code>v-xxx</code> の形でテンプレートから<br>
宣言的に再利用できる Vue の仕組みです。Nuxt では Plugins としてアプリ全体に登録できる
</div>

<div class="directive-code font-mono" :class="{ focusing: $clicks >= 1 }">
<div><span class="tok-tag">&lt;button </span><span class="part part-name" :class="{ focus: $clicks === 1 }">v-permission</span><span class="part part-arg" :class="{ focus: $clicks === 2 }">:edit</span><span class="part part-mod" :class="{ focus: $clicks === 3 }">.hide</span><span class="tok-tag">=</span><span class="part part-value" :class="{ focus: $clicks === 4 }">"'admin'"</span><span class="tok-tag">&gt;</span><span class="tok-text">編集</span><span class="tok-tag">&lt;/button&gt;</span></div>
</div>

<ul class="anatomy" :class="{ focusing: $clicks >= 1 }">
<li :class="{ focus: $clicks === 1 }"><code>v-permission</code> … ディレクティブ名</li>
<li :class="{ focus: $clicks === 2 }"><code>:edit</code> … 引数（<code>binding.arg</code>）</li>
<li :class="{ focus: $clicks === 3 }"><code>.hide</code> … 修飾子（<code>binding.modifiers</code>）</li>
<li :class="{ focus: $clicks === 4 }"><code>"'admin'"</code> … 値（<code>binding.value</code>）</li>
</ul>

<div class="hooks">
Vueのライフサイクルに合わせて、任意のタイミングでディレクティブの設定されたコンポーネント処理を差し込むことが可能になる。
</div>

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter"/>
</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.description {
  opacity: 0.8;
  padding-bottom: 12px;
}
.directive-code {
  margin: 4px 0 12px;
  padding: 12px 16px;
  border-radius: 6px;
  background: var(--slidev-code-background, #f5f5f5);
  font-size: 18px;
  line-height: 1.8;
}
.directive-code div,
.directive-code span {
  font-size: 18px;
}
.tok-comment {
  opacity: 0.5;
}
.tok-tag {
  color: #22863a;
}
.tok-text {
  color: inherit;
}
.part-name { color: #6f42c1; }
.part-arg { color: #e36209; }
.part-mod { color: #d73a49; }
.part-value { color: #032f62; }
html.dark .tok-tag { color: #85e89d; }
html.dark .part-name { color: #b392f0; }
html.dark .part-arg { color: #ffab70; }
html.dark .part-mod { color: #f97583; }
html.dark .part-value { color: #9ecbff; }
.part {
  padding: 1px 2px;
  border-radius: 4px;
  transition: opacity 0.3s, background-color 0.3s;
}
.directive-code.focusing .tok-tag,
.directive-code.focusing .tok-text,
.directive-code.focusing .tok-comment,
.directive-code.focusing .part:not(.focus) {
  opacity: 0.3;
}
.directive-code .part.focus {
  background: rgba(98, 166, 255, 0.25);
  font-weight: 700;
}
.anatomy li {
  font-size: 18px;
  line-height: 1.6;
  transition: opacity 0.3s;
}
.anatomy.focusing li:not(.focus) {
  opacity: 0.3;
}
.anatomy li.focus {
  font-weight: 700;
}
.hooks {
  margin-top: 12px;
  padding: 10px 16px;
  border-radius: 4px;
  background: rgba(98, 166, 255, 0.1);
  font-size: 16px;
}
.hooks code {
  font-size: 14px;
}
h1 {
   background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 20px;
  line-height: 1.5;
}
</style>

---
transition: slide-up
layout: section-two-cols
---

# カスタムディレクティブの使い方
① <code>plugins/</code> でディレクティブを登録 → ② テンプレートで <code>v-xxx</code> として使う

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter" />
</div>

<p class="summary"><code>
defineNuxtPlugin()</code> の内部で、 <br><code>nuxtApp.vueApp.directive()</code> を呼び出すことでアプリ全体に登録される


</p>

::left::

<p class="file-name">plugins/permission.ts</p>

```ts {*|1|2|3}
export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.directive('permission', {
    mounted(el: HTMLElement, binding) {
      // ここの処理を記載する
      // binding: { modifiers, value }
    },
  })
})
```

::right::

<p class="file-name">pages/index.vue</p>

```vue {*|3|5}
<template>
  <!-- admin 以外には表示しない -->
  <button v-permission.hide="'admin'">削除</button>
  <!-- admin 以外は押せない -->
  <button v-permission="'admin'">編集</button>
</template>
```

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.file-name {
  font-size: 18px;
}
.summary {
  position: absolute;
  left: 160px;
  right: 0;
  bottom: 3.5rem;
  margin: 0;
  text-align: left;
  font-size: 20px;
}
.note {
  margin-top: 16px;
  padding: 8px 12px;
  border-radius: 6px;
  background: rgba(178, 207, 249, 0.3);
}
.note li {
  font-size: 15px;
  line-height: 1.6;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 16px;
}
</style>

---
transition: slide-up
layout: section-two-cols
---

# カスタムディレクティブで権限管理を実現する
テンプレートには要素IDのみ記載し、ロールに応じた判定と状態管理はディレクティブ側に集約する

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter" />
</div>

::left::

<p class="file-name">pages/users.vue</p>

```vue {*|2|3|4}
<template>
  <button v-permission="'users-create-button'">新規作成</button>
  <button v-permission="'users-delete-button'">削除</button>
  <input v-permission="'users-email-input'" />
</template>
```

<div class="note">

- 対象要素に 権限ID を割り振り、ページごとに一覧で管理する
- 判定（ロール・本人かどうか等）はサーバーがJSON生成時に行う
- JSONにない要素はそのまま表示、ある要素は `status` を反映する
- 例：右のJSONでは、新規作成は押せず、削除ボタン・メール欄はそのまま表示

</div>

::right::

<p class="file-name">権限JSON（ユーザーごとにサーバーで生成）</p>

```json {*|5|7|8-12}
{
  "display": [{
    "group_id": "user_management",
    "features": [{
      "id": "users",
      "items": [{
        "id": "users-create-button",
        "status": {
          "show": true,
          "disabled": true,
          "readonly": false
        }
      }]
    }]
  }]
}
```

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.file-name {
  font-size: 16px;
}
:deep(.wrapper .col:first-child) {
  width: 58%;
}
:deep(.wrapper .col:last-child) {
  width: 42%;
}
.note {
  margin-top: 16px;
  padding: 8px 12px;
  border-radius: 6px;
  background: rgba(178, 207, 249, 0.3);
}
.note li {
  font-size: 15px;
  line-height: 1.6;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 16px;
}
</style>

---
transition: slide-up
layout: section-two-cols
---

# 権限JSONの取得と画面遷移の制御
画面遷移のタイミングで権限JSONを取得し、遷移先ページへのアクセス可否を判定する

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter" />
</div>

::left::

<p class="file-name">middleware/permission.global.ts</p>

```ts {*|2|4-7|9-13}
export default defineNuxtRouteMiddleware(async (to) => {
  if (import.meta.server) return

  const permissionStore = usePermissionStore()
  if (!permissionStore.isLoaded) {
    await permissionStore.fetchPermissions()
  }

  const featureId = String(to.name)
  if (!permissionStore.hasFeature(featureId)) {
    showError({ statusCode: 403, message: '権限がありません' })
    return navigateTo(permissionStore.getRedirectPath(featureId))
  }
})
```

::right::

<div class="steps">

1. 権限JSONは DOM と同じくクライアントで扱うため、サーバー実行時はスキップ
2. ログインユーザーの権限JSONをサーバーから取得<br>（取得済みならスキップ）
3. 遷移先ページの機能ID（ルート名）を含むグループが権限JSONにあるか確認
    - あれば、そのまま遷移
    - なければ、権限エラーを表示して<br>遷移先に応じたページへリダイレクト

</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.file-name {
  font-size: 18px;
}
:deep(.wrapper .col:first-child) {
  width: 58%;
}
:deep(.wrapper .col:last-child) {
  width: 42%;
  padding-top: 40px;
}
.steps li {
  font-size: 16px;
  line-height: 1.8;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 16px;
}
</style>

---
transition: slide-up
layout: section-two-cols
---

# v-permission の実装
現在のページと要素IDから権限JSONの <code>status</code> を引き、要素に反映する

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter" />
</div>

::left::

<p class="file-name">plugins/permission.client.ts</p>

```ts {*|4-5|6-7|8|9-10|11-14}
export function applyPermission(
  el: HTMLElement, binding: DirectiveBinding<string>,
) {
  const store = usePermissionStore()
  if (!store.isLoaded) return
  const featureId = String(useRoute().name)
  const status = store.getElementStatus(featureId, binding.value)
  if (status == null) return
  if (!status.show) {
    el.style.display = 'none'
  } else if (status.disabled) {
    el.setAttribute('disabled', 'true')
  } else if (status.readonly) {
    el.setAttribute('readonly', 'true')
  }
}
```

<p class="file-name">stores/permission.ts（抜粋）</p>

```ts
function getElementStatus(featureId: string, id: string) {
  return permissions.value.display
    .flatMap((group) => group.features)
    .find((feature) => feature.id === featureId)
    ?.items.find((item) => item.id === id)
    ?.status
}
```

::right::

<div class="steps">

1. 権限JSONが未取得なら何もしない
2. 現在のページの機能ID（ルート名）と要素IDから `status` を取得
3. 権限JSONに存在しない要素は何もしない（そのまま表示）
4. `show: false` なら非表示（以降の値は無視）
5. `disabled` → `readonly` の順に反映

</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.file-name {
  font-size: 16px;
}
:deep(.wrapper .col:first-child) {
  width: 64%;
}
:deep(.wrapper .col:last-child) {
  width: 36%;
  padding-top: 40px;
}
.steps li {
  font-size: 16px;
  line-height: 1.8;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 16px;
}
</style>

---
transition: slide-up
layout: section-two-cols
---

# v-permission の登録と実装のポイント
判定ロジックを描画前のタイミングで呼び出す

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter" />
</div>

::left::

<p class="file-name">plugins/permission.client.ts</p>

```ts {*|3|4}
export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.directive('permission', {
    beforeMount: applyPermission,
    updated: applyPermission,
  })
})
```

<div class="note">

もちろん、クライアントの表示制御だけでなく<br>
バックエンドでのハンドリングも必須。

</div>

::right::

<div class="points">

- **`beforeMount`**：描画前に判定し、一瞬表示されてから消える「ちらつき」やレイアウト崩れを防ぐ
- **`updated`**：権限JSONやバインド値の変更に追従する
- **ページと要素で既定の挙動を分ける**：`features` に機能IDがなければページに入れない。要素IDがなければ制限せず、そのまま表示する
- **`display: none`**：`el.remove()` で要素を取り除くと、Vue の再レンダリング時に問題が発生する可能性があるため、非表示にとどめる

</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.file-name {
  font-size: 18px;
}
.note {
  margin-top: 16px;
  padding: 8px 12px;
  border-radius: 6px;
  background: rgba(178, 207, 249, 0.3);
  font-size: 15px;
  line-height: 1.6;
}
.points li {
  font-size: 15px;
  line-height: 1.6;
  margin-bottom: 4px;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 16px;
}
</style>

---
transition: slide-up
layout: section-2
background: ./img/section-2.svg
---

# 堅牢なカスタムディレクティブを型で表現するために

<div class="description">
カスタムディレクティブに渡す値は、特に何もしなければ型チェックの対象にはならない。権限IDをタイポしてもエラーにならないので、このままでは開発者体験も悪く堅牢性に欠ける。
</div>

```vue
<!-- タイポしているのにエラーにならない -->
<button v-permission="'users-delte-button'">削除</button>
```

<div class="measures">

1. 権限IDを定数と型で一元管理する
2. `GlobalDirectives` でテンプレートに型を効かせる
3. 型チェックを開発フローに組み込む
4. 動的に来る値は実行時にも検証する

</div>

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter"/>
</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.description {
  opacity: 0.8;
  padding-bottom: 12px;
}
.measures {
  margin-top: 16px;
  padding: 8px 16px;
  border-radius: 6px;
  background: rgba(98, 166, 255, 0.1);
}
.measures li {
  font-size: 20px;
  line-height: 1.7;
}
h1 {
   background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 20px;
  line-height: 1.5;
}
</style>

---
transition: slide-up
layout: section-two-cols
---

# ① 権限IDを定数と型で一元管理する
機能（ページ）ごとの権限IDを定数で定義し、そこから型を導出する

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter" />
</div>

<p class="file-name">constants/features.ts</p>

```ts {*|1-9|9|11|12-14}
export const FEATURES = {
  users: {
    items: {
      'users-create-button': { label: '新規作成' },
      'users-delete-button': { label: '削除' },
      'users-email-input': { label: 'メールアドレス' },
    },
  },
} as const satisfies Record<string, FeatureDefinition>

export type FeatureId = keyof typeof FEATURES
export type ElementId<F extends FeatureId = FeatureId> = {
  [K in P]: keyof (typeof FEATURES)[K]['items']
}[P]
```

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.file-name {
  font-size: 16px;
}
:deep(.wrapper .col:first-child) {
  width: 62%;
}
:deep(.wrapper .col:last-child) {
  width: 38%;
  padding-top: 32px;
}
.points li {
  font-size: 15px;
  line-height: 1.6;
  margin-bottom: 4px;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 16px;
}
</style>

---
transition: slide-up
layout: section-two-cols
---

# ② GlobalDirectives でテンプレートに型を効かせる
ディレクティブに渡す値の型を宣言し、テンプレート上で型チェックさせる

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter" />
</div>

::left::

<p class="file-name">types/directives.d.ts</p>

```ts {*|4|6-10|11-15}
import type { Directive } from 'vue'
import type { ElementId } from '~/constants/features'

type VPermission = Directive<HTMLElement, ElementId>

declare module '@vue/runtime-core' {
  interface GlobalDirectives {
    vPermission: VPermission
  }
}
declare module '@vue/runtime-dom' {
  interface GlobalDirectives {
    vPermission: VPermission
  }
}
```

::right::

<p class="file-name">pages/users.vue</p>

```vue {*|2-3|4-5}
<template>
  <!-- OK -->
  <button v-permission="'users-delete-button'" />
  <!-- 型エラー：存在しない要素ID -->
  <button v-permission="'users-delte-button'" />
</template>
```

<div class="note">

- `@vue/runtime-core` と `@vue/runtime-dom` の両方に同じ宣言を行う（`runtime-dom` 側がないと VSCode でテンプレートの補完が効かない）
- エディタ上と `nuxi typecheck`（vue-tsc）で、タイポを書いた時点で検出できる
- `binding.value` も `ElementId` 型になり、判定ロジック側でも型の恩恵を受けられる

</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.file-name {
  font-size: 16px;
}
.note {
  margin-top: 12px;
  padding: 8px 12px;
  border-radius: 6px;
  background: rgba(178, 207, 249, 0.3);
  font-size: 15px;
  line-height: 1.6;
}
.note li {
  font-size: 14px;
  line-height: 1.6;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
div, p, span, li, button, tr, td {
  font-size: 16px;
}
</style>

<div class="note">

- 権限JSONはサーバーで動的に生成されるため、受け取った値は実行時にも検証する
- 型ガードにすると、検証後はそのまま型安全に扱える
- 判定ロジックは関数として export し、単体テストする

</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.file-name {
  font-size: 16px;
}
.note {
  margin-top: 12px;
  padding: 8px 12px;
  border-radius: 6px;
  background: rgba(178, 207, 249, 0.3);
}
.note li {
  font-size: 14px;
  line-height: 1.6;
}
h1 {
  background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
  font-size: 28px;
}
div, p, span, li, button, tr, td {
  font-size: 16px;
}
</style>

---
transition: slide-up
layout: section-2
background: ./img/section-2.svg
---

# おわりに

- Nuxtでカスタムディレクティブで権限管理を行う方法について説明した
- 権限の判定は、カスタムディレクティブに集約する
- 要素IDを型で管理して、堅牢な権限管理を実現する

<img class="duck" src="./img/duck-umbrella.png" alt="黄色い傘を頭に刺したアヒル" />

<div class="abs-br mr-2 counter">
  <SlideCurrentNo class="counter"/>
</div>

<style>
html {
  color: #3e3e3e;
}
html.dark {
  color: #efefef;
}
.counter {
  padding-bottom: 4px;
  font-family: "メイリオ";
  font-size: 12px;
}
.duck {
  position: absolute;
  right: 3rem;
  bottom: 2rem;
  height: 240px;
}
.description {
  opacity: 0.8;
  padding-bottom: 16px;
}
h1 {
   background-image: linear-gradient(90deg, rgba(167, 199, 240, 1), rgba(178, 207, 249, 1) 0%, rgba(98, 166, 255, 1) 26%, rgba(28, 128, 238, 1) 70%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: 500;
  padding-bottom: 8px;
}
h2 {
  padding-bottom: 8px;
  font-size: 26px;
}
div, p, span, li, button, tr, td {
  font-size: 22px;
  line-height: 1.8;
}
</style>
