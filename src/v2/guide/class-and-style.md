---
title: Binding di Stile e Classi
type: guide
order: 6
---

Una necessità comune nel data binding è manipolare la lista di classi di un elemento e il suo stile inline. Dato che sono entrambi attributi, possiamo utilizzare `v-bind` per gestirli: abbiamo bisogno solo di calcolare una stringa finale per la nostra espressione. Tuttavia, immischiarsi con la concatenazione di stringe è fastidioso e prono ad errori. Per questo motivo, Vue fornisce una funzionalità aggiuntiva quando `v-bind` è utilizzato con `class` e `style`. Oltre alle stringhe, l'espressione può anche valutare oggetti o array.


## Binding di Classi HTML
<div class="vueschool"><a href="https://vueschool.io/lessons/vuejs-dynamic-classes?friend=vuejs" target="_blank" rel="sponsored noopener" title="Free Vue.js Dynamic Classes Lesson">Guarda una video lezione gratuita su Vue School</a></div>

### Sintassi Oggetto

Possiamo passare un oggetto a `v-bind:class` per attivare dinamicamente le classi:

``` html
<div v-bind:class="{ active: isActive }"></div>
```

La sintassi che puoi vedere sopra significa che la presenza di `active` sarà determinata dalla [veridicità](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) della proprietà `isActive` presente in data.

Puoi avere multiple classi attivate utilizzando più campi nell'oggetto. Inoltre, la direttiva `v-bind:class` può anche co-esistere con il semplice attributo `class`. Quindi, dato il seguente template:

``` html
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
```

E il seguente oggetto data:

``` js
data: {
  isActive: true,
  hasError: false
}
```

Sarà renderizzato:

``` html
<div class="static active"></div>
```

Quando `isActive` o `hasError` cambiano, la lista di classi sarà aggiornata di conseguenza. Per esempio, se `hasError` diventa `true`, la lista di classi diventerà `"static active text-danger"`.

L'oggetto collegato non deve essere per forza inline:

``` html
<div v-bind:class="classObject"></div>
```
``` js
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

Questo renderizzerà lo stesso risultato. Possiamo anche fare il binding a una [computed property](computed.html) che ritorna un oggetto. Questo è un pattern comune ed estremamente utile:

``` html
<div v-bind:class="classObject"></div>
```
``` js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

### Sintassi Array

Possiamo passare un array a `v-bind:class` per applicare la lista di classi:

``` html
<div v-bind:class="[activeClass, errorClass]"></div>
```
``` js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

Che renderizzerà:

``` html
<div class="active text-danger"></div>
```

Inoltre se tu volessi attivare un classe condizionalmente nella lista, puoi farlo con un'espressione che utilizza l'operatore ternario: 

``` html
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```

Questo applicherà sempre `errorClass`, ma applicherà `activeClass` solo quando `isActive` è un'espressione veritiera.

Tuttavia questo metodo risulta un pò verboso se utilizzi multiple classi condizionali. Questo è il motivo per il quale è anche possibile utilizzare la sintassi oggetto all'interno della sintassi array:

``` html
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

### Con i Componenti

> Questa sezione assume la cononscienza dei [Componenti Vue](components.html). Sentiti libero di saltarla e tornare qui più tardi.

Quando utilizzi l'attributo `class` su un component custom, quelle classi saranno aggiunte all'elemento radice del tuo componente. Le classi esistenti su quell'elemento non verranno sovrascritte. 

Per esempio, puoi dichiarare questo componente:

``` js
Vue.component('my-component', {
  template: '<p class="foo bar">Hi</p>'
})
```

E poi aggiungerci alcune classi quando lo usi:

``` html
<my-component class="baz boo"></my-component>
```

L'HTML renderizzato sarà:

``` html
<p class="foo bar baz boo">Hi</p>
```

La stessa cosa è valida per il binding di classi:

``` html
<my-component v-bind:class="{ active: isActive }"></my-component>
```

Quando `isActive` è veritiero, l'HTML renderizzato sarà:

``` html
<p class="foo bar active">Hi</p>
```

## Binding dello Stile Inline

### Sintassi Oggetto

La sintassi oggetto per `v-bind:style` è molto chiara - assomiglia molto al CSS, tranne per il fatto che è un oggetto JavaScript. Puoi utilizzare il camelCase o il kebab-case (utilizza le virgolette con il kebab-case) per il nome delle proprietà CSS:

``` html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
``` js
data: {
  activeColor: 'red',
  fontSize: 30
}
```

È spesso una buona idea fare il binding direttamente a un oggetto stile, in modo che il template sia più pulito:

``` html
<div v-bind:style="styleObject"></div>
```
``` js
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

Di nuovo, la sintassi oggetto è spesso usata insieme a delle computed properties che ritornano degli oggetti.

### Sintassi Array

La sintassi array per `v-bind:style` ti permette di applicare molteplici oggetti stile allo stesso elemento:

``` html
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

### Auto-prefixing

Quando utilizzi una proprietà CSS che richiede [vendor prefixes](https://developer.mozilla.org/en-US/docs/Glossary/Vendor_Prefix) in `v-bind:style`, per esempio `transform`, Vue rileverà e aggiungerà automaticamente i prefissi appropriati allo stile applicato.

### Valori Multipli

> 2.3.0+

A partire dalla versione 2.3.0+ puoi fornire un array di valori multipli (con preffiso) a una proprietà stile, per esempio:

``` html
<div v-bind:style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

Questo renderizzerà solo l'ultimo valore dell'array che è supportato dal browser. In questo esempio, renderizzerà `display: flex` nei browsers che supportano la versione senza prefissi di flexbox.
