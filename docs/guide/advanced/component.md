# Component Interpolation

## Basic Usage

Sometimes, we need to localize text that includes an HTML tag or a component. For example:

```html
<p>I accept xxx <a href="/term">Terms of Service Agreement</a></p>
```

In the above message, if you use `$t`, you might try to compose the following locale messages:

```js
const messages = {
  en: {
    term1: 'I Accept xxx\'s',
    term2: 'Terms of Service Agreement'
  }
}
```

And your localized template may look like this:

```html
<p>{{ $t('term1') }}<a href="/term">{{ $t('term2') }}</a></p>
```

Output:

```html
<p>I accept xxx <a href="/term">Terms of Service Agreement</a></p>
```

Manually splitting translations like this can be cumbersome. Moreover, including the `<a>` tag directly in a locale message and rendering it with `v-html="$t('term')"` poses a potential XSS security risk.

To avoid this issue, you can use the Translation component (`i18n-t`). For example:

### Template:

```html
<div id="app">
  <!-- ... -->
  <i18n-t keypath="term" tag="label" for="tos">
    <a :href="url" target="_blank">{{ $t('tos') }}</a>
  </i18n-t>
  <!-- ... -->
</div>
```

### JavaScript:

```js
import { createApp } from 'vue'
import { createI18n } from 'vue-i18n'

const i18n = createI18n({
  locale: 'en',
  messages: {
    en: {
      tos: 'Term of Service',
      term: 'I accept xxx {0}.'
    },
    ja: {
      tos: '利用規約',
      term: '私は xxx の{0}に同意します。'
    }
  }
})

const app = createApp({
  data: () => ({ url: '/term' })
})

app.use(i18n)
app.mount('#app')
```

### Output:

```html
<div id="app">
  <!-- ... -->
  <label for="tos">
    I accept xxx <a href="/term" target="_blank">Term of Service</a>.
  </label>
  <!-- ... -->
</div>
```

For more details, see the [example](https://github.com/intlify/vue-i18n/blob/master/examples/legacy/components/translation.html).

The child elements of the Translation component are interpolated within the locale message defined by the `keypath` prop. In the above example:

:::v-pre
`<a :href="url" target="_blank">{{ $t('tos') }}</a>`
:::

is interpolated with the `term` locale message.

The component interpolation follows **list interpolation**, meaning the child elements are interpolated based on their order of appearance.

You can choose the root node element type by specifying a `tag` prop. If omitted, it defaults to [Fragments](https://v3-migration.vuejs.org/new/fragments.html).

## Slots Syntax Usage

Using named slots syntax is often more convenient. For example:

### Template:

```html
<div id="app">
  <!-- ... -->
  <i18n-t keypath="info" tag="p">
    <template v-slot:limit>
      <span>{{ changeLimit }}</span>
    </template>
    <template v-slot:action>
      <a :href="changeUrl">{{ $t('change') }}</a>
    </template>
  </i18n-t>
  <!-- ... -->
</div>
```

### JavaScript:

```js
import { createApp } from 'vue'
import { createI18n } from 'vue-i18n'

const i18n = createI18n({
  locale: 'en',
  messages: {
    en: {
      info: 'You can {action} until {limit} minutes from departure.',
      change: 'change your flight',
      refund: 'refund the ticket'
    }
  }
})

const app = createApp({
  data: () => ({
    changeUrl: '/change',
    refundUrl: '/refund',
    changeLimit: 15,
    refundLimit: 30
  })
})

app.use(i18n)
app.mount('#app')
```

### Output:

```html
<div id="app">
  <!-- ... -->
  <p>
    You can <a href="/change">change your flight</a> until <span>15</span> minutes from departure.
  </p>
  <!-- ... -->
</div>
```

Alternatively, you can use the shorthand syntax for named slots:

```html
<div id="app">
  <!-- ... -->
  <i18n-t keypath="info" tag="p">
    <template #limit>
      <span>{{ changeLimit }}</span>
    </template>
    <template #action>
      <a :href="changeUrl">{{ $t('change') }}</a>
    </template>
  </i18n-t>
  <!-- ... -->
</div>
```

:::warning LIMITATION
:warning: In the Translation component, slot props are not supported.
:::

## Pluralization Usage

To handle pluralization in component interpolation, use the `plural` prop. For example:

### Template:

```html
<div id="app">
  <!-- ... -->
  <i18n-t keypath="message.plural" locale="en" :plural="count">
    <template #n>
      <b>{{ count }}</b>
    </template>
  </i18n-t>
  <!-- ... -->
</div>
```

### JavaScript:

```js
const { createApp, ref } = Vue
const { createI18n } = VueI18n

const i18n = createI18n({
  legacy: false,
  locale: 'en',
  messages: {
    en: {
      message: {
        plural: 'no bananas | {n} banana | {n} bananas'
      }
    }
  }
})

const app = createApp({
  setup() {
    const count = ref(2)
    return { count }
  }
})
app.use(i18n)
app.mount('#app')
```

### Output:

```html
<div id="app" data-v-app="">
  <!-- ... -->
  <b>2</b> bananas
  <!-- ... -->
</div>
```

## Scope Resolving

The [Scope](../essentials/scope.md) resolving of the Translation component defaults to `parent`.

This means that the Translation component uses the scope enabled in the parent component. If the parent component calls `useI18n` with `useScope: 'global'`, it will use the Global Scope; otherwise, it will use the Local Scope of the parent component.

However, you may sometimes see the following warning message in your browser console:

```
[intlify] Not found parent scope. use the global scope.
```

This warning appears when the parent component does not call `useI18n`, meaning there is no defined local scope.

To suppress this warning, you can explicitly set `scope="global"` on the Translation component:

```html
<i18n-t keypath="message.foo" scope="global">
  ...
</i18n-t>
```

:::tip NOTE
This warning is not displayed in production builds.
:::

