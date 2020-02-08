# Alpine.js

![npm bundle size](https://img.shields.io/bundlephobia/minzip/alpinejs)

Alpine.js は、Vue や React などの大きなフレームワークのリアクティブで宣言的な性質をはるかに低いコストで提供します。

DOMを保持し、適切な動作を施すことができます。

[Tailwind](https://tailwindcss.com/) の JavaScript 版のようなものです。

> 注意: このツールのシンタックスは、ほぼ完全に [Vue](https://vuejs.org/) (それと、[Angular](https://angularjs.org/)による拡張)から借用しています。ウェブからの賜り物に感謝しています。

## Install

**CDNより:** `<head>` セクションの最後に次のスクリプトを追加します。
```html
<script src="https://cdn.jsdelivr.net/gh/alpinejs/alpine@v1.9.7/dist/alpine.js" defer></script>
```

それだけです。初期は自身で行われます。

**NPMより:** NPM からパッケージをインストールします。
```js
npm i alpinejs
```

各自スクリプトでインクルードします。
```js
import 'alpinejs'
```

IE11 では、ポリフィルを提供する必要があります。 次のスクリプトを上記の Alpine スクリプトの前にロードしてください。
```html
<script src="https://polyfill.io/v3/polyfill.min.js?features=MutationObserver%2CArray.from%2CArray.prototype.forEach%2CMap%2CSet%2CArray.prototype.includes%2CString.prototype.includes%2CPromise%2CNodeList.prototype.forEach%2CObject.values%2CReflect%2CReflect.set"></script>

<script src="https://cdn.jsdelivr.net/npm/proxy-polyfill@0.3.0/proxy.min.js"></script>
```

## 使う

*ドロップダウン/モーダル*
```html
<div x-data="{ open: false }">
    <button @click="open = true">Open Dropdown</button>

    <ul
        x-show="open"
        @click.away="open = false"
    >
        Dropdown Body
    </ul>
</div>
```

*タブ*
```html
<div x-data="{ tab: 'foo' }">
    <button :class="{ 'active': tab === 'foo' }" @click="tab = 'foo'">Foo</button>
    <button :class="{ 'active': tab === 'bar' }" @click="tab = 'bar'">Bar</button>

    <div x-show="tab === 'foo'">Tab Foo</div>
    <div x-show="tab === 'bar'">Tab Bar</div>
</div>
```

自明ではないことにも使用できます:
*ホバー時にドロップダウンのHTMLコンテンツをプリフェッチする*
```html
<div x-data="{ open: false }">
    <button
        @mouseenter.once="
            fetch('/dropdown-partial.html')
                .then(response => response.text())
                .then(html => { $refs.dropdown.innerHTML = html })
        "
        @click="open = true"
    >Show Dropdown</button>

    <div x-ref="dropdown" x-show="open" @click.away="open = false">
        Loading Spinner...
    </div>
</div>
```

## Learn

次の13のディレクティブを使用できます。

| ディレクティブ
| --- |
| [`x-data`](#x-data) |
| [`x-init`](#x-init) |
| [`x-show`](#x-show) |
| [`x-bind`](#x-bind) |
| [`x-on`](#x-on) |
| [`x-model`](#x-model) |
| [`x-text`](#x-text) |
| [`x-html`](#x-html) |
| [`x-ref`](#x-ref) |
| [`x-if`](#x-if) |
| [`x-for`](#x-for) |
| [`x-transition`](#x-transition) |
| [`x-cloak`](#x-cloak) |

それと5つのマジックプロパティ：

| マジックプロパティ
| --- |
| [`$el`](#el) |
| [`$refs`](#refs) |
| [`$event`](#event) |
| [`$dispatch`](#dispatch) |
| [`$nextTick`](#nexttick) |


### ディレクティブ

---

### `x-data`

**例:** `<div x-data="{ foo: 'bar' }">...</div>`

**構造:** `<div x-data="[JSON data object]">...</div>`

`x-data` は新しいコンポーネントスコープを宣言します。フレームワークに、データオブジェクトを使用して新しいコンポーネントを初期化するよう指示します。

Vueコンポーネントの `data`プロパティのように考えてください。

**コンポーネントロジックの抽出**

データ（と動作）を再利用可能な関数に抽出できます:

```html
<div x-data="dropdown()">
    <button x-on:click="open()">Open</button>

    <div x-show="isOpen()" x-on:click.away="close()">
        // Dropdown
    </div>
</div>

<script>
    function dropdown() {
        return {
            show: false,
            open() { this.show = true },
            close() { this.show = false },
            isOpen() { return this.show === true },
        }
    }
</script>
```

オブジェクトの構造化を使用して、複数のデータオブジェクトを混在も出来ます:

```html
<div x-data="{...dropdown(), ...tabs()}">
```

---

### `x-init`
**例:** `<div x-data="{ foo: 'bar' }" x-init="foo = 'baz'"></div>`

**構造:** `<div x-data="..." x-init="[式]"></div>`

`x-init` はコンポーネントが初期化されると式を実行します。

Alpine が DOM（VueJS の `mounted()` フックのようなもの）に最初の更新を行った後にコードを実行したい場合、 `x-init` からコールバックを返すことができ、その後実行されます：

`x-init="return () => { // ここで初期化後の DOM ステートにアクセスできます // }"`

---

### `x-show`
**例:** `<div x-show="open"></div>`

**構造:** `<div x-show="[式]]"></div>`

`x-show` は、式が `true` または `false` のどちらかの結果によって、要素の `display: none;` スタイルを切り替えます。

---

### `x-bind`

> 注意: 短い ":" シンタックスを使えます: `:type="..."`

**例:** `<input x-bind:type="inputType">`

**構造:** `<input x-bind:[属性]="[式]">`

`x-bind` は、属性の値を JavaScript 式の結果を設定します。この式は、コンポーネントのデータオブジェクトのすべてのキーにアクセスでき、データが更新されるたびに反映されます。

> 注意: 属性バインディングは、依存関係が更新されたときにのみ更新されます。 このフレームワークは、データの変化を観察し、どのバインディングがそれらを検出するのか最適化されています。

**`x-bind` for class attributes**

`x-bind` は、`class` 属性にバインドするときの動作が少し異なります。

クラスの場合、キーがクラス名であり、値がそれらのクラス名が適用されるかどうかを決定するブール式であるオブジェクトを渡します。

例:
`<div x-bind:class="{ 'hidden': foo }"></div>`

この例では、"hidden"クラスは、`foo` データ属性値が `true` の場合にのみ適用されます。

**ブール属性の `x-bind`**

`x-bind` は、値属性と同じ方法でブール値属性をサポートします。条件として変数を使用するか、`true` または `false` に解決される JavaScript 式を使用します。

例:
`<button x-bind:disabled="myVar">Click me</button>`

`myVar` が true または false の場合に `disabled` 属性を追加または削除します。

`readonly`、`required` など、最も一般的なブール属性がサポートされています。

---

### `x-on`

> 注意： より短い"@"シンタックスを自由に使用できます: `@click="..."`

**例:** `<button x-on:click="foo = 'bar'"></button>`

**構造:** `<button x-on:[event]="[expression]"></button>`

`x-on` は、イベントリスナを宣言された要素にアタッチします。そのイベントが発行されると、その値として設定された JavaScript 式が実行されます。

式でデータが変更されると、このデータに"バインドされた"他の要素属性が更新されます。

**`keydown` modifiers**

**Example:** `<input type="text" x-on:keydown.escape="open = false">`

You can specify specific keys to listen for using keydown modifiers appended to the `x-on:keydown` directive. Note that the modifiers are kebab-cased versions of `Event.key` values.

Examples: `enter`, `escape`, `arrow-up`, `arrow-down`

> Note: You can also listen for system-modifier key combinations like: `x-on:keydown.cmd.enter="foo"`

**`.away` modifier**

**Example:** `<div x-on:click.away="showModal = false"></div>`

When the `.away` modifier is present, the event handler will only be executed when the event originates from a source other than itself, or its children.

This is useful for hiding dropdowns and modals when a user clicks away from them.

**`.prevent` modifier**
**Example:** `<input type="checkbox" x-on:click.prevent>`

Adding `.prevent` to an event listener will call `preventDefault` on the triggered event. In the above example, this means the checkbox wouldn't actually get checked when a user clicks on it.

**`.stop` modifier**
**Example:** `<div x-on:click="foo = 'bar'"><button x-on:click.stop></button></div>`

Adding `.stop` to an event listener will call `stopPropagation` on the triggered event. In the above example, this means the "click" event won't bubble from the button to the outer `<div>`. Or in other words, when a user clicks the button, `foo` won't be set to `'bar'`.

**`.window` modifier**
**Example:** `<div x-on:resize.window="isOpen = window.outerWidth > 768 ? false : open"></div>`

Adding `.window` to an event listener will install the listener on the global window object instead of the DOM node on which it is declared. This is useful for when you want to modify component state when something changes with the window, like the resize event. In this example, when the window grows larger than 768 pixels wide, we will close the modal/dropdown, otherwise maintain the same state.

>Note: You can also use the `.document` modifier to attach listeners to `document` instead of `window`

**`.once` modifier**
**Example:** `<button x-on:mouseenter.once="fetchSomething()"></button>`

Adding the `.once` modifier to an event listener will ensure that the listener will only be handled once. This is useful for things you only want to do once, like fetching HTML partials and such.

---

### `x-model`
**Example:** `<input type="text" x-model="foo">`

**Structure:** `<input type="text" x-model="[data item]">`

`x-model` adds "two-way data binding" to an element. In other words, the value of the input element will be kept in sync with the value of the data item of the component.

> Note: `x-model` is smart enough to detect changes on text inputs, checkboxes, radio buttons, textareas, selects, and multiple selects. It should behave [how Vue would](https://vuejs.org/v2/guide/forms.html) in those scenarios.

---

### `x-text`
**Example:** `<span x-text="foo"></span>`

**Structure:** `<span x-text="[expression]"`

`x-text` works similarly to `x-bind`, except instead of updating the value of an attribute, it will update the `innerText` of an element.

---

### `x-html`
**Example:** `<span x-html="foo"></span>`

**Structure:** `<span x-html="[expression]"`

`x-html` works similarly to `x-bind`, except instead of updating the value of an attribute, it will update the `innerHTML` of an element.

---

### `x-ref`
**Example:** `<div x-ref="foo"></div><button x-on:click="$refs.foo.innerText = 'bar'"></button>`

**Structure:** `<div x-ref="[ref name]"></div><button x-on:click="$refs.[ref name].innerText = 'bar'"></button>`

`x-ref` provides a convenient way to retrieve raw DOM elements out of your component. By setting an `x-ref` attribute on an element, you are making it available to all event handlers inside an object called `$refs`.

This is a helpful alternative to setting ids and using `document.querySelector` all over the place.

---

### `x-if`
**Example:** `<template x-if="true"><div>Some Element</div></template>`

**Structure:** `<template x-if="[expression]"><div>Some Element</div></template>`

For cases where `x-show` isn't sufficient (`x-show` sets an element to `display: none` if it's false), `x-if` can be used to  actually remove an element completely from the DOM.

It's important that `x-if` is used on a `<template></template>` tag because Alpine doesn't use a virtual DOM. This implementation allows Alpine to stay rugged and use the real DOM to work its magic.

> Note: `x-if` must have a single element root inside the `<template></template>` tag.

---

### `x-for`
**Example:**
```html
<template x-for="item in items" :key="item">
    <div x-text="item"></div>
</template>
```

`x-for` is available for cases when you want to create new DOM nodes for each item in an array. This should appear similar to `v-for` in Vue, with one exception of needing to exist on a `template` tag, and not a regular DOM element.

> Note: the `:key` binding is optional, but HIGHLY recommended.

---

### `x-transition`
**Example:**
```html
<div
    x-show="open"
    x-transition:enter="transition ease-out duration-300"
    x-transition:enter-start="opacity-0 transform scale-90"
    x-transition:enter-end="opacity-100 transform scale-100"
    x-transition:leave="transition ease-in duration-300"
    x-transition:leave-start="opacity-100 transform scale-100"
    x-transition:leave-end="opacity-0 transform scale-90"
>...</div>
```

```html
<template x-if="open">
    <div
        x-transition:enter="transition ease-out duration-300"
        x-transition:enter-start="opacity-0 transform scale-90"
        x-transition:enter-end="opacity-100 transform scale-100"
        x-transition:leave="transition ease-in duration-300"
        x-transition:leave-start="opacity-100 transform scale-100"
        x-transition:leave-end="opacity-0 transform scale-90"
    >...</div>
</template>
```

Alpine offers 6 different transition directives for applying classes to various stages of an element's transition between "hidden" and "shown" states. These directives work both with `x-show` AND `x-if`.

These behave exactly like VueJs's transition directives, except they have different, more sensible names:

| Directive | Description |
| --- | --- |
| `:enter` | Applied during the entire entering phase. |
| `:enter-start` | Added before element is inserted, removed one frame after element is inserted. |
| `:enter-end` | Added one frame after element is inserted (at the same time `enter-start` is removed), removed when transition/animation finishes.
| `:leave` | Applied during the entire leaving phase. |
| `:leave-start` | Added immediately when a leaving transition is triggered, removed after one frame. |
| `:leave-end` | Added one frame after a leaving transition is triggered (at the same time `leave-start` is removed), removed when the transition/animation finishes.

---

### `x-cloak`
**Example:** `<div x-data="{}" x-cloak></div>`

`x-cloak` attributes are removed from elements when Alpine initializes. This is useful for hiding pre-initialized DOM. It's typical to add the following global style for this to work:

```html
<style>
    [x-cloak] { display: none; }
</style>
```

### Magic Properties

---

### `$el`
**Example:**
```html
<div x-data>
    <button @click="$el.innerHTML = 'foo'">Replace me with "foo"</button>
</div>
```

`$el` is a magic property that can be used to retrieve the root component DOM node.

### `$refs`
**Example:**
```html
<span x-ref="foo"></span>

<button x-on:click="$refs.foo.innerText = 'bar'"></button>
```

`$refs` is a magic property that can be used to retrieve DOM elements marked with `x-ref` inside the component. This is useful when you need to manually manipulate DOM elements.

---

### `$event`
**Example:**
```html
<input x-on:input="alert($event.target.value)">
```

`$event` is a magic property that can be used within an event listener to retrieve the native browser "Event" object.

---

### `$dispatch`
**Example:**
```html
<div @custom-event="console.log($event.detail.foo)">
    <button @click="$dispatch('custom-event', { foo: 'bar' })">
    <!-- When clicked, will console.log "bar" -->
</div>
```

`$dispatch` is a shortcut for creating a `CustomEvent` and dispatching it using `.dispatchEvent()` internally. There are lots of good use cases for passing data around and between components using custom events. [Read here](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Creating_and_triggering_events) for more information on the underlying `CustomEvent` system in browsers.

You will notice that any data passed as the second parameter to `$dispatch('some-event', { some: 'data' })`, becomes available through the new events "detail" property: `$event.detail.some`. Attaching custom event data to the `.detail` property is standard practice for `CustomEvent`s in browsers. [Read here](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent/detail) for more info.

You can also use `$dispatch()` to trigger data updates for `x-model` bindings. For example:

```html
<div x-data="{ foo: 'bar' }">
    <span x-model="foo">
        <button @click="$dispatch('input', 'baz')">
        <!-- After the button is clicked, `x-model` will catch the bubbling "input" event, and update foo to "baz". -->
    </span>
</div>
```

---

### `$nextTick`
**Example:**
```html
<div x-data="{ fruit: 'apple' }">
    <button
        x-on:click="
            fruit = 'pear';
            $nextTick(() => { console.log($event.target.innerText) });
        "
        x-text="fruit"
    ></button>
</div>
```

`$nextTick` is a magic property that allows you to only execute a given expression AFTER Alpine has made its reactive DOM updates. This is useful for times you want to interact with the DOM state AFTER it's reflected any data updates you've made.

## License

Copyright © 2019-2020 Caleb Porzio and contributors

Licensed under the MIT license, see [LICENSE.md](LICENSE.md) for details.
