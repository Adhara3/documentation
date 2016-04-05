# Publisher subscriber in AngularJS
---

## 1. Introduzione
Navigando nel codice della LPPM dashboard avrete notato che in un paio di punti appare una cosa di questo tipo:
```javascript
.service('Dataview', function($rootScope){
    /...
    $rootScope.$broadcast('dataview.updated');
});
```
e alcuni di voi diranno: ma come? Non era una cosa da non fare _mai_? Io vi cito Fabrizio de André:
> non spalancare le labbra ad un ingorgo di parole \
> le tue labbra così frenate nelle fantasie dell'amore \
> dopo l'amore così sicure a rifugiarsi nei "sempre", \
> nell'ipocrisia dei "mai" 

Mi sono senza dubbio opposto a questa abitudine, frenandola in modo molto deciso. Il motivo è che, per esperienza, gli eventi, ad occhio inesperto, sembrano _comodi_ e _veloci_ ma troppo spesso li ho visti degenerare diventando uno **strumento automatico di generazione di bordello**.

Questo non significa che il modello publisher/subscriber sia _sbagliato_ di per sé, nè ho nulla contro questo, volevo solo che venisse usato, come ogni strumento, nel modo giusto.
> A volte ho avvitato viti con la punta di un coltello, perché non avevo un cacciavite \
> Questo non significa che il coltello sia un cacciavite. \
> Né che, in presenza di un cacciavite, userei un coltello.\
> Né che taglierei la bistecca con un cacciavite.\
> Nè che io abbia nulla contro i coltelli o i cacciaviti.

## 2. Angular best practices
A parte il mio essere [manicheo] [manicheismo], c'è anche una questione di _best practices_.  
* La comunicazione tra controllers avviene attraverso i servizi (documentatissimo). Usare solo gli eventi vuol dire fare un bordello.
* Spesso non ho bisogno di un evento, ma è molto più espressivo, comprensibile, leggibile e **tracciabile** dire ad un servizio di fare qualcosa e lui lo fa ed eventualmente segnala (altri, a mia insaputa).
* In generale, è buona pratica _non iniettare_ `$scope` _nei controllers_. Per esperienza, se serve lo scope, serve una directive (e le sue `link` functions).
* Non per nulla, i servizi (e le factory) non possono avere iniettato uno `$scope`, ma solo `$rootScope`
* I controllers non sono altro che un'API per la UI per avere i dati. La logica per ottenere questi dati non va necessariamente (e sarebbe in effetti meglio evitare di metterla) nei controllers (anche se viene spesso più comodo, lo so)
* `$emit` e `$broadcast` vanno capiti bene. Uno va in su, uno in giù. Ma su e giù di cosa? E qui si apre un _vaso di Pandora_: l'ereditarietà prototipale degli `$scope` vs lo `$scope.$parent` (no, non sono necessariamente la stessa cosa).
* Il caso precedente è simile al `onModelChanged` che abbiamo messo in piedi in LPPM. Lì ho spinto per farlo a mano. Forse ho esagerato... ma non so.

## 3. Cos'è `$rootScope`
Quando una app viene su, viene creato un `$rootScope`. Questo è lo scope padre di tutti gli scope. Vive per sempre, dall'inizio alla fine della vita dell'applicazione. Su Stack Overflow è pieno di gente che ha preso questo concetto e ha detto: bene, allora io butto roba nel `$rootScope` e di fatto ce l'ho ovunque. Tradotto: creo dei globali. Che sono un male assoluto (e questo lo dico senza mezze misure e senza timore di essere smentito[^martello]).
>`$rootScope` **non** è uno storage globale.

Però di `$rootScope` si possono usare i `$watch*` e, perché no, il `$broadcast`, che sono di fatto delle utility. Con parsimonia però. E tenendo conto che:
* `$rootScope` va bene iniettarlo in un servizio, ma così si fa solo `$broadcast` (segnalazione verso i figli)
* Nei controller bisognerebbe fare di tutto per evitare di iniettare `$scope`. Perché? Si veda dopo...
* `$emit`, che nasce dal basso, quindi da uno `$scope` va limitato:
    * Non è sempre evidente chi sono i padri (vd directive e transclude)
    * Non funziona coi fratelli
    * Hanno inventato i servizi

[^martello]: Questa è una di quelle cose per cui davvero userei il martello.

## 4. Controllers e `$scope`

Dillinger is a cloud-enabled, mobile-ready, offline-storage, AngularJS powered HTML5 Markdown editor.

  - Type some Markdown on the left
  - See HTML in the right
  - Magic

Markdown is a lightweight markup language based on the formatting conventions that people naturally use in email.  As [John Gruber] writes on the [Markdown site][df1]

> The overriding design goal for Markdown's
> formatting syntax is to make it as readable
> as possible. The idea is that a
> Markdown-formatted document should be
> publishable as-is, as plain text, without
> looking like it's been marked up with tags
> or formatting instructions.

This text you see here is *actually* written in Markdown! To get a feel for Markdown's syntax, type some text into the left window and watch the results in the right.

### Version
3.2.7

### Tech

Dillinger uses a number of open source projects to work properly:

* [AngularJS] - HTML enhanced for web apps!
* [Ace Editor] - awesome web-based text editor
* [Marked] - a super fast port of Markdown to JavaScript
* [Twitter Bootstrap] - great UI boilerplate for modern web apps
* [node.js] - evented I/O for the backend
* [Express] - fast node.js network app framework [@tjholowaychuk]
* [Gulp] - the streaming build system
* [keymaster.js] - awesome keyboard handler lib by [@thomasfuchs]
* [jQuery] - duh

And of course Dillinger itself is open source with a [public repository][dill]
 on GitHub.

### Installation

You need Gulp installed globally:

```sh
$ npm i -g gulp
```

```sh
$ git clone [git-repo-url] dillinger
$ cd dillinger
$ npm i -d
$ gulp build --prod
$ NODE_ENV=production node app
```

### Plugins

Dillinger is currently extended with the following plugins

* Dropbox
* Github
* Google Drive
* OneDrive

Readmes, how to use them in your own application can be found here:

* [plugins/dropbox/README.md] [PlDb]
* [plugins/github/README.md] [PlGh]
* [plugins/googledrive/README.md] [PlGd]
* [plugins/onedrive/README.md] [PlOd]

### Development

Want to contribute? Great!

Dillinger uses Gulp + Webpack for fast developing.
Make a change in your file and instantanously see your updates!

Open your favorite Terminal and run these commands.

First Tab:
```sh
$ node app
```

Second Tab:
```sh
$ gulp watch
```

(optional) Third:
```sh
$ karma start
```

### Docker
Dillinger is very easy to install and deploy in a Docker container.

By default, the Docker will expose port 80, so change this within the Dockerfile if necessary. When ready, simply use the Dockerfile to build the image. 

```sh
cd dillinger
docker build -t <youruser>/dillinger:latest .
```
This will create the dillinger image and pull in the necessary dependencies. Once done, run the Docker and map the port to whatever you wish on your host. In this example, we simply map port 80 of the host to port 80 of the Docker (or whatever port was exposed in the Dockerfile):

```sh
docker run -d -p 80:80 --restart="always" <youruser>/dillinger:latest
```

Verify the deployment by navigating to your server address in your preferred browser.

### N|Solid and NGINX

More details coming soon.

#### docker-compose.yml

Change the path for the nginx conf mounting path to your full path, not mine!

### Todos

 - Write Tests
 - Rethink Github Save
 - Add Code Comments
 - Add Night Mode

License
----

MIT


**Free Software, Hell Yeah!**

   [manicheismo]: <https://it.wikipedia.org/wiki/Manicheismo>
   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [@thomasfuchs]: <http://twitter.com/thomasfuchs>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [marked]: <https://github.com/chjj/marked>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [keymaster.js]: <https://github.com/madrobby/keymaster>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]:  <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>

