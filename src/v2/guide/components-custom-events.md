---
title: Eventi Custom
type: guide
order: 103
---

> Questa pagina assume che tu abbia già letto le [Nozioni di Base sui Componenti](components.html). Leggi prima quella guida se il concetto di componente ti è nuovo.

<div class="vueschool"><a href="https://vueschool.io/lessons/communication-between-components?friend=vuejs" target="_blank" rel="sponsored noopener" title="Learn how to work with custom events on Vue School">Impara come lavorare con eventi custom in una lezione gratuita di Vue School</a></div>


## Nomi degli Eventi

Diversamente dai componenti e dalle props, i nomi degli eventi non forniscono nessuna trasformazione del carattere (da maiuscolo a minuscolo e viceversa). Infatti, il nome di un evento emesso deve corrispondere esattamente con il nome utilizzato per ascoltare quell'evento. Per esempio se emetti un evento con nome in camelCase:

```js
this.$emit('myEvent')
```

Ascoltare l'evento con il nome in kebab-case non avrà alcun effetto:

```html
<!-- Won't work -->
<my-component v-on:my-event="doSomething"></my-component>
```

Diversamente dai componenti e dalle props, i nomi degli eventi non saranno mai utilizzati come variabili o nomi di proprietà in JavaScript, quindi non c'è alcun motivo per usare il camelCase o il PascalCase. Inoltre, i listeners degli eventi `v-on` all'interno del template DOM saranno automaticamente trasformati in minuscolo (dato che l'HTML è case-insensitive), quindi `v-on:myEvent` diventerà `v-on:myevent` -- rendendo impossibile ascoltare l'evento `myEvent`.

Per questo motivo, ti consigliamo di **utilizzare sempre il kebab-case per il nome degli eventi**

## Personalizzare il `v-model` dei Componenti

> Nuovo nella versione 2.2.0+

Come scelta predefinita, `v-model` su un componente usa `value` come prop e `input` come evento, ma alcuni tipi di input come checkbox e radio buttons potrebbero voler utilizzare l'attributo `value` per uno [scopo differente](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#Value). Usando l'opzione `model` ti può evitare un conflitto in questi casi:

```js
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```

Adesso quando si usa `v-model` su questo componente:

```html
<base-checkbox v-model="lovingVue"></base-checkbox>
```

il valore di `lovingVue` sarà passato alla prop `checked`. La proprietà `lovingVue` sarà poi aggiornata quando `<base-checkbox>` emettetà un evento `change` con un valore nuovo.

<p class="tip">Nota che dovrai dichiarare lo stesso la prop <code>checked</code> nell'opzione <code>props</code> del componente.</p>

## Binding di Eventi Nativi ai Componenti

Ci potrebbero essere casi in cui tu vorresti ascoltare un evento nativo direttamente sull'elemento root di un componente. In questi casi, puoi usare il modificatore `.native` su `v-on`:

```html
<base-input v-on:focus.native="onFocus"></base-input>
```

Questo può essere utile a volte, ma non è una buona idea quando stai provando ad ascoltare un'evento su un elemento molto specifico, come un `<input>`. Per esempio il componente `<base-input>` di sopra dovrebbe subire un refactoring in modo che l'elemento root diventi un elemento `<label>`:

```html
<label>
  {{ label }}
  <input
    v-bind="$attrs"
    v-bind:value="value"
    v-on:input="$emit('input', $event.target.value)"
  >
</label>
```

In questo caso, il listener `.native` nel componente padre smetterebbe di funzionare silenziosamente. Non ci sarebbero errori, ma l'handler `onFocus` non sarebbe chiamato quando ci si aspetta che lo sia.

Per risolvere questo problema, Vue fornisce una proprietà `$listeners` contenente un oggetto di listeners utilizzati sul componente. Per esempio: 

```js
{
  focus: function (event) { /* ... */ }
  input: function (value) { /* ... */ },
}
```

Usando la proprietà `$listeners`, puoi inoltrare tutti i listener degli eventi del componente verso un elemento specifico del componente figlio con `v-on:$listeners`.  Per elementi come `<input>`, che vuoi anche che funzionino con `v-model`, è spesso utile creare una nuova computed property per i listener, come `inputListeners` qui sotto:

```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function () {
      var vm = this
      // `Object.assign` fonde gli oggetti insieme per creare un nuovo oggetto
      return Object.assign({},
        // Aggiungiamo tutti i listeners del component padre
        this.$listeners,
        // Poi possiamo aggiungere i listener custom e sovrascrivere
        // il comportamento di alcuni listener.
        {
          // Questo rende certo che il componente funzioni con v-model
          input: function (event) {
            vm.$emit('input', event.target.value)
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on="inputListeners"
      >
    </label>
  `
})
```

Ora il componente `<base-input>` è un **wrapper completamente trasparente**, ovvero può essere utilizzato come un normale elemento `<input>`: funzioneranno gli stessi a attributi e listener, senza il modificatore `.native`.

## Modificatore `.sync`

> Nuovo nella versione 2.3.0+

In alcuni casi, potremmo aver bisogno di un "binding a due vie" per una prop. Sfortunatamente, una un reale binding a due vie può creare problemi di manutenzione, perchè i componenti figli possono mutare i componenti padri senza che la sorgente della mutazione sia ovvia sia nel padre che nel figlio.

Ecco perchè al posto di un binding a due vie, noi consigliamo di emettere eventi nella forma `update:myPropName`. Per esempio, in un ipotetico componente con una prop `title`, potremmo comunicare l'intento di assegnare un nuovo valore con:

```js
this.$emit('update:title', newTitle)
```

Poi il componente padre può ascoltare quell'evento ed aggiornare la proprietà locale in data, se vuole. Per esempio:

```html
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
```

Per comodità, offriamo un'abbreviazione per questa forma con il modificatore `.sync`: 

```html
<text-document v-bind:title.sync="doc.title"></text-document>
```

<p class="tip">Nota che <code>v-bind</code> con il modificatore <code>.sync</code> <strong>non</strong> funziona con le espressioni (es. <code>v-bind:title.sync="doc.title + '!'"</code> non è valido). Invece, puoi solo specificare il nome della proprietà con la quale vuoi fare il binding, simile a come utilizzeresti <code>v-model</code>.</p>

Il modificatore `.sync` può anche essere utilizzato con `v-bind` quando utilizzi un oggetto per impostare più props allo stesso tempo:

```html
<text-document v-bind.sync="doc"></text-document>
```

Questo passa ogni proprietà nell'oggetto `doc` (es. `title`) come una prop singola, poi aggiunge il listeners di aggiornamento `v-on` per ognuna di esse. 

<p class="tip">Usare <code>v-bind.sync</code> con un oggetto esplicito, con in <code>v-bind.sync="{ title: doc.title }"</code>, non funzionerà, perchè ci sono troppi casi limiti da considerare quando si fa il parsing di un'espressione complessa come questa.</p>
