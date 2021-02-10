# Principio

Il primo approccio funzionale in JavaScript lo potremmo aver fatto con librerie come [UnderscoreJS](https://underscorejs.org/) o [Lodash](https://lodash.com/). Successivamente grazie alle nuove funzionalità di **ES6** abbiamo imparato ad usare le funzioni come **map**, **filter** e **reduce**, cioè funzioni che accettano altre funzioni iterando i nostri Array e permettendoci di focalizzarci più su cosa fare che sul come iterare la nostra lista.
Librerie come React e tecniche di virtualizzazione del DOM ci portano a creare funzioni pure per la manipolazione dello stato.

Per chi viene da esperienze di NodeJS invece saprà usare il pattern delle callback, in cui passiamo funzioni ad altre funzioni e che verranno eseguite in maniera asincrona.

Le difficoltà però rimangono. Con JavaScript non abbiamo una tipizzazione che aiuta sia noi, sia il compilatore, a catturare errori a compile-time. La sintassi è verbosa, se comparata con linguaggi puramente funzionali come Haskell, Lisp o Erlang. La gestione dell'astrazione, delle uguaglianze, del pattern matching per la gestione dei branch di codice non è delle migliori.

Per cominciare a "giocare" con la FP in JavaScript dobbiamo porci dei limiti per pensare ad approci alternativi al nostro codice "imperativo":

* Niente if (solo operatori ternari)
* Niente Loop
* Niente riassegnazioni all'interno delle funzioni pure
* Le funzioni non modificano i parametri di ingresso
* No Side Effects
* Body delle funzioni con una singola istruzione di return contenente una espressione (complessa a piacere)
* Le funzioni hanno arietà 1

Altri imposizioni che potremmo fare:

* Non usare Array ma oggetti List
* Utilizzare TypeScript
* Utilizzare FP-TS

Vediamo un esempio pratico:

```js
const sommaRange = i => i<=0 ? 0 : sommaRange(i-1);
sommaRange(10); // 55
```

Vediamo come rendere la somma di due unaria:

```js
const somma = primoAddendo => secondoAddendo => primoAddendo + secondoAddendo;
```

Un oggetto List lo possiamo pensare come una coppia `<v,c>` in cui `v` rappresenta il valore i-esimo e `c` la continuazione della lista (coda). Considerando che la lista vuota ha come valore `null` e che il primo elemento lo costruisco con la coppia `<valore, null>`:

```js
const pair = first => second => ({ first, second});
const lista = pair(20)(pair(10)(null));
```

Ora definisco delle funzionalità per restituire il primo elemento ed il secondo:

```js
const first = list => list.first;
const second = list => list.second;
```

Questo vuol dire che `fist` mi ritorna la testa `head` della lista e `second` mi ritorna la coda `tail`:

```js
const head = first;
const tail = second;
```

Ora costruisco una funzione di comodo che rappresenta il "ponte" tra il mondo dichiarativo e quello imperativo, in questo caso ignoro le limitazioni:

```js
const list2array = list => {
    const array = [];
    while(list) {
        array.push(head(list));
        list = tail(list)
    }
    return array;
}
```

e ora l'esatto contrario:

```js
const array2list = array => {
    let list = null;
    Array.from(array).reverse().forEach(elem => {
        list = pair(elem)(list);
    });
    return list;
}
```

Il reverse è necessario altrimenti otterrei una lista speculare all'array di partenza.
ed ora posso scrivere qualcosa del tipo:

```js
list2array(array2list([1,2,3]));
```

Come creo una funzione "range" in maniera funzionale? Prendo due valori "high" e "low" e in maniera ricorsiva chiamo la funzione "pair". Il caso base sarà quando "low>high" e ritorno null, altrimenti ritorno pair(low)(ricorsione). La parte di ricorsione sarà richiamare la range(low+1)(high):

```js
const range = low => high => low > high ? null : pair(low)(range(low+1)(high));
```

Adesso per applicare dei morfis...ops trasformazioni userò "map". La map prende una funzione e ritorna una funzione che a sua volta prende una lista e applica la funzione ad ogni singolo elemento.

Devo creare una nuova Lista e non modificare la mia Lista sorgente!

Anche qui applico la ricorsione, in cui il caso base è la lista a null e ritorno null, altrimenti quello che farò è mettere in PAIR la mia funzione che prende la TESTA della lista: pair(f(head(lista)))(ricorsione)
la parte ricorsiva sarà richiamare map a cui passo la funzione, a seguite la coda della lista: map(f(tails(lista)))
Quindi avrò: `pair(f(head(lista)))(map(f)(tail(lista)))`:

```js
const map = f => lista => lista
    ? pair(f(head(lista)))(map(f)(tail(lista)))
    : null;

list2array(map(x => x + x)(range(1)(3)))
```