# Riflessioni su AngularJS: per capire la logica della dashboard
---

## 1. Introduzione
Navigando nel codice della LPPM dashboard avrete notato che in un paio di punti appare una cosa di questo tipo:
```javascript
.service('Dataview', function($rootScope){
    //...
    $rootScope.$broadcast('dataview.updated');
});
```
e alcuni di voi diranno: ma come? Non era una cosa da non fare _mai_? Io vi cito Fabrizio de Andri:
> non spalancare le labbra ad un ingorgo di parole  
> le tue labbra cosl, frenate nelle fantasie dell'amore  
> dopo l'amore cosl, sicure a rifugiarsi nei "sempre",  
> nell'ipocrisia dei "mai" 

Mi sono senza dubbio opposto a questa abitudine, frenandola in modo molto deciso. Il motivo h che, per esperienza, gli eventi, ad occhio inesperto, sembrano _comodi_ e _veloci_ ma troppo spesso li ho visti degenerare diventando uno **strumento automatico di generazione di bordello**.

Questo non significa che il modello publisher/subscriber sia _sbagliato_ di per sh, nh ho nulla contro questo, volevo solo che venisse usato, come ogni strumento, nel modo giusto.
> A volte ho avvitato viti con la punta di un coltello, perchh non avevo un cacciavite  
> Questo non significa che il coltello sia un cacciavite.  
> Nh che, in presenza di un cacciavite, userei un coltello.  
> Nh che taglierei la bistecca con un cacciavite.  
> Nh che io abbia nulla contro i coltelli o i cacciaviti.

## 2. Angular best practices
A parte il mio essere [manicheo] [manicheismo], c'h anche una questione di _best practices_.  
* La comunicazione tra controllers avviene attraverso i servizi (documentatissimo). Usare solo gli eventi vuol dire fare un bordello.
* Spesso non ho bisogno di un evento, ma h molto piy espressivo, comprensibile, leggibile e **tracciabile** dire ad un servizio di fare qualcosa e lui lo fa ed eventualmente segnala (altri, a mia insaputa).
* In generale, h buona pratica _non iniettare_ `$scope` _nei controllers_. Per esperienza, se serve lo scope, serve una directive (e le sue `link` functions).
* Non per nulla, i servizi (e le factory) non possono avere iniettato uno `$scope`, ma solo `$rootScope`
* I controllers non sono altro che un'API per la UI per avere i dati. La logica per ottenere questi dati non va necessariamente (e sarebbe in effetti meglio evitare di metterla) nei controllers (anche se viene spesso piC9 comodo, lo so)
* `$emit` e `$broadcast` vanno capiti bene. Uno va in su, uno in giC9. Ma su e giC9 di cosa? E qui si apre un _vaso di Pandora_: l'ereditariet`  prototipale degli `$scope` vs lo `$scope.$parent` (no, non sono necessariamente la stessa cosa).
* Il caso precedente h simile al `onModelChanged` che abbiamo messo in piedi in LPPM. LC, ho spinto per farlo a mano. Forse ho esagerato... ma non so.

## 3. Cos'è `$rootScope`
Quando una app viene su, viene creato un `$rootScope`. Questo h lo scope padre di tutti gli scope. Vive per sempre, dall'inizio alla fine della vita dell'applicazione. Su Stack Overflow h pieno di gente che ha preso questo concetto e ha detto: bene, allora io butto roba nel `$rootScope` e di fatto ce l'ho ovunque. Tradotto: creo dei globali. Che sono un male assoluto (e questo lo dico senza mezze misure e senza timore di essere smentito[^martello]).
>`$rootScope` **non** h uno storage globale.

Perr di `$rootScope` si possono usare i `$watch*` e, perchh no, il `$broadcast`, che sono di fatto delle utility. Con parsimonia perr. E tenendo conto che:
* `$rootScope` va bene iniettarlo in un servizio, ma cosl si fa solo `$broadcast` (segnalazione verso i figli)
* Nei controller bisognerebbe fare di tutto per evitare di iniettare `$scope`. Perchh? Si veda dopo...
* `$emit`, che nasce dal basso, quindi da uno `$scope` va limitato:
    * Non h sempre evidente chi sono i padri (vd directive e transclude)
    * Non funziona coi fratelli
    * Hanno inventato i servizi

[^martello]: Questa h una di quelle cose per cui davvero userei il martello.

## 4. Controllers e `$scope`
Anche se Javascript non h _tecnicamente_ un linguaggio orientato agli oggetti, mi piace pensare di poter miscelare concetti di OOP per quel che riguarda l'architettura e concetti di funzionale per la parte piy algoritmica. Alla fine AngularJS h un framework e come tale punta sull'architettura, con delle cose piy zuccherose qui e l`.  
Non solo, Angular2 e ECMA6 hanno il concetto di classi. Sone sempre classi ad ereditariet` prototipale, quindi un concetto diverso da quello a cui siamo abituati con altri linguaggi OO, come Java o C#.
Questi sono comunque i motivi per cui mi piace l'idea di avere dei controller senza `$scope` iniettato, ma che usino la sintassi `this` e `controllerAs`. Non so se h piy chiara, forse no, forse h anche piy complessa, ma mi evita una cosa: confondere lo `$scope` con un posto in cui versare mille variabili e funzioni. Vorrei continuare a vederlo solo come il modo di comunicare con il framework.

Non solo, il controller h in sostanza solo una API con la UI (HTML) e deve fornire ad essa i dati e il modo per interagire con questi dati da parte dell'utente. Per il resto ci sono i servizi e, ancor piy importante, le directives.

## 5. Directives
Secondo me le directives sono la prima cosa da capire di AngularJS. Non con gli esempietti stupidi, proprio i concetti dietro a:
* scope isolato
* transclusion
* controller
* pre e post link functions

Questi concetti aprono letteralmenet un mondo e mi hanno portato a vedere i controller come la _parte di logica di una directive_ piy che una cosa a parte e a si stante.

Quando si crea un controller all'inizio lo si appiccica su una elemento del DOM con `ng-controller="myCtrl"`. Tutto bene, ma non si capisce cosa sta succedendo. Si sta trattando quel DOM element di fatto come una mega directive in puro HTML, quindi priva di aggiornamenti, di inizializzazione ed eventualmente... con quale... `$scope`?

Le uniche interazioni dinamiche sono attraverso l'uso di directive bult-in come le classiche `ng-model` e `ng-repeat`. Appena si vuole fare qualcosa di piy, e purtroppo la rete h piena di questi esempi veramente di bassa qualit`, h iniettare lo `$scope` nel controller ed usarlo da ll. 
> Iniettare lo `$scope` nel controller h solo una scorciatoia

Perchi quello `$scope` h anche di (alcuni) figli e di tutti gli elementi dentro l'elemento padre in cui abbiamo messo il controller. E' solo zucchero invece di una piy complessa:
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
Qui perr si vede chiaramente tutto il mondo: 
* c'h dell'HTML
* quell'HTML h passato alla link function attraverso elem, quindi ho accesso al DOM
* viene instanziato un controller con cui posso parlare sia dalla link che dall'HTML
* nella link ho lo `$scope` iniettato come variabile

Vista cosl, a cosa serve iniettare lo `$scope` nel controller?
E non h solo questo:
* Succede lo stesso con UI router
* Succede lo stesso con `$uibModal`
* due indizi fanno... quasi una prova.

## 6. Ma quindi... sti eventi?
Premesso tutto questo, gli eventi che girano negli `$scope` sono una _feature_ utilizzabile, visto soprattutto un grandissimo vantaggio: le sottoscrizioni _muoiono con lo `$scope` stesso_. Quindi non serve deregistrarsi. Ripeto perr che secondo me servono due requisiti:
* Si usa solo il `$broadcast`, ovvero vengono solo tirati dall'alto
* Non se ne abusa, meglio una chiamata esplicita ed un API chiara.

Vediamo un esempio tratto da una storia vera: la dashboard.
Attori:
* **Dataset**: Dataset h il _set completo dei dati_ (che arrivano dal server). E' un `service`
* **Dataview**: h una view sul dataset, ovvero h il dataset al netto dei filtri. Dataview h l'unico che conosce il Dataset e la sua esistenza. Il resto del mondo conosce solo Dataview. E' un `service` che espone di fatto solo `flush()` che prende il dataset, lo fa filtrare dai filtri e mette a disposizione il risultato.
* **Sidebar**: il controller a sinistra. Di fatto mostra i filtri, scatena l'edit (che h responsabilit` dei filtri stessi) e chiede al Dataview di far `flush()`
Quando cambio un filtro nella sidebar, la sidebar(controller) dice al Dataview (servizio) di fare `flush()`
* **Main** il controller di destra. Chiede ai filtri di dar la descrizione dei filtri attivi (i tag in alto), si occupa di mostrare le diverse _view_ (i report). Prende dal Dataview i dati quando disponibili e li _inoltra_ alle view sotto. Non fa praticmente nulla.

Il problema h come far comunicare `sidebar` e `main`. La soluzione piy semplice h: renderli un solo controller. Non vi h nulla di male a farlo cosl tranne che ha molte dipendenze iniettate, che anche a livello di DOM h molto "ingombrante" e che fa cose molto diverse dal setup dei filtri, alla gestione dei repot. Perr nessun problema sulla comunicazione. Perchi non l'ho fatto cosl?

Perchi guardando la pagina quelle due parti, sinistra e centro, sono due directive. O se volete due view dello stesso `$state`, come il menu, i dettagli e il QPL di LPPM. C'h anche una questione di ordine, non voglio controller giganti e di curiosit`: come farli parlare diventa una sfida.  
Quindi l'ho impostato cosl, non ho fatto le views dello `$state` nh due directive, ma stanno in un solo HTML con due controller dichiarati nell'HTML stesso. Passare alle directives o alla route e le view h banale da questo punto di partenza.

Ok, come segnalare che i filtri sono cambiati?  
Soluzione 1: eventi puri
```javascript
.service('Dataview', function($rootScope){
    //...
    $rootScope.$on('filters.changed', function(){
        // Refresh dataview data
        $rootScope.$broadcast('dataview.updated');
    });
})
.controller('Sidebar', function($scope){
    //...
    $scope.$emit('filters.changed');
})
.controller('Main', function($scope){
    //...
    $scope.$on('dataview.updated', function(){
        // refresh the local dataview that is shared with the below reports
    });
})
```
l' `$emit` va su su fino al `$rootScope`, viene pescato dal Dataview, che  fa quel che deve e rilancia un altro evento. Sembra tutto bello e separato anche perchi la Sidebar h agnostica sul fatto che ci sia una Dataview o chiss` chi altro in ascolto. Perr:
* se in ascolto dell'`$emit` h un fratello, non lo sente
* con un evento h facile, anche se vi potrebbe servire fare debug e capire chi becca sto evento. Se ne mettete due o tre, poi ci sono eventi che si chiamo in sequenza, vi assicuro, in un attimo perdete il controllo di chi chiama cosa e in che ordine.
* Il pub/sub h sincrono. Quindi hai voglia a mettere piy operazione in sequenza.
* Ma quindi se voglio rinfrescare i dati del Dataview devo tirare un evento che si chiama `filters.changed`? Allora forse sto evento lo deve tirare il Filter Engine stesso. Oppure si deve chiamare `dataview.update.request` ma a quel punto io so che serve al dataview. Boh.
* Ho iniettato lo scope nei controller.
* Mi sta meglio il `$broadcast` del servizio perchi lui mette a disposizione un array e sta segnalando ad altri che h cambiato. E' meglio di un `$watch` sull'array da parte dei figli perchi costa di meno e mi evita ancora di avere lo `$scope`.

Ho preferito una cosa piy standard:
```javascript
.service('Dataview', function($rootScope){
    //...
    this.flush = function(){
        // Refresh dataview data
        $rootScope.$broadcast('dataview.updated');
    });
})
.controller('Sidebar', function(Dataview){
    //...
    Dataview.flush(); // Piy esplicito
})
.controller('Main', function($scope){
    //...
    $scope.$on('dataview.updated', function(){
        // refresh the local dataview that is shared with the below reports
    });
})
```
Si potrebbe avere anche una directive dedicata ad ascoltare gli eventi provenienti dal dataview (e che quindi sta nel modulo con Dataview)
```javascript
.directive('dataviewListener', function(){
   return {
        scope: {
            onDataviewUpdated: '&'
        }
        link: function(scope){
            scope.on('dataview.updated', onDataviewUpdated());
        }
   };
});
```
che implica nell'HTML una cosa cosl:
```html
<div ng-controller="mainCtrl as main" 
     dataview-listener 
     onDataviewUpdated="main.onDataviewChanged()">
<!-- other stuff -->
</div>
```
Verboso? Forse, ma:
* il nome dell'vento non lo sa nessuno, se non i membri di Dataview.Module
* **reuse**: lo pur utilizzare chiungue abbia bisgno di sapere che h cambiata la dataview
* ho disaccoppiato l'ascolto dell'evento (del modulo Dataview) dall'azione (la decide il controller in cui voglio agire)
* Nessuno `$scope` nel posto sbagliato....

## 7. Save configuration
Lo stesso identico problema, anche piy grande in realt`, si trova se voglio salvare le configurazioni.  
La sidebar ha il bottone: faccio save. Cosa salvo:
* Lo stato dei filtri
* Lo stato dei report (quali, in che posizione e con quali settaggi)

Quindi:
* Ogni filtro deve sapersi serializzare
* Ogni report deve sapersi serializzare
* La sidebar genera il tutto
* Il main deve saperlo, perchi deve avvisare le view figlie.
* Oppure le view figlie devono essere avvisate e il main manco lo sa... eventi? Pur essere un buon uso qui, oppure directive ad hoc come prima


---
**Thanks to [Dillinger.io] [dill] for providing the sotware to easily write this document**

**Free Software, Hell Yeah!**

   [manicheismo]: <https://it.wikipedia.org/wiki/Manicheismo>
   [dill]: <https://github.com/joemccann/dillinger>
