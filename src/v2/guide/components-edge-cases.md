---
title: Handling Edge Cases
type: guide
order: 106
---

> Questa pagina assume che tu abbia già letto le [Nozioni di Base sui Componenti](components.html). Leggi prima quella guida se il concetto di componente ti è nuovo.

<p class="tip">Tutte le funzionalità in questa pagina documentano la gestione dei casi limite, ovvero situazioni inusuali che a volte richiedono di piegare un pò le regole di Vue. Nota però, che tutte hanno degli svantaggi o situazioni dove potrebbero essere pericolose. Questi sono annotati in ogni caso, quindi tienili a mente quando decidi di usare ogni funzionalità.</p>

## Accesso a Componenti ed Elementi

Nella maggior parte dei casi, è meglio evitare avere a che fare con altri componenti o manipolare manualmente gli elementi del DOM. Ci sono casi, tuttavia, dove può essere appropriato.

### Accesso all'istanza di Root

In ogni sottocomponente di un'istanza `new Vue`, si può accedere a questa istanza root con la proprietà `$root`. Per esempio, in questa istanza di root:

```js
// L'istanza root di Vue
new Vue({
  data: {
    foo: 1
  },
  computed: {
    bar: function () { /* ... */ }
  },
  methods: {
    baz: function () { /* ... */ }
  }
})
```

Tutti i sottocomponenti saranno ora in grado di accedere a questa istanza e usarla come store globale:

```js
// Ottieni i dati di root
this.$root.foo

// Imposta i dati di root
this.$root.foo = 2

// Accedi alle computed properties di root
this.$root.bar

// Chiama un metodo di root
this.$root.baz()
```

<p class="tip">Questo può essere comodo per delle demo o app veramente piccole con una manciata di componenti. Tuttavia, questo pattern non scala bene con applicazione di medie o larga scala, quindi ti consigliamo fortemente di utilizzare <a href="https://github.com/vuejs/vuex">Vuex</a> per gestire lo stato nella maggior parte dei casi.</p>

### Accesso all'Istanza del Componente Padre

Simile a `$root`, la proprietà `$parent` può essere usata per accedere all'istanza del componente padre da un figlio. Può essere una tentazione utilizzare questa proprietà come alternativa pigra al passaggio di dati utilizzando una prop.

<p class="tip">Nella maggio parte dei casi, comunicare con l'istanza del padre rende l'applicazione più difficile da comprendere e rende più difficile il debugging, specialmente se vengono mutati dei dati nel padre. Quando si osserverà un componente successivamente, sarà veramente difficile capire quale mutazione è arrivata da quale parte.</p>

Ci sono casi tuttavia, in particolare componenti condivisi di librerie, dove questo _potrebbe_ essere appropriato. Per esempio, in componenti astratti che interagiscono con le API di JavaScript invece di renderizzare HTML, come questi ipotetici componenti Google Maps:

```html
<google-map>
  <google-map-markers v-bind:places="iceCreamShops"></google-map-markers>
</google-map>
```

Il componente `<google-map>` potrebbe definire una proprietà `map` alla quale tutti i sottocomponenti hanno bisogno di accedere. In questo caso `<google-map-markers>` potrebbe voler accedere a quella mappa con qualcosa del tipo `this.$parent.getMap`, per poter aggiungere un insieme di marcatori ad essa. Puoi vedere questo pattern [in uso qui](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-accessing-parent-component-instance).

Tieni a mente, tuttavia, che componenti costruiti con questo pattern sono comunque instrinsecamente fragili. Per esempio, immagina che aggiungiamo un nuovo componente `<google-map-region>` e quando `<google-map-markers>` appare al suo interno, dovrebbe solo renderizzare i marcatori che rientrano in quella regione:

```html
<google-map>
  <google-map-region v-bind:shape="cityBoundaries">
    <google-map-markers v-bind:places="iceCreamShops"></google-map-markers>
  </google-map-region>
</google-map>
```

Allora all'interno di `<google-map-markers>` potresti trovarti ad utilizzare un piccolo espediente come questo:

```js
var map = this.$parent.map || this.$parent.$parent.map
```

Il tutto è andato velocemente fuori controllo. Questo è il motivo per il quale per fornire informazioni di contesto a componenti discendenti con profondità arbitraria, raccomandiamo di usare l'[iniezione di dipendenze](#Dependency-Injection).

### Accesso all'Istanza di Componenti Figli ed Elementi Figli

Nonostante l'esistenza di prop ed eventi, a volte potresti aver bisogno di accedere direttamente a un componente figlio in JavaScript. Per farlo puoi assegnare un ID di referenza al componente figlio usando l'attributo `ref`. Per esempio:

```html
<base-input ref="usernameInput"></base-input>
```

Ora nel componente nel quale hai definito questo `ref`, puoi usare:

```js
this.$refs.usernameInput
```

per accedere all'istanza `<base-input>`. Questo ti può essere utile quando vuoi, per esempio, programmaticamente mettere il focus su questo input dal padre. In quel caso, il componente `<base-input>` può similarmente utilizzare `ref` per fornire accesso ad elementi specifici all'interno di se stesso, come:

```html
<input ref="input">
```

E anche definire metodi da utilizzare nel padre:

```js
methods: {
  // Usato per focalizzare l'input dal padre
  focus: function () {
    this.$refs.input.focus()
  }
}
```

Permettendo quindi al componente padre di focalizzare l'input dentro `<base-input>` con:

```js
this.$refs.usernameInput.focus()
```

Quando `ref` è utilizzato insieme a `v-for`, il ref ottenuto sarà un array contenente i componenti figli rispecchianti la sorgente di dati.

<p class="tip"><code>$refs</code> sono solo popolati dopo che il componente è stato renderizzato, e non sono reattivi. E' una soluzione solo pensata come scappatoia per manipolare direttamente i componenti figli - dovresti evitare di accedere ai <code>$refs</code> dall'interno di un template o di una computed property.</p>

### Iniezione di Dipendenze

Precedentemente, quando abbiamo discusso l'[Accesso all'Istanza di un Componente Padre](#Accessing-the-Parent-Component-Instance), abbiamo mostrato un esempio di questo tipo:

```html
<google-map>
  <google-map-region v-bind:shape="cityBoundaries">
    <google-map-markers v-bind:places="iceCreamShops"></google-map-markers>
  </google-map-region>
</google-map>
```

In questo componente, tutti i discendenti di `<google-map>` avevano bisogno di accedere al metodo `getMap`, per poter sapere con quale mappa dovevano interagire. Sfortunatamente, l'uso della proprietà `$parent` non scalava bene con componenti nidificati più in profondità. Questo è il caso in cui l'iniezione di dipendenze può essere utile, utilizzando due nuove opzioni dell'istanza: `provide` e `inject`.

L'opzione `provide` ci permette di specificare i dati/metodi che vogliamo **fornire** al componente discendente. Nel nostro caso, il metodo `getMap` dentro `<google-map>`:

```js
provide: function () {
  return {
    getMap: this.getMap
  }
}
```

Poi in ogni discendente, possiamo utilizzare l'opzione `inject` per ricevere specifiche proprietà che ci piacerebbe aggiungere a quell'istanza:

```js
inject: ['getMap']
```

Puoi vedere l'[esempio completo qui](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-dependency-injection). Il vantaggio rispetto ad usare `$parent` è che possiamo accedere a `getMap` in _ogni_ componente discendente, senza esporre l'intera istanza di `<google-map>`. Questo ci permette di continuare a sviluppare quel componente con più sicurezza, senza aver paura di cambiare/cancellare qualcosa sul quale un componente figlio conta. L'interfaccia tra questi componenti rimane chiaramente definita, così come con le `props`.

Infatti, puoi pensare all'iniezione di dipendenze come una sorta di "props a lungo raggio", eccetto che:

* componenti antenati non hanno bisogno di sapere quale discendente utilizza le proprietà che fornisce
* componenti discendenti non hanno bisogno di sapere da dove le proprietà iniettate provengono 

<p class="tip">Tuttavia, ci sono svantaggi nell'iniezione di dipendenze. Connette componenti nella tua applicazione al modo in cui sono già organizzati, rendendo il refactoring più difficile. Le proprietà fornite inoltre non sono reattive. Questa è una scelta di design, perchè usarle per creare una fonte centrale di dati scala male come <a href="#Accessing-the-Root-Instance">usare <code>$root</code></a> per lo stesso scopo. Se le proprietà che vuoi condividere sono specifiche della tua applicazione, invece che generiche, o se mai vorrai aggiornare dati forniti da antenati, allora questo è un buon segno che probabilmente hai bisogno di una vera soluzione di state management come <a href="https://github.com/vuejs/vuex">Vuex</a></p>

Leggi di più sull'iniezione di dipendenze all'interno della [API doc](https://vuejs.org/v2/api/#provide-inject).

## Listener di Eventi Programmatici

Fin'ora, hai visto utilizzi di `$emit`, ascoltato con `v-on`, ma l'istanza di Vue offre anche altri metodi nella sua interfaccia degli eventi. Possiamo:

- Ascoltare un evento con `$on(eventName, eventHandler)` 
- Ascoltare un evento solo una volta con `$once(eventName, eventHandler)`
- Smettere di ascoltare un evento con `$off(eventName, eventHandler)`

Normalmente non dovrai usare queste interfacce, ma sono disponibili per casi in cui hai bisogno di ascoltare manualmente gli eventi provenienti dall'istanza di un componente. Possono anche essere un potente strumento di organizzazione del codice. Per esempio, spesso puoi trovare questo pattern utilizzato per integrare librerie di terze parti:

```js
// Attacca il datepicker ad un input una volta 
// è montato nel DOM.
mounted: function () {
  // Pikaday è una libreria datepicker di terze parti 
  this.picker = new Pikaday({
    field: this.$refs.input,
    format: 'YYYY-MM-DD'
  })
},
// Poco prima che il componente è distrutto, 
// distrugge anche il datepicker, 
beforeDestroy: function () {
  this.picker.destroy()
}
```

Questo ha due problemi potenziali:

- Richiede salvare l'istanza del componente `picker`, quando è possibile che solo i lifecycle hooks hanno bisogno di accedervi. Questo non è terribile, ma può essere considerato disordine. 
- Il nostro codice di setup è tenuto separato dal codice di pulizia, rendendo programmaticamente più difficile pulire qualsiasi cosa impostiamo. 

Potresti risolvere entrambe le cose con un listener programmatico: 

```js
mounted: function () {
  var picker = new Pikaday({
    field: this.$refs.input,
    format: 'YYYY-MM-DD'
  })

  this.$once('hook:beforeDestroy', function () {
    picker.destroy()
  })
}
```

Usando questa strategia, potremmo anche usare Pickday con diversi elementi di input, con ogni nuova istanza che pulisce automaticamente se stessa:

```js
mounted: function () {
  this.attachDatepicker('startDateInput')
  this.attachDatepicker('endDateInput')
},
methods: {
  attachDatepicker: function (refName) {
    var picker = new Pikaday({
      field: this.$refs[refName],
      format: 'YYYY-MM-DD'
    })

    this.$once('hook:beforeDestroy', function () {
      picker.destroy()
    })
  }
}
```

Guarda [questo esempio](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-programmatic-event-listeners) per il codice completo. Nota, tuttavia, che se ti trovi a dover fare molti setup e pulizie all'interno di un singolo componente, la migliore soluzione probabilmente sarà creare dei componenti pià modulari. In questo caso, noi raccomenderemo creare un componente `<input-datepicker>` riutilizzabile.

Per sapere di più sui listener programmatici, leggi le API per i [Metodi su Istanze degli Eventi](https://vuejs.org/v2/api/#Instance-Methods-Events).

<p class="tip">Nota che il sistema di eventi di Vue è diverso da quello delle<a href="https://developer.mozilla.org/en-US/docs/Web/API/EventTarget">API EventTarget</a> del browser. Anche se funzionano in maniera simile, <code>$emit</code>, <code>$on</code>, e <code>$off</code> <strong>non</strong> sono aliases per <code>dispatchEvent</code>, <code>addEventListener</code>, e <code>removeEventListener</code>.</p>

## Referenze Circolari 

### Componenti Ricorsivi

Components can recursively invoke themselves in their own template. However, they can only do so with the `name` option:
I componenti possono invocare ricorsivamente se stessi nel proprio template. Tuttavia, lo possono fare solo con l'opzione `name:

``` js
name: 'unique-name-of-my-component'
```

Quando registri un componente globalmente utilizzando `Vue.component`, l'ID globale è automaticamente impostato come opzione `name`del componente.

``` js
Vue.component('unique-name-of-my-component', {
  // ...
})
```

Se non sei attento, i componenti ricorsivi possono portare a cicli infiniti:

``` js
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```

Un componente come quello sopra causerà un errore di "max stack size exceeded", quindi assicurati che l'invocazione ricorsiva è considizonale (es. usa un `v-if` che ad un certo punto diventerà `false`).

### Referenze Circolari tra Componenti

Diciamo che stai costruirendo un albero di cartelle di file, come in Finder o nell'Esplora File. Potresti avere un componente `tree-folder` con questo template:

``` html
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>
```

Poi un componente `tree-folder-contents` con questo template: 

``` html
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```

Guardando attentamente, vedrai che questi componenti sono in realtà discendenti _e_ antenati l'uno dell'altro nell'albero di render - un paradosso! Quando registri dei componenti globalmente con `Vue.component`, questo paradosso è risolto per te automaticamente. Se questo è il tuo caso, puoi smettere di leggere qui.

Tuttavia, se stai utilizzando dei componenti tramite require/import usando un __module system__, es. via Webpack o Browserify, riceverai un errore: 

```
Failed to mount component: template or render function not defined.
```

Per spiegare cosa sta accadendo, chiamiamo i nostri componenti A e B. Il sistema di moduli vede che ha bisogno di A, ma prima A ha bisogno di B, ma B ha bisogno di A, ma A ha bisogno di B, ecc. E' bloccato in un ciclo, non sapendo come trovare nessuno dei due componenti senza prima trovare l'altro. Per evitare questa situazione, abbiamo bisogno di dare al sistema di moduli un punto nel quale può dire, "A ha bisogno di B _prima o poi_, ma non c'è bisogno di trovare B per primo."

Nel nostro caso, questo punto può essere il componente `tree-folder`. Sappiamo che il figlio che crea il paradosso è il componente `tree-folder-contents`, quindi aspettiamo fino a quando il lifecycle hook `beforeCreate` lo registra:

``` js
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue').default
}
```

O alternativamente, potresti usare l'`import` asincrono di Webpack quando registri il componente localmente:

``` js
components: {
  TreeFolderContents: () => import('./tree-folder-contents.vue')
}
```

Problema risolto!

## Definizione di template alternative

### Template Inline

Quando l'attributo speciale `inline-template`è presente in un componente figlio, il componente userà il suo inner content come suo template, invece di trattarlo come contenuto. Questo permette una creazione di template più flessibile:

``` html
<my-component inline-template>
  <div>
    <p>Questi sono compilati come template propri del componente.</p>
    <p>Non come contenuto transclusivo del padre.</p>
  </div>
</my-component>
```

Il tuo template inline deve essere definito dentro l'emento del DOM al quale Vue è attaccato.

<p class="tip">Tuttavia, i <code>template inline</code> rendono lo scopo del tuo template più difficile da capire. Come best practice, raccomandiamo di definire il template dentro il componente usando l'opzione <code>template</code> o in un elemento <code>&lt;template&gt;</code> in un file <code>.vue</code>.</p>

### X-Templates

Un altro modo di definire i template è all'interno di un elemento script con il tipo `text/x-template`, e poi referenziando il template on un id. Per esempio: 

``` html
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>
```

``` js
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

Il tuo x-template ha bisogno di essere definito fuori dall'elemento del DOM al quale Vue è attaccato.

<p class="tip">Questi possono essere utili per demo con grandi template o in applicazioni estremamente piccole, altrimenti dovrebbero essere evitate, perchè separano il template dal resto della definizione del componente.</p>

## Controllo degli Aggiornamenti 

Grazie al suo sistema di reattività, Vue sa sempre quando aggiornare (se usato correttamente). Ci sono casi limite, tuttavia, dove potresti voler forzare un aggiornamento, nonostante il fatto che nessun dato reattivo è stato cambiato. Poi ci sono altri casi in cui potresti voler evitare aggiornamenti non necessari.

### Forzatura di un Aggiornamento

<p class="tip">Se ti trovi a dover forzare un aggiornamento in Vue, nel 99.99% dei casi, hai fatto un errore da qualche parte.</p>

Potresti non aver contato le particolarità del rilevamento dei cambiamenti [con gli arrays](https://vuejs.org/v2/guide/list.html#Caveats) o gli [oggetti](https://vuejs.org/v2/guide/list.html#Object-Change-Detection-Caveats), o potresti star utilizzando stati che non sono tracciati dal sistema di reattività di Vue, es. con `data`.

Tuttavia, se hai controllato le situazioni presentate qui sopra e ti trovi in questa situazione estremamente rara di dover forzare un aggiornamento, puoi farlo con [`$forceUpdate`](../api/#vm-forceUpdate).

### Componenti Statici con `v-once`

Renderizzare elementi HTML puri è molto veloce in Vue, ma a volte potresti avere un componente che contiente **tanti** contenuti statici. In questi casi, puoi assicurarti che è valutato solo una volta e poi messo in cach aggiungendo la direttiva `v-once` all'elemento radice, in questo modo:

``` js
Vue.component('terms-of-service', {
  template: `
    <div v-once>
      <h1>Terms of Service</h1>
      ... a lot of static content ...
    </div>
  `
})
```

<p class="tip">Di nuovo, prova a non abusare di questo pattern. Anche se è conveniente in quei rari casi in cui devi renderizzare tanti contenuti statici, è semplicemente non necessario a meno che tu non noti un render lento -- inoltre, potrebbe causare molta confusione in seguito. Per esempio, immagina un altro sviluppatore che non è familiare con <code>v-once</code> o semplicemente non lo nota. Potrebbe spendere ore cercando di capire perchè il template non si sta aggiornando correttamente.</p>
