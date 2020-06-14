---
title: もしもVue.jsにTypescriptを導入したら
tags: Vue, Typescript
description: View the slide with "Slide Mode".
slideOptions:
    theme: white
    slideNumber: 'c/t'
    transition: 'slide'
    keyboard: true

---

# もしもVue.jsにTypescriptを導入したら ①調査編

---

## まずは現行環境を整理

仮にメディアの管理画面とすると・・・
<!-- .element: class="fragment fade-up" -->

- Laravel 6
- Vue 2.5
- vuex 3.0
- vue-router 3.0
- element-ui 2.4
<!-- .element: class="fragment fade-up" -->

---

### とりあえず調べてみる

VueにTypescriptを導入するにはVue 2.5以上、Typescript 2.4以上が必要らしい。
そしてTypescriptを使うには3パターンがある
<!-- .element: class="left fragment fade-up" -->

1. `Vue.extend`だけ使う<!-- .element: class="fragment fade-up" -->
2. `vue-class-component`を使う<!-- .element: class="fragment fade-up" -->
3. `vue-property-decorator`を使う<!-- .element: class="fragment fade-up" -->

---

Vue公式
- やって来る Vue 2.5 での TypeScript の変更
https://jp.vuejs.org/2017/09/23/upcoming-typescript-changes-in-vue-2.5/

---

### 1. `Vue.extend`だけ使う

Vue公式より
> Vue コンポーネントオプション内部で TypeScript が型を適切に推測できるようにするには、Vue.component または Vue.extend でコンポーネントを定義する必要があります:

---

:thinking_face:

<!-- .slide: data-transition="fade-in" -->

---

- つまり Vue.extend は基本的に必要らしい
- ちなみに Vue.component を使っても内部的に Vue.extend を呼んでるらしい
- それとVue コンポーネントオプションというのは dataとかpropsのことだ

---

Vue公式
- コンポーネント
https://v1-jp.vuejs.org/guide/components.html
- TypeScript のサポート
https://jp.vuejs.org/v2/guide/typescript.html

---

### 2. `vue-class-component`を使う

公式ドキュメントより
> Vueクラスコンポーネントは、クラススタイルの構文でVueコンポーネントを作成できるライブラリです。

**つまりライブラリです**
<!-- .element: class="fragment fade-up" -->

---

GitHub
https://github.com/vuejs/vue-class-component

公式ドキュメント
https://class-component.vuejs.org/

---

### 3. `vue-property-decorator`を使う

GitHubより
> This library fully depends on vue-class-component, so please read its README before using this library.

---

訳
> このライブラリは、vue-class-componentに完全に依存しているため、このライブラリを使用する前に、READMEをお読みください。

**つまりライブラリです。**
**vue-class-componentの拡張版**
<!-- .element: class="fragment fade-up" -->

---

GitHub
https://github.com/kaorun343/vue-property-decorator

---

# で？結局どれ使えばいいの？

---

## さらに調べてみる

- "vue typescript"で検索するとvue-property-decoratorが一番ヒットするのでさらに調べてみる。
<!-- .element: class="fragment fade-up" -->
- どうやらこのライブラリはtypescriptの機能である「デコレータ」を使ってかけるものらしい。
<!-- .element: class="fragment fade-up" -->

---

### デコレータとは、クラス宣言とメンバーの注釈とメタプログラミング構文の両方を追加する方法を提供する

Typescript公式ドキュメント
- デコレータ
https://www.typescriptlang.org/docs/handbook/decorators.html

---

:thinking_face: :thinking_face:
<!-- .slide: data-transition="fade-in" -->

---

# まあとりあえずどんな感じに書くん？

---

```typescript
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Prop(Number) readonly propA: number | undefined
  @Prop({ default: 'default value' }) readonly propB!: string
  @Prop([String, Boolean]) readonly propC: string | boolean | undefined
}
```

```typescript
import { Vue, Component, Emit } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  count = 0

  @Emit()
  addToCount(n: number) {
    this.count += n
  }

  @Emit('reset')
  resetCount() {
    this.count = 0
  }

  @Emit()
  returnValue() {
    return 10
  }

  @Emit()
  onInputChange(e) {
    return e.target.value
  }

  @Emit()
  promise() {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve(20)
      }, 0)
    })
  }
}
```

---

## なにコレ書きづらっ・・・

こうサクッとTypescriptの型定義の恩恵を受けたいのよ
<!-- .element: class="fragment fade-up" -->

ちなみに@xxxがクラスデコレータ、メソッドデコレータというやつらしい。@の下のクラスやメソッドを@xxx側で監視、変更、または置換できるらしい
<!-- .element: class="fragment fade-up" -->

詳しくは公式ドキュメントをチェック
https://www.typescriptlang.org/docs/handbook/decorators.html
<!-- .element: class="fragment fade-up" -->

---

じゃあ次にvue-class-componentを使うとするとどうなるのよ

1. `Vue.extend`だけ使う
2. `vue-class-component`を使う
3. ~~`vue-property-decorator`を使う~~



---

このvue-class-componentには気をつけなければいけない点があって、propsのAPIが定義されていないんです。（ドキュメントを読んだ感じwatchとかemitとかも同様）
propsはVue.extendしてあげないといけないらしく、つまり・・・

Vue Class Component公式
- Props Definition
https://class-component.vuejs.org/guide/props-definition.html

---

## こんな感じ

```typescript
import Vue from 'vue'
import Component from 'vue-class-component'

// Define the props by using Vue's canonical way.
const GreetingProps = Vue.extend({
  props: {
    name: String
  }
})

// Use defined props by extending GreetingProps.
@Component
export default class Greeting extends GreetingProps {
  get message(): string {
    // this.name will be typed
    return 'Hello, ' + this.name
  }
}
```

---

## えぇ・・・。

一つにまとめれないんかい。そのためのvue-property-decoratorらしいです
<!-- .element: class="fragment fade-up" -->

なので選択肢としては実質、Vue.extendだけかvue-property-decoratorを使うかのようです
<!-- .element: class="fragment fade-up" -->

ということでVue.extendだけ使う方法でTypescriptを導入してみたいと思います
<!-- .element: class="fragment fade-up" -->

1. `Vue.extend`だけ使う
2. ~~`vue-class-component`を使う~~
3. ~~`vue-property-decorator`を使う~~
<!-- .element: class="fragment fade-up" -->

---

# もしもVue.jsにTypescriptを導入したら ②実践編

---

とりあえず`vue create`で作ったVue+TypescriptのサンプルPJと、Vue公式やマイクロソフト様が書いたありがたいドキュメント等を参考にしていきます

- TypeScript Vue Starter
https://github.com/microsoft/TypeScript-Vue-Starter
- TypeScript のサポート
https://jp.vuejs.org/v2/guide/typescript.html

---

### 管理画面側に各種設定

- `npm i -D typescript ts-loader`で必要なモジュールをインストール
<!-- .element: class="fragment fade-up" -->
- `./node_modules/.bin/tsc --version`でインストール確認
<!-- .element: class="fragment fade-up" -->
- サンプルPJを参考に必要そうなconfigを設定。詳細は公式ドキュメント参照
<!-- .element: class="fragment fade-up" -->

---

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "module": "es6",
    "strict": true,
    "importHelpers": true,
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "sourceMap": true,
    "lib": [
      "es6",
      "dom",
      "dom.iterable",
      "scripthost"
    ]
  },
  "include": [
    "resources/js/**/*.ts",
    "resources/js/**/*.tsx",
    "resources/js/**/*.vue",
  ],
  "exclude": [
    "node_modules"
  ]
}
```

---

Typescript公式ドキュメント
- tsconfig.json
https://www.typescriptlang.org/docs/handbook/tsconfig-json.html
- Compiler Options
https://www.typescriptlang.org/docs/handbook/compiler-options.html

---

### webpack.mix.jsを修正

```js
- mix.js('resources/js/app.js', 'public/js/app.js')
+ mix.ts('resources/js/app.ts', 'public/js/app.js')
```

---

### vue-shims.d.tsファイルを追加

`.vue`ファイルがなんであるかをTypescritpに伝えるためにvue-shims.d.tsファイルを追加する必要があるらしいので追加。includeするディレクトリ配下で良いらしい

```typescript
// resources/js/shims-vue.d.ts

declare module "*.vue" {
    import Vue from "vue";
    export default Vue;
}
```

---

app.jsをTypescriptに書き換え

ポイント
- TypeScript未対応のjsをimportしようとすると定義ファイルが見つからないエラーになるので一旦requireなどで対応する。（ベストは`.d.ts`の定義ファイルを作ることらしい）
<!-- .element: class="fragment fade-up" -->
- さらにrequireを使うためにはnodeの定義ファイルが必要になるので`npm i @types/node`でインストール
<!-- .element: class="fragment fade-up" -->
- vueファイルをimportする際は拡張子をつける必要あり
<!-- .element: class="fragment fade-up" -->

---

### router.jsをTypescriptに書き換え

RouteConfig型でroutesを作成する必要があるらしい

▼サンプルPJ
```typescript
import Vue from 'vue'
import VueRouter, { RouteConfig } from 'vue-router'
import Home from '../views/Home.vue'

Vue.use(VueRouter)

  const routes: Array<RouteConfig> = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  }
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router

```

---

### つらみポイント

既存ライブラリの型定義ファイル(d.ts)に沿った使い方をしていない場合や、そもそも実装と定義ファイルに差異があってメソッドが足りない場合などは自前で型定義を拡張しなければならない
<!-- .element: class="fragment fade-up" -->

その場合はdeclare(アンビエント宣言)を拡張します。コレをDeclaration Mergingというらしい
<!-- .element: class="fragment fade-up" -->

---

こんな感じに

```typescript
declare module 'element-ui/types/message' {
    interface ElMessage {
        closeAll(): void
    }
}
```

参考サイト
- 既存ライブラリのクラス拡張
https://kakkoyakakko2.hatenablog.com/entry/2018/06/28/003000

---

色々試行錯誤した結果・・・

# 動いた！！ログインもできた
<!-- .element: class="fragment fade-up" -->

---

vueファイルもTypescript化してみる

ブラックリストURL登録ページ

### Before

```typescript
<script>
import * as types from '../../../store/mutation-types'
import Pagination from '../../../components/Pagination'
import RegistrationModal from './Blacklist/RegistrationModal'

export default {
    components: {
        Pagination,
        RegistrationModal
    },
    data() {
        return {
            blacklists: [],
            pagination: {},
            path: '',
            registration : {
                isActive: false,
            },
        }
    },
    ...
    methods {
        async destroyBlacklist(blacklistId) {
        ...
    }
}
```

---

### After

```typescript
<script lang="ts">
import Vue from 'vue'
import axios from 'axios'
import * as types from '../../../store/mutation-types'
import Pagination from '../../../components/Pagination.vue'
import RegistrationModal from './Blacklist/RegistrationModal.vue'

export type DataType = {
    blacklists: any[],
    pagination: object,
    path: string,
    registration : {
        isActive: boolean,
    },
}

export default Vue.extend({
    components: {
        Pagination,
        RegistrationModal
    },
    data(): DataType {
        return {
            blacklists: [],
            pagination: {},
            path: '',
            registration : {
                isActive: false,
            },
        }
    },
    ...
    methods {
        async destroyBlacklist(blacklistId: number) {
        ...
    }
}
```

---

ええんちゃう〜？

---

まとめ

- 既存のVueをts化はいけなくはなさそう
- ただ既存ライブラリが絡む部分はかなりハマった(どういう型指定するば良いのか分からない)
- 今回は付け焼き刃的に対応してみたが、Typescriptの基礎を知らないと辛い
- 正直Vue3.0で正式対応版きたらそれにのっかるのがベストだと思う

---

今回参考にしたページ
- [Vue+TypeScript] Vue.extend で Vue らしさを保ちつつ TypeScript で書くときの型宣言についてまとめた
https://qiita.com/is_ryo/items/6fc799ba4214db61d8ab

- とにかく楽してVue.jsでTypeScriptを使いたい
https://www.slideshare.net/sakura_pr/vuejstype-script-81228009

- Revised Revised 型の国のTypeScript
http://typescript.ninja/typescript-in-definitelyland/typescript-basic.html

- 仕事ですぐに使えるTypeScript
https://future-architect.github.io/typescript-guide/index.html

---
