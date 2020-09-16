---
title: Componenti Dinamici e Asincroni
type: guide
order: 105
---

> Questa pagina assume che tu abbia già letto le [Nozioni di Base sui Componenti](components.html). Leggi prima quella guida se il concetto di componente ti è nuovo.

## `keep-alive` con Componenti Dinamici

Prima, abbiamo usato l'attributo `is` per passare da un componente ad un altro in un'interfaccia con delle tab:

{% codeblock lang:html %}
<component v-bind:is="currentTabComponent"></component>
{% endcodeblock %}

Passando da un componente ad un altro però, a volte ti troverai a voler mantenere il loro stato o evitare il ri-rendering per questioni di prestazioni. Per esempio, espandendo un pò la nostra interfaccia: 

{% raw %}
<div id="dynamic-component-demo" class="demo">
  <button
    v-for="tab in tabs"
    v-bind:key="tab"
    v-bind:class="['dynamic-component-demo-tab-button', { 'dynamic-component-demo-active': currentTab === tab }]"
    v-on:click="currentTab = tab"
  >{{ tab }}</button>
  <component
    v-bind:is="currentTabComponent"
    class="dynamic-component-demo-tab"
  ></component>
</div>
<script>
Vue.component('tab-posts', {
  data: function () {
    return {
      posts: [
        {
          id: 1,
          title: 'Cat Ipsum',
          content: '<p>Dont wait for the storm to pass, dance in the rain kick up litter decide to want nothing to do with my owner today demand to be let outside at once, and expect owner to wait for me as i think about it cat cat moo moo lick ears lick paws so make meme, make cute face but lick the other cats. Kitty poochy chase imaginary bugs, but stand in front of the computer screen. Sweet beast cat dog hate mouse eat string barf pillow no baths hate everything stare at guinea pigs. My left donut is missing, as is my right loved it, hated it, loved it, hated it scoot butt on the rug cat not kitten around</p>'
        },
        {
          id: 2,
          title: 'Hipster Ipsum',
          content: '<p>Bushwick blue bottle scenester helvetica ugh, meh four loko. Put a bird on it lumbersexual franzen shabby chic, street art knausgaard trust fund shaman scenester live-edge mixtape taxidermy viral yuccie succulents. Keytar poke bicycle rights, crucifix street art neutra air plant PBR&B hoodie plaid venmo. Tilde swag art party fanny pack vinyl letterpress venmo jean shorts offal mumblecore. Vice blog gentrify mlkshk tattooed occupy snackwave, hoodie craft beer next level migas 8-bit chartreuse. Trust fund food truck drinking vinegar gochujang.</p>'
        },
        {
          id: 3,
          title: 'Cupcake Ipsum',
          content: '<p>Icing dessert soufflé lollipop chocolate bar sweet tart cake chupa chups. Soufflé marzipan jelly beans croissant toffee marzipan cupcake icing fruitcake. Muffin cake pudding soufflé wafer jelly bear claw sesame snaps marshmallow. Marzipan soufflé croissant lemon drops gingerbread sugar plum lemon drops apple pie gummies. Sweet roll donut oat cake toffee cake. Liquorice candy macaroon toffee cookie marzipan.</p>'
        }
      ],
      selectedPost: null
    }
  },
  template: '\
    <div class="dynamic-component-demo-posts-tab">\
      <ul class="dynamic-component-demo-posts-sidebar">\
        <li\
          v-for="post in posts"\
          v-bind:key="post.id"\
          v-bind:class="{ \'dynamic-component-demo-active\': post === selectedPost }"\
          v-on:click="selectedPost = post"\
        >\
          {{ post.title }}\
        </li>\
      </ul>\
      <div class="dynamic-component-demo-post-container">\
        <div \
          v-if="selectedPost"\
          class="dynamic-component-demo-post"\
        >\
          <h3>{{ selectedPost.title }}</h3>\
          <div v-html="selectedPost.content"></div>\
        </div>\
        <strong v-else>\
          Click on a blog title to the left to view it.\
        </strong>\
      </div>\
    </div>\
  '
})
Vue.component('tab-archive', {
  template: '<div>Archive component</div>'
})
new Vue({
  el: '#dynamic-component-demo',
  data: {
    currentTab: 'Posts',
    tabs: ['Posts', 'Archive']
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
.dynamic-component-demo-tab-button.dynamic-component-demo-active {
  background: #e0e0e0;
}
.dynamic-component-demo-tab {
  border: 1px solid #ccc;
  padding: 10px;
}
.dynamic-component-demo-posts-tab {
  display: flex;
}
.dynamic-component-demo-posts-sidebar {
  max-width: 40vw;
  margin: 0 !important;
  padding: 0 10px 0 0 !important;
  list-style-type: none;
  border-right: 1px solid #ccc;
}
.dynamic-component-demo-posts-sidebar li {
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
  cursor: pointer;
}
.dynamic-component-demo-posts-sidebar li:hover {
  background: #eee;
}
.dynamic-component-demo-posts-sidebar li.dynamic-component-demo-active {
  background: lightblue;
}
.dynamic-component-demo-post-container {
  padding-left: 10px;
}
.dynamic-component-demo-post > :first-child {
  margin-top: 0 !important;
  padding-top: 0 !important;
}
</style>
{% endraw %}

Noterai che se selezioni un post, passi alla tab _Archive_, e poi torni alla tab _Posts_, il post che avevi selezionato non è più visibile. Questo perchè ogni volta che passi a una nuova tab, Vue crea una nuova istanza di `currentTabComponent`.

Ricreare componenti dinamici solutamente è utile, ma in questo caso, ci piacerebbe che le istanze dei componenti tab siano messe in cache quando vengono create per la prima volta. Per rispondere a questa esigenza, possiamo racchiudere il nostro componente dinamico in un elemento `keep-alive`:

``` html
<!-- I componenti inattivi saranno messi in cache! -->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

Controlla il risultato qui sotto:

{% raw %}
<div id="dynamic-component-keep-alive-demo" class="demo">
  <button
    v-for="tab in tabs"
    v-bind:key="tab"
    v-bind:class="['dynamic-component-demo-tab-button', { 'dynamic-component-demo-active': currentTab === tab }]"
    v-on:click="currentTab = tab"
  >{{ tab }}</button>
  <keep-alive>
    <component
      v-bind:is="currentTabComponent"
      class="dynamic-component-demo-tab"
    ></component>
  </keep-alive>
</div>
<script>
new Vue({
  el: '#dynamic-component-keep-alive-demo',
  data: {
    currentTab: 'Posts',
    tabs: ['Posts', 'Archive']
  },
  computed: {
    currentTabComponent: function () {
      return 'tab-' + this.currentTab.toLowerCase()
    }
  }
})
</script>
{% endraw %}

Ora la tab _Posts_ mantiene il suo stato (il post selezionato) anche quando non è renderizzato. Guarda [questo esempio](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-keep-alive-with-dynamic-components) per il codice completo.

<p class="tip">Nota che `<keep-alive>` richiede che i tutti i componenti tra i quali ci si sposta abbiano un nome, o utilizzando l'opzione `name` sul componente, oppure attraverso la registrazione globale o locale.</p>

Per maggiori dettagli controlla `<keep-alive>` nella [API reference](../api/#keep-alive).

## Componenti Asincroni

<div class="vueschool"><a href="https://vueschool.io/lessons/dynamically-load-components?friend=vuejs" target="_blank" rel="sponsored noopener" title="Free Vue.js Async Components lesson">Guarda una video lezione gratuita su Vue School</a></div>

In applicazioni grandi, potremmo aver bisogno di dividere l'applicazione in parti più piccole e caricare un componente dal server solo quando è necessario. Per rendere questa procedura più semplice, Vue ti permette di definire il tuo componente come una funzione factory che in modo asincrono determina la definizione del tuo componente. Vue attiverà la funzione factory solo quando il componente deve essere renderizzato e metterà il risultato in cache per i futuri ri-renders. Per esempio: 

``` js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // Passa la definizione del componente per risolvere la callback
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

Come puoi vedere, la funzione factory riceve una callback `resolve`, la quale dovrebbe essere chiamata quando hai recuperato la definizione del tuo componente dal server. Puoi anche chiamare `reject(reason)` per indicare che il caricamento è fallito. Il `setTimeout` è presente per dimostrazione; come recuperare il componente è a tua discrezione. Un approccio raccomandato è usare i componenti asincroni insieme alla [funzionalità di code-splitting di Webpack](https://webpack.js.org/guides/code-splitting/):

``` js
Vue.component('async-webpack-example', function (resolve) {
  // Questa sintassi speciale di require indicherà a Webpack di
  // spezzare automaticamente il tuo codice in pacchetti
  // che sono caricati con richieste Ajax.
  require(['./my-async-component'], resolve)
})
```

Puoi anche ritornare una `Promise` nella funziona factory, così che con Webpack 2 e la sintassi ES2015 puoi utilizzare gli import dinamici:

``` js
Vue.component(
  'async-webpack-example',
  // Un import dinamico ritorna una Promise.
  () => import('./my-async-component')
)
```

Quando usi la [registrazione locale](components-registration.html#Local-Registration), puoi anche fornire direttamente una funzione che ritorna una `Promise`:

``` js
new Vue({
  // ...
  components: {
    'my-component': () => import('./my-async-component')
  }
})
```

<p class="tip">Se sei un utilizzatore di <strong>Browserify</strong> che vorrebbe utilizzare i componenti asincroni, il suo creatore ha sfortunatamente [messo in chiaro](https://github.com/substack/node-browserify/issues/58#issuecomment-21978224) che il caricamento asincrono "non è qualcosa che verrà mai supportato da Browserify." Ufficialmente, per lo meno. La community di Browserify ha trovato [acluni espedienti](https://github.com/vuejs/vuejs.org/issues/620), i quali potrebbero essere utili per applicazioni complesse e già esistenti. Per tutti gli altri scenari, raccomandiamo di utilizzare la soluzione di supporto asincrono già inclusa in Webpack.</p>

### Gestire lo Stato di Caricamento

> Nuovo nella versione 2.3.0+

La factory di componenti asincroni può anche ritornare un oggetto nel seguente formato:

``` js
const AsyncComponent = () => ({
  // Il componente da caricare (dovrebbe essere una Promise)
  component: import('./MyComponent.vue'),
  // Un componente da usare mentre il componente asincrono sta caricando
  loading: LoadingComponent,
  // Un componente da usare se il caricamento fallisce
  error: ErrorComponent,
  // Ritardo prima di mostrare il componente in caricamento. Predefinito: 200ms.
  delay: 200,
  // Il componente di errore verrà mostrato se è stato fornito e 
  // superato un timeout. Predefinito: Infinito.
  timeout: 3000
})
```

> Nota che hai bisogno di utilizzare [Vue Router](https://github.com/vuejs/vue-router) 2.4.0+ se vuoi utilizzare la sintassi mostrata sopra per componenti collegati a delle route.
