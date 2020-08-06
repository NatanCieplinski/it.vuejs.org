---
title: Nozioni di Base sui Componenti
type: guide
order: 11
---

<div class="vueschool"><a href="https://vueschool.io/courses/vuejs-components-fundamentals?friend=vuejs" target="_blank" rel="sponsored noopener" title="Free Vue.js Components Fundamentals Course">Guarda un video gratuito su Vue School</a></div>

## Base Example

Questo è un esempio di un componente Vue:

``` js
// Define a new component called button-counter
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```

I componenti sono istanze riutilizzabili di Vue con un nome: in questo caso, `<button-counter>`. Possiamo usare questo componente come elemento custom all'interno dell'istanza root di Vue creata con `new Vue`:

```html
<div id="components-demo">
  <button-counter></button-counter>
</div>
```

{% codeblock lang:js %}
new Vue({ el: '#components-demo' })
{% endcodeblock %}

{% raw %}
<div id="components-demo" class="demo">
  <button-counter></button-counter>
</div>
<script>
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count += 1">You clicked me {{ count }} times.</button>'
})
new Vue({ el: '#components-demo' })
</script>
{% endraw %}

Dato che i componenti sono istanze riutilizzabili di Vue, accettano le stesse opzioni accettate da `new Vue`, come `data`, `computed`, `watch`, `methods` e i lifecycle hooks. Le uniche eccezioni sono alcune opzioni specifiche per la root come `el`.

## Riutilizzare i Componenti

I componenti possono essere riutilizzati quante volte vuoi:

```html
<div id="components-demo">
  <button-counter></button-counter>
  <button-counter></button-counter>
  <button-counter></button-counter>
</div>
```

{% raw %}
<div id="components-demo2" class="demo">
  <button-counter></button-counter>
  <button-counter></button-counter>
  <button-counter></button-counter>
</div>
<script>
new Vue({ el: '#components-demo2' })
</script>
{% endraw %}

Nota che quando clicchi il pulsante, ognuno conserva il proprio `count` separatamente. Questo avviene perchè ogni volta che usi un componete, una sua nuova **istanza** viene creata.

### `data` Deve Essere una Funzione

Quando abbiamo definito il componente `<button-counter>`, avrai notato che `data` non era fornito come un oggetto, in questo modo:

```js
data: {
  count: 0
}
```

Invece, **l'opzione `data` di un componente deve essere una funzione**, in modo che ogni istanza possa mantenere una copia indipendente che ritorna un oggetto:

```js
data: function () {
  return {
    count: 0
  }
}
```

Se Vue non avesse avuto questa regola, cliccare su un pulsante avrebbe camiato i dati di _tutte le altre istanza_, come puoi vedere sotto:

{% raw %}
<div id="components-demo3" class="demo">
  <button-counter2></button-counter2>
  <button-counter2></button-counter2>
  <button-counter2></button-counter2>
</div>
<script>
var buttonCounter2Data = {
  count: 0
}
Vue.component('button-counter2', {
  data: function () {
    return buttonCounter2Data
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
new Vue({ el: '#components-demo3' })
</script>
{% endraw %}

## Organizzazione dei Componenti

È comune per un'applicazione essere organizzata in un albero di componenti annidati:

![Component Tree](/images/components.png)

Per esempio, potresti avere un componente per l'header, sidebar, e l'area dei contenuti, ed ognuno tipicamente contiene altri componenti come link per la navigazione, blog post, ecc.

Per usare questi componenti nel template, devi registrarli in modo che Vue sappia della loro esistenza. Ci sono due tipi di registrazione di un componente: **globale** e **locale**. Fin'ora, abbiamo registrato i componenti solo globalmente, utilizzando `Vue.component`:

```js
Vue.component('my-component-name', {
  // ... options ...
})
```

I componenti registrati globalmente possono essere usati nel template di qualsiasi istanza root di Vue (`new Vue`) creata successivamente -- e anche dentro tutti i sotto componenti del suo albero di componenti.

Questo è tutto quello che hai bisogno di sapere sulla registrazioni dei componenti per ora, ma una volta che avrai finito di leggere questa pagina e ti sentirai a tuo agio con il suo contenuto, ti consigliamo di tornare indietro per leggere la guida completa sulla [Registrazione dei Componenti](components-registration.html).

## Passare Dati ai Componenti Figli con le Props

Prima, avevamo accennato la creazione di un componente per un blog post. Il problema è, che il componente non sarà utile a meno che tu non gli possa passare dei dati, come un titolo e il contenuto del particolare post che vogliamo mostrare. Quì è dove le props entrano in gioco.

Le props sono attributi custom che puoi registrare in un componente. Quando un valore è passato all'attributo di una prop, questo diventa una proprietà su quell'istanza del componente. Per passare un titolo al nostro componente per i blog post, possiamo includerlo nella lista di props che il componente accetta, usando l'opzione `props`:

```js
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})
```

Un componente può avere tante props quante ne vuoi e per impostazione predefinita, qualsiasi valore può essere passato a qualsiasi prop. Nel template qui sopra, vedrai che possiamo accedere a questo valore nell'istanza del componete, esattamente come con `data`.

Una volta che una prop è registrata, puoi passargli dati come attributi custom, in questo modo:

```html
<blog-post title="My journey with Vue"></blog-post>
<blog-post title="Blogging with Vue"></blog-post>
<blog-post title="Why Vue is so fun"></blog-post>
```

{% raw %}
<div id="blog-post-demo" class="demo">
  <blog-post1 title="My journey with Vue"></blog-post1>
  <blog-post1 title="Blogging with Vue"></blog-post1>
  <blog-post1 title="Why Vue is so fun"></blog-post1>
</div>
<script>
Vue.component('blog-post1', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})
new Vue({ el: '#blog-post-demo' })
</script>
{% endraw %}

Tuttavia in una tipica app è molto probabile che tu abbia un array di post in `data`:

```js
new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})
```

E poi tu voglia renderizzare un componente per ogni post:

```html
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>
```

Sopra puoi vedere che possiamo utilizzare `v-bind` per passare dinamicamente delle props. Questo è specialmente utile quando non conosci in anticipo l'esatto contenuto di ciò sarà renderizzato, come nel caso in cui esegui il [fetch dei post da un'API](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-component-blog-post-example).

Questo è tutto quello che hai bisogno di sapere sulle props per ora, ma una volta che avrai finito di leggere questa pagina e ti sentirai a tuo agio con il suo contenuto, ti consigliamo di tornare indietro per leggere la guida completa alle [Props](components-props.html).

## Un Solo Elemento di Root

Quando creerai il tuo componente `<blog-post>`, il tuo template eventualmente conterrà più del solo titolo:

```html
<h3>{{ title }}</h3>
```

Come minimo, vorrai includere il contenuto del post:

```html
<h3>{{ title }}</h3>
<div v-html="content"></div>
```

Tuttavia se tu provi questo nel tuo template, Vue ti mostrerà un errore, che ti spiega che **ogni componente deve avere un solo elemento di root**. Puoi evitare questo errore racchiudendo il tuo template in un elemento padre, in questo modo:

```html
<div class="blog-post">
  <h3>{{ title }}</h3>
  <div v-html="content"></div>
</div>
```

Mentre il nostro componente cresce, è probabile che avremo bisogno di altro oltre al contenuto e al titolo, come una data di pubblicazione, commenti, ed altro. Definire una prop per ogni pezzo di informazione collegato può diventare fastidioso:

```html
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
  v-bind:content="post.content"
  v-bind:publishedAt="post.publishedAt"
  v-bind:comments="post.comments"
></blog-post>
```

Questo quindi può essere un buon momento per fare refactoring sul componente `<blog-post>` per accettare invece una singola prop `post`:

```html
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
></blog-post>
```

```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```

<p class="tip">La funzionalità mostrata qui sopra ed altre mostrate in futuro fanno uso dei [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) di JavaScript per rendere i template multi linea più leggibili. Questi non sono supportati da Internet Explorer (IE), quindi se tu devi supportare Internet Explorer e non stai facendo il transpiling (es. con Babel o TypeScript), utilizza invece i [newline escapes](https://css-tricks.com/snippets/javascript/multiline-string-variables-in-javascript/)</p>

Adesso, ogni volta che una proprietà è aggiunta all'oggetto `post`, questa sarà automaticamente disponibile all'interno di `<blog-post>`.

## Ascoltare Eventi dai Componenti Figli

Mentre sviluppiamo il nostro componente `<blog-post>`, alcune funzionalità potrebbero richiedere la necessità di comunicare con il componente padre. Per esempio, potremmo decidere di includere una funzionalità di accessibilità per ingrandire il testo del nostro blog post, mentre il resto della pagina deve rimanere della dimensione predefinita:

Nel componente padre, possiamo supportare questa funzionalità aggiungendo `postFontSize` come proprietà in data:

```js
new Vue({
  el: '#blog-posts-events-demo',
  data: {
    posts: [/* ... */],
    postFontSize: 1
  }
})
```

Che può essere usata nel template per controllare la dimensione di tutti i blog post:

```html
<div id="blog-posts-events-demo">
  <div :style="{ fontSize: postFontSize + 'em' }">
    <blog-post
      v-for="post in posts"
      v-bind:key="post.id"
      v-bind:post="post"
    ></blog-post>
  </div>
</div>
```

Adesso aggiungiamo un pulsante per ingrandire il testo prima del contenuto di ogni post:

```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <button>
        Enlarge text
      </button>
      <div v-html="post.content"></div>
    </div>
  `
})
```

Il problema è che questo pulsante non fa nulla:

```html
<button>
  Enlarge text
</button>
```

Quando clicchiamo il pulsante, abbiamo bisogno di comunicare al componente padre che esso dovrebbe ingrandire il testo di tutti i post. Fortunatamente, l'istanza di Vue fornisce un sistema di eventi custom per risolvere questo problema. Il componente padre può scegliere di ascoltare qualsiasi evento del componente figlio con `v-on:`, così come avrebbe fatto con un evento DOM nativo:

```html
<blog-post
  ...
  v-on:enlarge-text="postFontSize += 0.1"
></blog-post>
```

Il componente figlio può successivamente emettere un evento su sè stesso chiamando il [metodo **`$emit`**](../api/#vm-emit), passandogli il nome dell'evento:

```html
<button v-on:click="$emit('enlarge-text')">
  Enlarge text
</button>
```

Grazie al listener `v-on:enlarge-text="postFontSize += 0.1"`, il componente padre riceverà l'evento e l'aggiornamento del valore `postFontSize`.

{% raw %}
<div id="blog-posts-events-demo" class="demo">
  <div :style="{ fontSize: postFontSize + 'em' }">
    <blog-post
      v-for="post in posts"
      v-bind:key="post.id"
      v-bind:post="post"
      v-on:enlarge-text="postFontSize += 0.1"
    ></blog-post>
  </div>
</div>
<script>
Vue.component('blog-post', {
  props: ['post'],
  template: '\
    <div class="blog-post">\
      <h3>{{ post.title }}</h3>\
      <button v-on:click="$emit(\'enlarge-text\')">\
        Enlarge text\
      </button>\
      <div v-html="post.content"></div>\
    </div>\
  '
})
new Vue({
  el: '#blog-posts-events-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue', content: '...content...' },
      { id: 2, title: 'Blogging with Vue', content: '...content...' },
      { id: 3, title: 'Why Vue is so fun', content: '...content...' }
    ],
    postFontSize: 1
  }
})
</script>
{% endraw %}

### Emettere un Valore con un Evento

A volte è utile emettere uno specifico valore con un evento. Per esempio, potremmo volere che il componente `<blog-post>` si occupi di quanto il testo vada ingrandito. In questi casi, possiamo usare il secondo parametro di `$emit` per inviare il valore:

```html
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```

Poi quando ascoltiamo l'evento nel componente padre, possiamo accedere al valore emessi dall'evento con `$event`:

```html
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```

Oppure, se l'handler dell'evento è un metodo:

```html
<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"
></blog-post>
```

Allora il valore sarà passato come primo parametro del metodo:

```js
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```

### Usare `v-model` sui Componenti

Gli eventi custom possono anche essere utilizzati per creare input custom che funzionano utilizzando `v-model`. Ricorda che:

```html
<input v-model="searchText">
```

fa la stessa cosa di:

```html
<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
```

Quando è usato su un componete, `v-model` invece fa questo:

``` html
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>
```

Internamente, perchè questo funzioni, l'`<input>` dentro il componente deve:

- Fare il binding dell'attributo `value` a una prop `value`
- In caso di `input`, emettere il proprio evento custom `input` con il nuovo valore

Qui puoi vedere tutto questo in azione:

```js
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
```

Ora `v-model` dovrebbe funzionare perfettamente con questo componente:

```html
<custom-input v-model="searchText"></custom-input>
```

Questo è tutto quello che hai bisogno di sapere sugli eventi custom dei componenti per ora, ma una volta che avrai finito di leggere questa pagina e ti sentirai a tuo agio con il suo contenuto, ti consigliamo di tornare indietro per leggere la guida completa agli [Eventi Custom](components-custom-events.html).

## Distribuzione del Contenuto con gli Slot

Così come con un elemento HTML, è spesso utile avere la capacità di passare del contenuto a un componente, in questo modo:

``` html
<alert-box>
  Something bad happened.
</alert-box>
```

Che potrebbe renderizzare qualcosa simile a:

{% raw %}
<div id="slots-demo" class="demo">
  <alert-box>
    Something bad happened.
  </alert-box>
</div>
<script>
Vue.component('alert-box', {
  template: '\
    <div class="demo-alert-box">\
      <strong>Error!</strong>\
      <slot></slot>\
    </div>\
  '
})
new Vue({ el: '#slots-demo' })
</script>
<style>
.demo-alert-box {
  padding: 10px 20px;
  background: #f3beb8;
  border: 1px solid #f09898;
}
</style>
{% endraw %}

Fortunatamente, questo compito è reso molto semplice dall'elemento custom di Vue `<slot>`:

```js
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```

Come puoi vedere sopra, abbiamo solo aggiunto lo slot dove vogliamo che questo vada -- tutto qui. Abbiamo finito!

Questo è tutto quello che hai bisogno di sapere sugli slot per ora, ma una volta che avrai finito di leggere questa pagina e ti sentirai a tuo agio con il suo contenuto, ti consigliamo di tornare indietro per leggere la guida completa agli [Slot](components-slots.html).

## Componenti Dinamici

A volte, è utile passare dinamicamente da un componente all'altro, come nel caso di un'interfaccia con delle tab:

{% raw %}
<div id="dynamic-component-demo" class="demo">
  <button
    v-for="tab in tabs"
    v-bind:key="tab"
    class="dynamic-component-demo-tab-button"
    v-bind:class="{ 'dynamic-component-demo-tab-button-active': tab === currentTab }"
    v-on:click="currentTab = tab"
  >
    {{ tab }}
  </button>
  <component
    v-bind:is="currentTabComponent"
    class="dynamic-component-demo-tab"
  ></component>
</div>
<script>
Vue.component('tab-home', { template: '<div>Home component</div>' })
Vue.component('tab-posts', { template: '<div>Posts component</div>' })
Vue.component('tab-archive', { template: '<div>Archive component</div>' })
new Vue({
  el: '#dynamic-component-demo',
  data: {
    currentTab: 'Home',
    tabs: ['Home', 'Posts', 'Archive']
  },
  computed: {
    currentTabComponent: function () {
      return 'tab-' + this.currentTab.toLowerCase()
    }
  }
})
</script>
<style>
.dynamic-component-demo-tab-button {
  padding: 6px 10px;
  border-top-left-radius: 3px;
  border-top-right-radius: 3px;
  border: 1px solid #ccc;
  cursor: pointer;
  background: #f0f0f0;
  margin-bottom: -1px;
  margin-right: -1px;
}
.dynamic-component-demo-tab-button:hover {
  background: #e0e0e0;
}
.dynamic-component-demo-tab-button-active {
  background: #e0e0e0;
}
.dynamic-component-demo-tab {
  border: 1px solid #ccc;
  padding: 10px;
}
</style>
{% endraw %}

L'esempio di sopra è reso possibile dall'elemento `<component>` di Vue con l'attributo speciale `is`:

```html
<!-- Component changes when currentTabComponent changes -->
<component v-bind:is="currentTabComponent"></component>
```

Nell'esempio di sopra, `currentTabComponent` può contenere sia:

- il nome di un componente registrato, oppure 
- l'oggetto options di un componente

Guarda [questo esempio](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-dynamic-components) per sperimentare con il codice completo, o [questa versione](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-dynamic-components-with-binding) per un esempio di binding all'oggetto options di un componente, invece del suo nome registrato.

Tieni a mente che questo attributo può essere utilizzato con un elemento standard HTML,che tuttavia sarà trattato come un componente, il che vuol dire che tutti gli attributi **saranno convertiti ad attributi DOM**. Per qualche proprietà come `value`, per farla funzionare sarà necessario fare il binding utilizzando il [modificatore `.prop`](../api/#v-bind).

Questo è tutto ciò che hai bisogno di sapere sui componenti dinamici per ora, ma una volta che avrai finito di leggere questa pagina e ti sentirai a tuo agio con il suo contenuto, ti consigliamo di tornare indietro per leggere la guida completa ai [Componenti Dinamici e Asincroni](components-dynamic-async.html).

## Particolarità del Parsing del Template DOM

Alcuni elementi HTML, come `<ul>`, `<ol>`, `<table>` e `<select>` hanno delle restrizioni su quali elementi possono contenere, e alcuni elementi come `<li>`, `<tr>`, e `<option>` possono apparire solo all'interno di alcuni elementi specifici.

Questo può portare a problemi quando si usano i componenti con elementi che hanno tali restrizioni. Per esempio:

``` html
<table>
  <blog-post-row></blog-post-row>
</table>
```

Il componente custom `<blog-post-row>` sarà recepito come contenuto invalido, causando un errore nell'eventuale output renderizzato. Fortunatamente, l'attributo speciale `is` offre un espediente: 

``` html
<table>
  <tr is="blog-post-row"></tr>
</table>
```

È importante notare che **questa limitazion _non_ si applica se stai utilizzando uno string template da una delle seguenti sorgenti**:

- String templates (es. `template: '...'`)
- [Componenti in file singolo (`.vue`)](single-file-components.html)
- [`<script type="text/x-template">`](components-edge-cases.html#X-Templates)

Questo è tutto quello che hai bisogno di sapere sulle particolarità del parsing del DOM per ora -- ed effettivamente, questa è la fine dei Vue _Essentials_. Congratulazioni! C'è ancora altro da imparare, ma prima, ti consigliamo di prenderti una pausa per giocare tu stesso con Vue e provare a costruire qualcosa di divertente.

Una volta che ti sentirai a tuo agio con le conoscenze che hai appena digerito, ti consigliamo di tornare indietro per leggere la guida completa ai [Componenti Dinamici e Asincroni](components-dynamic-async.html), oltre alle altre pagine nella sezione della sidebar Componenti approfonditi.
