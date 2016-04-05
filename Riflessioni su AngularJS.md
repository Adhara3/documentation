# Riflessioni su AngularJS: per capire la logica della dashboard
---

## 1. Introduzione
Navigando nel codice della LPPM dashboard avrete notato che in un paio di punti appare una cosa di questo tipo:
```javascript
.service('Dataview', function($rootScope){
    /...
    $rootScope.$broadcast('dataview.updated');
});
```
e alcuni di voi diranno: ma come? Non era una cosa da non fare _mai_? Io vi cito Fabrizio de Andr�:
> non spalancare le labbra ad un ingorgo di parole \
> le tue labbra cos�, frenate nelle fantasie dell'amore \
> dopo l'amore cos�, sicure a rifugiarsi nei "sempre", \
> nell'ipocrisia dei "mai" 

Mi sono senza dubbio opposto a questa abitudine, frenandola in modo molto deciso. Il motivo � che, per esperienza, gli eventi, ad occhio inesperto, sembrano _comodi_ e _veloci_ ma troppo spesso li ho visti degenerare diventando uno **strumento automatico di generazione di bordello**.

Questo non significa che il modello publisher/subscriber sia _sbagliato_ di per s�, n� ho nulla contro questo, volevo solo che venisse usato, come ogni strumento, nel modo giusto.
> A volte ho avvitato viti con la punta di un coltello, perch� non avevo un cacciavite \
> Questo non significa che il coltello sia un cacciavite. \
> N� che, in presenza di un cacciavite, userei un coltello.\
> N� che taglierei la bistecca con un cacciavite.\
> N� che io abbia nulla contro i coltelli o i cacciaviti.

## 2. Angular best practices
A parte il mio essere [manicheo] [manicheismo], c'� anche una questione di _best practices_.  
* La comunicazione tra controllers avviene attraverso i servizi (documentatissimo). Usare solo gli eventi vuol dire fare un bordello.
* Spesso non ho bisogno di un evento, ma � molto pi� espressivo, comprensibile, leggibile e **tracciabile** dire ad un servizio di fare qualcosa e lui lo fa ed eventualmente segnala (altri, a mia insaputa).
* In generale, C( buona pratica _non iniettare_ `$scope` _nei controllers_. Per esperienza, se serve lo scope, serve una directive (e le sue `link` functions).
* Non per nulla, i servizi (e le factory) non possono avere iniettato uno `$scope`, ma solo `$rootScope`
* I controllers non sono altro che un'API per la UI per avere i dati. La logica per ottenere questi dati non va necessariamente (e sarebbe in effetti meglio evitare di metterla) nei controllers (anche se viene spesso piC9 comodo, lo so)
* `$emit` e `$broadcast` vanno capiti bene. Uno va in su, uno in giC9. Ma su e giC9 di cosa? E qui si apre un _vaso di Pandora_: l'ereditariet�  prototipale degli `$scope` vs lo `$scope.$parent` (no, non sono necessariamente la stessa cosa).
* Il caso precedente � simile al `onModelChanged` che abbiamo messo in piedi in LPPM. LC, ho spinto per farlo a mano. Forse ho esagerato... ma non so.

## 3. Cos'� `$rootScope`
Quando una app viene su, viene creato un `$rootScope`. Questo � lo scope padre di tutti gli scope. Vive per sempre, dall'inizio alla fine della vita dell'applicazione. Su Stack Overflow � pieno di gente che ha preso questo concetto e ha detto: bene, allora io butto roba nel `$rootScope` e di fatto ce l'ho ovunque. Tradotto: creo dei globali. Che sono un male assoluto (e questo lo dico senza mezze misure e senza timore di essere smentito[^martello]).
>`$rootScope` **non** � uno storage globale.

Per� di `$rootScope` si possono usare i `$watch*` e, perch� no, il `$broadcast`, che sono di fatto delle utility. Con parsimonia per�. E tenendo conto che:
* `$rootScope` va bene iniettarlo in un servizio, ma cos� si fa solo `$broadcast` (segnalazione verso i figli)
* Nei controller bisognerebbe fare di tutto per evitare di iniettare `$scope`. Perch�? Si veda dopo...
* `$emit`, che nasce dal basso, quindi da uno `$scope` va limitato:
    * Non � sempre evidente chi sono i padri (vd directive e transclude)
    * Non funziona coi fratelli
    * Hanno inventato i servizi

[^martello]: Questa � una di quelle cose per cui davvero userei il martello.

## 4. Controllers e `$scope`
Anche se Javascript non � _tecnicamente_ un linguaggio orientato agli oggetti, mi piace pensare di poter miscelare concetti di OOP per quel che riguarda l'architettura e concetti di funzionale per la parte pi� algoritmica. Alla fine AngularJS � un framework e come tale punta sull'architettura, con delle cose pi� zuccherose qui e l�. \
Non solo, Angular2 e ECMA6 hanno il concetto di classi. Sone sempre classi ad ereditariet� prototipale, quindi un concetto diverso da quello a cui siamo abituati con altri linguaggi OO, come Java o C#.\
Questi sono comunque i motivi per cui mi piace l'idea di avere dei controller senza `$scope` iniettato, ma che usino la sintassi `this` e `controllerAs`. Non so se � pi� chiara, forse no, forse � anche pi� complessa, ma mi evita una cosa: confondere lo `$scope` con un posto in cui versare mille variabili e funzioni. Vorrei continuare a vederlo solo come il modo di comunicare con il framework.

Non solo, il controller � in sostanza solo una API con la UI (HTML) e deve fornire ad essa i dati e il modo per interagire con questi dati da parte dell'utente. Per il resto ci sono i servizi e, ancor pi� importante, le directives.

## 5. Directives
Secondo me le directives sono la prima cosa da capire di AngularJS. Non con gli esempietti stupidi, proprio i concetti dietro a:
* scope isolato
* transclusion
* controller
* pre e post link functions

Questi concetti aprono letteralmenet un mondo e mi hanno portato a vedere i controller come la _parte di logica di una directive_ pi� che una cosa a parte e a s� stante.

Quando si crea un controller all'inizio lo si appiccica su una elemento del DOM con `ng-controller="myCtrl"`. Tutto bene, ma non si capisce cosa sta succedendo. Si sta trattando quel DOM element di fatto come una mega directive in puro HTML, quindi priva di aggiornamenti, di inizializzazione ed eventualmente... con quale... `$scope`?

Le uniche interazioni dinamiche sono attraverso l'uso di directive bult-in come le classiche `ng-model` e `ng-repeat`. Appena si vuole fare qualcosa di pi�, e purtroppo la rete � piena di questi esempi veramente di bassa qualit�, � iniettare lo `$scope` nel controller ed usarlo da l�. 
> Iniettare lo `$scope` nel controller � solo una scorciatoia

Perch� quello `$scope` � anche di (alcuni) figli e di tutti gli elementi dentro l'elemento padre in cui abbiamo messo il controller. E' solo zucchero invece di una pi� complessa:
```javascript
.directive('myDirective', function(){
    return {
        scope: false,
        controller: 'myCtrl',
        controllerAs: 'my',
        template: '<div></div>', // even better, templateUrl
        link: function(scope, elem, attrs, ctrl){
            //... here we have the scope, the current element and... the controller!!
        }
    };
});
```
Qui per� si vede chiaramente tutto il mondo: 
* c'� dell'HTML
* quell'HTML � passato alla link function attraverso elem, quindi ho accesso al DOM
* viene instanziato un controller con cui posso parlare sia dalla link che dall'HTML
* nella link ho lo `$scope` iniettato come variabile

Vista cos�, a cosa serve iniettare lo `$scope` nel controller?
E non � solo questo:
* Succede lo stesso con UI router
* Succede lo stesso con `$uibModal`


---
**Thanks to [Dillinger.io] [dill]

**Free Software, Hell Yeah!**

   [manicheismo]: <https://it.wikipedia.org/wiki/Manicheismo>
   [dill]: <https://github.com/joemccann/dillinger>
