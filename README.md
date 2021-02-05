# Programmazione Funzionale

Questo repository rappresenta un notebook virtuale di appunti presi durante il mio tentativo 😅 di studio e comprensione della Programmazione Funzionale.
Non è una guida, un libro o qualcosa di strutturato e pensato a priori, pertanto è soggetta ad: errori, orrori e mal di testa 🤕

## Indice

* [Concetti di base](./algebra.md)
* [Introduzione FP](./intro-fp.md)
* [Contenitori](./container.md)

## Concetti brevi

> Il tipo mi rappresenta la categoria con cui qualcosa, un dato, un espressione, una funzione possa essere rappresentata/o. E' il dominio di valori ammissibili che posso avere.

> "First Class Citizen" quando posso trattare questa entità come tratto un qualsiasi dato, un linguaggio in cui le funzioni sono first class citizen significa che posso passare una funzione o riceverla come risultato di una esecuzione.

> Una funzione pura non presenta interazioni col mondo esterno, in pratica non si osservano side effects cioè non si effettuano elaborazioni al di fuori del suo scopo di prendere un input e fornire un output che sarà uguale per il medesimo input. Non sono funzioni pure se: leggo un dato nel db, faccio richieta remota, accedo a variabili esterne alla funzione, accedo ad un valore che cambia nel tempo (data/timer), stampo a video. Tutto questo mi rende la funzione non più deterministica. Si useranno modelli che permettono un cambiamento (side effect) in maniera "ordinata".

> La composizione di funzione permette di riutilizzare funzioni insieme combinandole per ottenre funzionalità più complesse.