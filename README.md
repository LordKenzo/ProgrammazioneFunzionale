# Programmazione Funzionale

Questo repository rappresenta un notebook virtuale di appunti presi durante il mio tentativo 😅 di studio e comprensione della Programmazione Funzionale.
Non è una guida, un libro o qualcosa di strutturato e pensato a priori, pertanto è soggetta ad: errori, orrori e mal di testa 🤕

Per definire la **Programmazione Funzionale** dobbiamo "incastrare" più concetti, alcuni presi dalla matematica ed in particolare della lambda calcolo. E' un paradigma che fa uso della composizione di funzioni che ritornano valori a partire da espressioni contenute nel body. Le funzioni sono **Cittadini di Primi Classe**, in cui la funzione è trattata come un qualsiasi altro valore e pertanto posso usarla come parametro o valore di ritorno. Le caratteristiche fondamentali della PF sono:

* Trasparenza referenziale;
* Funzioni Pure;
* No Side Effect;
* Immutabilità;
* Ricorsione;

Quando creiamo un applicazione completa avremo una parte puramente funzionale ed un'altra che si occuperà del lato "oscuro" 🥷🏽.

## Indice

* [Principio](./principio.md)
* [Concetti di base](./algebra.md)
* [Introduzione FP](./intro-fp.md)
* [Primi Step con FP-TS](./first-contact-fp-ts.md)
* [Contenitori](./container.md)

## Concetti brevi

> Programmazione Dichiarativa: si contrappone alla Programmazione Imperativa in cui ci preoccupiamo a specificare il "come" otteniamo qualcosa, step-by-step. La programmazione dichiarativa invece si preoccupa di indicare "cosa" vogliamo ottenere, senza scendere nel dettaglio implementativo.

> Il tipo mi rappresenta la categoria con cui qualcosa, un dato, un espressione, una funzione possa essere rappresentata/o. E' il dominio di valori ammissibili che posso avere.

> "First Class Citizen" quando posso trattare questa entità come tratto un qualsiasi dato, un linguaggio in cui le funzioni sono first class citizen significa che posso passare una funzione o riceverla come risultato di una esecuzione.

> Una funzione pura non presenta interazioni col mondo esterno, in pratica non si osservano side effects cioè non si effettuano elaborazioni al di fuori del suo scopo di prendere un input e fornire un output che sarà uguale per il medesimo input. Non sono funzioni pure se: leggo un dato nel db, faccio richieta remota, accedo a variabili esterne alla funzione, accedo ad un valore che cambia nel tempo (data/timer), stampo a video. Tutto questo mi rende la funzione non più deterministica. Si useranno modelli che permettono un cambiamento (side effect) in maniera "ordinata", tramite Funtori, Monadi, ecc...

> Una funzione pura restituisce sempre lo stesso output per lo stesso input, questo è la proprietà di trasparenza referenziale. Questo mi permette di mettere in pratica tecniche di memoizzazione.

> La composizione di funzione permette di riutilizzare funzioni insieme combinandole per ottenre funzionalità più complesse. In generale parliamo di funzioni con arietà pari ad 1.

> I SideEffects vengono gestiti o "wrappati" in maniera controllata.

> Immutable Data Structure: in un linguaggio puro (Haskell) ho questa garanzia.

> Strictly Typed: significa che una volta che assegno il Tipo ad un oggetto/entità/variabile non potrò più variarla. Il compilatore ha conoscenza del tipo a compile time.

> Il caching e la memoizzazione sono entrambi relativi alla memorizzazione, il primo ha a che fare con strutture mutabili, il secondo con strutture immutabili.