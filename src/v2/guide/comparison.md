---
title: Confronto con Altri Framework
type: guide
order: 801
---

Questa è decisamente la pagina più difficile da scrivere in tutta la guida, ma noi crediamo sia importante. Probabilmente tu hai avuto problematiche che hai provato a risolvere utilizzando un'altra libreria. Ti trovi qui perchè vuoi sapere se Vue può risolvere la tua specifica problematica in modo migliore. Questa è la domanda alla quale speriamo di darti una risposta. 

Inoltre proveremo con il massimo impegno ad essere imparziali. Come core team, ovviamente Vue ci piace molto. Ci sono alcune problematiche che pensiamo Vue possa risolvere meglio delle alternative. Se non lo credessimo, non ci lavoreremmo. Vogliamo però essere giusti e accurati. Dove altre librerie offrono vantaggi significativi, come l'ecosistema vasto di renderers per React o il supporto a IE6 offerto da Knockout, proveremo ad elencarli. 

Ci piacerebbe anche il **tuo** aiuto per mantenere questo documento aggiornato, perchè il mondo di JavaScript si muove velocemente! Se noti un'inaccuratezza o qualcosa che non ti sembra corretto, per favore faccelo sapere [aprendo un issue](https://github.com/vuejs/vuejs.org/issues/new?title=Inaccuracy+in+comparisons+guide).

## React

React e Vue hanno molti aspetti in comune. Entrambi: 

- utilizzano un DOM virtuale
- forniscono una view di componenti reattiva e componibile
- si concentrano sul core della libreria, lasciando la gestione di aspetti come routing e state management a librerie affiliate

Essendo così simili nell'approccio, abbiamo dedicato più tempo a dettagliare questo confronto rispetto a tutti gli altri. Vogliamo assicurarci non solo che ci sia accuratezza tecnica, ma anche equilibrio. Evidenziamo dove React brilla più di Vue, per esempio nella ricchezza dell'ecosistema e l'abbondanza dei renderers custom.

Detto questo, è inevitabile che a qualche utilizzatore di React il confronto apparirà a favore di Vue, dato che molti degli aspetti esplorati sono parzialmente soggettivi. Riconosciamo l'esistenza di diverse preferenze riguardanti i tecnicismi, e questo confronto ha come scopo primario sottolineare le ragioni per le quali Vue può essere un'alternativa migliore se le tue preferenze coincidono con le nostre. 

Inoltre alcune di queste sezioni inoltre potrebbero essere leggermente antiquate dato i recenti aggiornamenti in React 16+, e stiamo pianificando di lavorare con la community di React per rinnovare questa sezione in un prossimo futuro.

### Prestazioni a Runtime

Sia React che Vue eccezzionalmente e similarmente veloci, quindi la velocità è poco probabile che sia un fattore decisivo nella scelta tra i due. Per delle metriche specifiche, puoi controllare questo [benchmark di terze parti](https://stefankrause.net/js-frameworks-benchmark8/table.html), che si concentra sulle prestazioni brute nel render/aggiornamento di un albero di componenti molto semplice. 

#### Sforzo Richiesto per l'Ottimizzazione

In React, quando lo state di un componente cambia, si attiva un re-render di tutto il sottoalbero di componenti, partendo da quel componente come radice. Per evitare re-render inutili devi utilizzare i `PureComponent` o implementare `shouldComponentUpdate` ogni volta che ti è possibile. Inoltre potresti dover utilizzare strutture dati immutabili in modo da rendere l'ottimizzazione dei cambiamenti di state più semplice. Tuttavia, in certi casi potrebbe non essere possibile affidarsi a questa funzionalità di ottimizzazione perchè `PureComponent/shouldComponentUpdate` assume che il render di tutto il sottoalbero è determinato dalle props del componente corrente. Se questo non fosse il caso, allora questo tipo di ottimizzazione potrebbe portare a dei DOM state inconsistenti.

In Vue, le dipendenze di un componente sono controllare durante il suo render, in modo che il sistema sappia precisamente quale componente effettivamente ha bisogno di un re-render quando lo state cambia. Ogni componente può essere considerato come se avesse `shouldComponentUpdate` automaticamente implementato per te, senza le annesse particolari problematiche. 

Complessivamente questo rimuove la necessità di un'intera classe di ottimizzazioni delle prestazioni dalle mani dello sviluppatore, e permette quindi di concentrarsi di più sullo sviluppo dell'app in sè mentre questa scala in dimensione. 

### HTML & CSS

In React, tutto è JavaScript. Non solo le strutture HTML sono espresse tramite JSX, ma i recenti trend tendono a gestire anche il CSS dentro Javascript. Questo approccio ha i suoi benefici, ma porta con se anche vari compromessi che non è detto valgano la pena per tutti gli sviluppatori.

Vue abbraccia le tecnologie web classiche e costruisce al di sopra di esse. Per mostrare cosa questo comporta, esamineremo alcuni esempi.

#### JSX vs Templates

In React, tutti i componenti dichiarano la propria UI nella render function utilizzando JSX, una sintassi XML-like dichiarativa che funziona all'interno di JavaScript.

Le render function con JSX hanno alcuni vantaggi:

- Puoi approfittare della piena potenzialità di un linguaggio di programmazione (JavaScript) per costruire la tua interfaccia. Questo include variabili temporanee, controllo del flusso e referenze dirette ai valori JavaScript in scope.

- Gli strumenti di supporto (es. linting, controllo dei tipi, autocompletamento dell'editor) per JSX sono sotto alcuni aspetti più avanzati rispetto a ciò che è attualmente disponibile per i template Vue.

In Vue, abbiamo anche le [render functions](render-function.html) e anche il [supporto a JSX](render-function.html#JSX), perchè a volte hai bisogno di quella potenzialità. Tuttavia, come esperienza predefinita offriamo i template come alternativa più semplice. Ogni HTML valido è anche un valido template Vue, e questo porta alcuni vantaggi:

- Per molti sviluppatori che stanno già lavorando con HTML, i template sono più naturali da leggere e scrivere. La preferenza in sè può essere soggettiva, ma se questa rende lo sviluppatore più produttivo allora i benefici sono oggettivi.

- I template basati su HTML rendono più semplice migrare applicazioni esistenti per approfittare dei vantaggi offerti dalle funzionalità reattive di Vue.

- Rende inoltre più semplice per designer e sviluppatori meno esperti analizzare e contribuire alla codebase.

- Puoi anche utilizzare pre-processori come Pug (in passato noto come Jade) per scrivere i tuoi template Vue.

Alcuni discutono sul fatto che avresti bisogno di imparare un altro LDS (linguaggio con dominio specifico) per essere capace di scrivere templates - noi crediamo che questa differenza sia al massimo superficiale. Inanzitutto, l'uso di JSX non implica che l'utente non ha bisogno di imparare nulla - è sintassi addizionale al di sopra del semplice JavaScript, quindi può essere facile da imparare per qualcuno che è familiare con JavaScript, ma dire che è essenzialmente semplice è ingannevole. Similarmente, un template è solo sintassi addizionale al di sopra del semplice HTML. Con l'LDS possiamo inoltre aiutare l'utente a concludere di più con meno codice (ex. il modificatore `v-on`). Lo stesso compito può richiedere molto più codice usando il JSX o le render function.

Su un livello più astratto, possiamo dividire i componenti in due categorie: quelli di presentazione e quelli logici. Noi raccomandiamo usare i template per quelli di presentazione e le render function / JSX per quelli logici. La percentuale di questi componenti dipende dal tipo di applicazione che stai sviluppando, ma in generale troviamo che i componenti di presentazione sono molto più comuni. 

#### CSS Scoped per Componente

A meno che tua non divida i componenti in diversi file (per esemptio con i [Moduli CSS](https://github.com/gajus/react-css-modules)), fare lo scope del CSS in React spesso richiede soluzioni CSS-in-JS (es. [styled-components](https://github.com/styled-components/styled-components) e [emotion](https://github.com/emotion-js/emotion)). Questo introduce un nuovo paradigma di stile orientato ai componenti che è diverso dal normale CSS. In aggiunta, anche se esiste il supporto per estrarre il CSS in una sola pagina a tempo di build, è ancora comune che sia richiesto includere un runtime nel bundle per far funzionare correttamente lo stile. Mentre gudagagni l'accesso al dinamismo di JavaScript mentre costruisci il tuo stile, il compromesso è spesso una dimensione del bundle maggiore e un maggiore costo di runtime. 

Se sei un fan del CSS-in-JS, molte librerie popolari CSS-in-JS supportano Vue (es. [styled-components-vue](https://github.com/styled-components/vue-styled-components) e [vue-emotion](https://github.com/egoist/vue-emotion)). La differenza principale tra React e Vue è che il modo predefinito per stilizzare in Vue è l'uso del familiare tag `style` nei [single-file components](single-file-components.html).

I [single-file components](single-file-components.html) ti danno accesso completo al CSS nello stesso file dove il resto del codice del tuo componente risiede. 

``` html
<style scoped>
  @media (min-width: 250px) {
    .list-container:hover {
      background: orange;
    }
  }
</style>
```

L'attributo opzionale `scoped` limita automaticamente lo scope di questo CSS al tuo componente, aggiungendo un attributo unico (come `data-v-21e5b78`) all'elemento e compilando `.list-container:hover` in qualcosa come `.list-container[data-v-21e5b78]:hover`

Infine, lo stile in un componente in file singolo di Vue è molto flessibile. Tramite il [vue-loader](https://github.com/vuejs/vue-loader), puoi utilizzare qualsiasi preprocessore, postprocessore, e anche profonde integrazioni con i [CSS Modules](https://vue-loader.vuejs.org/en/features/css-modules.html) -- tutto all'interno dell'elemento `<style>`.

### Scalabilità

#### Scalabilità in positivo

Per grandi applicazioni, sia Vue che React offrono soluzioni di routing robuste. La community di React è stata inoltre molto innovativa in termini di soluzioni di state management (es. Flux/Redux). Questi pattern di state management e [anche Redux in sè](https://yarnpkg.com/en/packages?q=redux%20vue&p=1) possono essere facilmente integrate in applicazioni Vue. Infatti, Vue ha anche sviluppato aggiuntivamente questi modelli con [Vuex](https://github.com/vuejs/vuex), una soluzione di state management ispirata da Elm che si integra profondamente con Vue e che pensiamo offre un'esperienza di sviluppo superiore.

Un'altra differenza importante tra queste offerte è che le librerie annesse a Vue per lo state management e il routing (insieme ad [altro](https://github.com/vuejs)) sono tutte ufficialmente supportate e mantenute aggiornate insieme alla libreria core. React invece sceglie di lasciare questo aspetto alla community, creando un ecosistema più frammentato. Essendo più popolare però, l'ecosistema di React è decisamente più ricco di quello di Vue.

Infine, Vue offre una [CLI generatrice di progetti](https://github.com/vuejs/vue-cli) che rende estremamente semplice iniziare un nuovo progetto dato che include una procedura guidata di scaffolding di esso. Puoi anche usarla per creare un [prototipo istantaneo](https://cli.vuejs.org/guide/prototyping.html#instant-prototyping) di un componente. Anche React sta facendo passi da gigante in quest'area con [create-react-app](https://github.com/facebookincubator/create-react-app), ma attualmente ha alcune limtazioni:

- Non permette nessuna configurazione durante la generazione del progetto, mentre la Vue CLI è eseguita al di sopra di una dipendenza runtime aggiornabile che può essere estesa con dei [plugins](https://cli.vuejs.org/guide/plugins-and-presets.html#plugins).
- Offre un solo template che assume che stai sviluppando una single-page application, mentre Vue offre una larga varietà di opzioni predefinite per diversi scopi e sistemi.
- Non può generare progetti da [presets](https://cli.vuejs.org/guide/plugins-and-presets.html#presets) costruiti da utenti, i quali possono essere specialmente utili in ambienti enterprise con convezioni pre-stabilite.

È importante notare che molte di queste limitazioni sono scelte di progettazione intenzionali fatte dal team di create-react-app e offrono i propri vantaggi. Per esempio, fino a quando le necessità del tuo progetto sono molto semplici, non avrai bisogno di scomodarti per personalizzare il tuo processo di build, e sarai capace di aggiornarlo come una dipendenza. Puoi leggere di più sulla [diversa filosofia qui](https://github.com/facebookincubator/create-react-app#philosophy).

#### Scalabilità in negativo

React è rinomato per la sua curva di apprendimento ripida. Prima che tu possa realmente cominciare, hai bisogno di conoscere JSX e probabilmente ES2015+, dato che molti esempi utilizzano la sintassi con classe di React. Dovrai inoltre imparare i sistemi di build, perchè anche se tecnicamente potresti usare Babel da solo per compilare live il tuo codice nel browser, è assolutamente sconsigliato utilizzare una soluzione simile in production.

Mentre Vue scala in positivo così come React, può anche scalare in negativo così come jQuery. Esatto - per cominciare, tutto quello di cui hai bisogno è aggiungere un singolo script tag nella pagina:

``` html
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```

Poi puoi cominciare a scrivere codice Vue e anche lanciare la versione minimizzata in production senza sentirti in colpa e senza preoccuparti di problemi di prestazione. 

Dato che non hai bisogno di sapere JSX, ES2015, o i sistemi di build per cominciare ad usare Vue, solitamente è necessario meno di un giorno di lettura di [questa guida](./) agli sviluppatori per imparare abbastanza per creare applicazioni anche non banali.

### Rendering Nativo

React Native ti permette di scrivere applicazioni renderizzate nativamente per iOS e Android usando lo stesso modello di componenti React. Questo è grandioso dato che come sviluppatore, puoi semplicemente applicare la tua conoscenza di un framework a più piattaforme. Su questo fronte, Vue ha una collaborazione ufficiale con [Weex](https://weex.apache.org/), un framework UI cross-platform creato dal Gruppo Alibaba e incubato dalla Apache Software Foundation (ASF). Weex ti permette di usare la stessa sintassi Vue dei componenti per creare componenti che possono essere renderizzato non solo nel browser, ma anche su iOS e Android!

In questo momento, Weex è ancora in fase di sviluppo attivo e non è maturo e testato in battaglia come React Native, ma il suo sviluppo è spinto dalle necessità del business di e-commerce più grande del mondo, ed inoltre il team di Vue collaborerà attivamente con il team di Weex per assicurarsi un'esperienza di sviluppo più liscia possibile per gli sviluppatori Vue. 

Un'altra opzione è [NativeScript-Vue](https://nativescript-vue.org/), un plugin [NativeScript](https://www.nativescript.org/) per creare applicazioni realmente native utilizzando Vue.js.

### Con MobX

Mobx è diventata alquanto popolare nella community React e utilizza un sistema di reattività praticamente identico al sistema di Vue. In linea di massima, il workflow React + MobX può essere pensato come un Vue più verboso, quindi se tu stai usando questa combinazione e ti sta piacendo, passare a Vue è probabilmente il prossimo passo più sensato.

### Preact e Altre Librerie React-Like

Le librerie React-like solitamente provano a condividere il più possibile delle loro API ed ecosistema con React. Per questa ragione, la maggior parte dei confronti fatti sopra saranno abblicabili anche a loro. La principale differenza sarà tipicamente un ecosistema ridotto, spesso significativamente, rispetto a React. Dato che queste librerie non possono essere compatibili con tutto ciò che viene dall'ecosistema di React al 100%, alcuni strumenti e librerie annesse potrebbero non essere utilizzabili. Oppure, anche quando sembra che funzionino, potrebbero rompersi in qualsiasi momento a meno che la tua specifica libreria React-like non sia supportata ufficialmente e al pari di React.

## AngularJS (Angular 1)

Alcuna della sintassi di Vue sembrerà molto simile a quella di AngularJS (es. `v-if` vs `ng-if`). Questo perchè ci sono molti aspetti che AngularJS ha azzeccato alla perfezione e questi furono un'ispirazione per Vue nelle fasi iniziali dello sviluppo. Ci sono anche molti aspetti negativi inclusi con AngularJS tuttavia, che Vue ha cercato di mitigare con miglioramenti significativi.

### Complessità

Vue è molto più semplice di AngularJS, sia in termini di API che in termini di design. Imparare a sviluppare un'applicazione non banale necessita tipicamente meno di un giorno, cosa non vera per AngularJS.

### Flessibilità e Modularità
AngularJS ha una forte opinione su come la tua applicazione dovrebbe essere strutturata, mentre Vue è una soluzione più flessibile e modulare. Mentre questo rende Vue adottabile da una più vasta varietà di proggetti, riconosciamo che a volte è utile avere qualcuno che ha preso decisioni al posto tuo, in modo che tu possa semplicemente partire con lo sviluppo.

Ecco perchè noi offriamo un sistema completo per lo sviluppo rapido in Vue.js. La [Vue CLI](https://github.com/vuejs/vue-cli) punta ad essere lo strumento predefinito di base per l'ecosistema Vue. Si assicura che i vari strumenti di build lavorino insieme in senza problemi in modo che tu possa concentrarti a sviluppare la tua app invece di spendere ore districandoti tra configurazioni. Allo stesso tempo, la Vue CLI offre la flessibilità di modificare la configurazione di ogni strumento per adattarlo a dei bisogni specifici.

### Binding di dati

AngularJS utilizza un binding a due vie tra i contesti, mentre Vue impone un flusso di dati a una via tra i componenti. Questo rende il flusso di dati più semplice da pensare in un'applicazione non banale.

### Direttive vs Componenti

Vue ha una separazione più chiara tra direttive e componenti. Le direttive sono pensate per incapsulare solo le manipolazioni del DOM, mentre i componenti sono unità autosufficienti che hanno la loro interfaccia e logica dei dati. In AngularJS, le direttive fanno tutto e i componenti sono solo uno specifico tipo di direttiva.

### Prestazioni a Runtime

Vue ha prestazioni migliori ed è molto, molto più semplice da ottimizzare perchè non richiede il dirty checking. AngularJS diventa lento quando ci sono molti watcher, perchè ogni volta che qualcosa nel contesto cambia, tutti questi watcher hanno bisogno di essere eseguiti nuovamente. Inoltre, il ciclo di digest potrebbe dover essere eseguito multiple volte per "stabilire" se qualche watcher innesca qualche altro aggiornamento. Gli utenti di AngularJS spesso devono ridursi all'uso di tecniche esoteriche per aggirare il ciclo di digest, e in alcune situazioni, non c'è nessun modo di ottimizzare un contesto con molti watcher.

Vue non soffre neanche lontanamente di tutto ciò perchè utilizza un sistema di tracciamento e controllo delle dipendenze trasparente con code asincrone - tutti i cambiamenti sono innescati in maniera indipendente a meno che questi non abbiano una relazione di dipendenza esplicita. 

È interessante notare che, ci sono non poche somiglianze in come Angular e Vue stanno affrontando questi problemi di AngularJS.

## Angular (In passato noto come Angular 2)

Abbiamo una sezione separata per il nuovo Angular perchè è veramente un framework completamente diverso da AngularJS. Per esempio, fornisce un sistema di componenti di prima classe, molti dettagli implementativi sono stati completamente riscritti, e anche le API sono cambiate drasticamente.

### TypeScript

Angular essenzialmente richiede l'utilizzo di TypeScript, dato che quasi tutta la sua documentazione e le risorse per imparare sono basate su TypeScript. TypeScript ha i suoi vantaggi - controllo statico dei tipi che può risultare molto utile per applicazioni di larga scara, e può essere un potenziamento per la produttività dello sviluppatore con un passato in Java e C#.

Tuttavia, non tutti vogliono utilizzare TypeScript. In molti utilizzi di piccola scala, introdurre un sistema di tipi può risultare in un lavoro aggiuntivo sproporzionato ai vantaggi e all'incremento di produttività. In questi casi faresti meglio ad utilizzare Vue, dato che Angular senza TypeScript può essere impegnativo.

Infine, anche se non profondamente integrato come TypeScript e Angular lo sono, Vue anche offre [tipi ufficiali](https://github.com/vuejs/vue/tree/dev/types) e [decoratori ufficiali](https://github.com/vuejs/vue-class-component) per chi desidererebbe utilizzare TypeScript con Vue. Stiamo inoltre attivamente collaborando con il team di TypeScript e VSCode di Microsoft per migliorare l'esperienza TS/IDE per utenti Vue + TS.

### Prestazioni a Runtime

Entrambi i framework sono eccezzionalmente veloci, con molte similarità nei valori dei benchmark. Puoi [esplorare metriche specifiche](https://stefankrause.net/js-frameworks-benchmark8/table.html) per un confronto più granulare, ma la velocità è improbabile che sia un fattore decisivo.

### Dimensioni

Le versioni recenti di Angular, con la [compilazione AOT](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) e il [tree-shaking](https://en.wikipedia.org/wiki/Tree_shaking), sono state capace di ridurre la sua dimensione considerevolmente. Tuttavia, un proggetto completo in Vue 2 con Vuex + Vue Router inclusi (~30KB gzipped) è comunque significativamente più leggero di un'applicazione con compilazione AOT generata dalla `angular-cli` (~65KB gzipped).

### Flessibilità

Vue ha un'opinione molto meno forte rispetto ad Angular, offrendo supporto ufficiale a una varietà di sistemi di build, senza alcuna restrizione su come tu strutturi la tua applicazione. Molti sviluppatori gradiscono questa libertà, mentre altri preferiscono avere un solo modo di creare qualsiasi applicazione. 

### Curva di Apprendimento

Per cominciare con Vue, tutto ciò di cui hai bisogno è familiarità con HTML e JavaScript ES5 (ovvero, vanilla Javascript). Con queste abilità di base, puoi cominciare a sviluppare applicazioni non banali in meno di un giorno di lettura [della guida](./).

La curva di apprendimento di Angular è molto più ripida. La superficie di API del framework è enorme e come utilizzatore tu avrai bisogno di acquistare familiarità con molti più concetti prima di essere produttivo. La complessità di Angular è largamente dovuta al suo scopo di puntare solo a grandi e complesse applicazioni - ma questo rende il framework molto più difficile da imparare dagli sviluppatori meno esperti.

## Ember

Ember è un framework full-optional progettato con delle forti opinioni su come le cose dovrebbero essere. Fornisce molte convinzioni già stabilite e una volta che sei famigliare con abbastanza di esse, ti può rendere veramente produttivo. Tuttavia, questo significa che la curva di apprendimento è alta e la flessibilità è scarsa. È un compromesso quando cerchi di scegliere tra un framework con convizioni forti e una libreria con un insieme di strumenti non fortemente correlati che lavorano insieme. La seconda ti dà più libertà ma allo stesso tempo ti richiede di prendere più scelte architetturali.

Detto questo, sarebbe probabilmente meglio fare un confronto tra il core di Vue e i livelli di [templating](https://guides.emberjs.com/v2.10.0/templates/handlebars-basics/) e [object model](https://guides.emberjs.com/v2.10.0/object-model/) di Ember:

- Vue fornisce un piano di reattività non intrusiva al di sopra di JavaScript e delle computed properties completamente automatizzate. In Ember, devi sempre utilizzare un Ember Object come wrapper e dichiarare manualmente le dipendenze per le computed properties.

- La sintassi di template di Vue imbriglia il pieno potere delle espressioni JavaScript, mentre le espresioni Handlebars e la sintassi helper è intenzionalmente limitata in confronto.

- Dal punto di vista delle prestazioni, Vue supera Ember con un [margine notevole](https://stefankrause.net/js-frameworks-benchmark8/table.html), anche dopo l'ultimo aggiornamento del Glimmer engine in Ember 3.x. Vue esegue automaticamente il batch degli update, mentre in Ember devi gestire manualmente i cicli in situazioni di prestazioni critiche. 

## Knockout

Knockout fu un pioniere nel MVVM e nel mondo del controllo delle dipendenze e il suo sistema di reattività è molto simile a quello di Vue. Il suo [supporto ai browser](http://knockoutjs.com/documentation/browser-support.html) è anche molto impressionante considerando tutto quello che può fare, con supporto fino a IE6! Vue invece supporta solo IE9+. 

Nel corso del tempo tuttavia, lo sviluppo di Knockout è rallentato e ha cominciato a mostrare un pò i suoi anni. Per esempio il suo sistema di componenti manca di un insieme completo di lifecycle hooks e sebbene sia una circostanza di utilizzo molto comune, l'interfaccia per passare figli a dei componenti sembra un pò goffa rispetto a quella di [Vue](components.html#Content-Distribution-with-Slots).

Sembra anche esserci una differenza filosofica nel design delle API che se sei curioso, può essere dimostrata da come ognuno gestisce la creazione di una [semplice todo list](https://gist.github.com/chrisvfritz/9e5f2d6826af00fcbace7be8f6dccb89). È decisamente qualcosa di soggettivo sotto qualche aspetto, ma molti considerano le API di Vue meno complesse e meglio strutturate.

## Polymer

Polymer è un'altro progetto sponsorizzato da Google ed infatti fu anch'esso fonte di isparzione per Vue. I componenti di Vue possono essere paragonati ai Custom element di Polymer ed entrambi forniscono uno stile di sviluppo molto simile. La più grande differenza è che Polymer è costruito al di sopra dell'ultima funzionalità di Web Components e richiede polyfills non banali per funzionare (con prestazioni degradati) nei browser che non supportano questa funzionalità nativamente. Vue invece funziona senza alcuna dipendenza o polyfills fino a IE9.

In Polymer inoltre, il team ha reso il sistema di binding dei dati molto limitato per compensare le prestazioni. Per esempio, l'unica espressione supportata nei template Polymer sono le negazioni booleane e le singole chiamate a metodo. Anche la sua implementazione delle computed properties non è molto flessibile.

## Riot

Riot 3.0 fornisce un modello di sviluppo simile basato sui componenti (il quale è chiamato "tag" in Riot), con delle API minimali e ben progettate. Riot e Vue probabimente condividono molto per quanto riguarda la filosofia di design. Tuttavia, anche essendo leggermente più pesante di Riot, Vue offre alcuni vantaggi significativi:

- Prestazioni migliori. Riot esegue un [tree traversal sul DOM](https://v3.riotjs.now.sh/compare/#virtual-dom-vs-expressions-binding) invece di usare un virtual DOM, quindi soffre degli stessi problemi di prestazione di AngularJS
- Strumenti di supporto più maturi. Vue offre supporto ufficiale a [webpack](https://github.com/vuejs/vue-loader) e [Browserify](https://github.com/vuejs/vueify), mentre Riot si appoggia al supporto della community per l'integrazione dei sistemi di build.
