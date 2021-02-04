# Principi

Quando scrivo codice ho degli obiettivi da perseguire per creare codice di qualità:

* Modulare
* Estendibile
* Performante
* Mantenibile
* Leggibile
* Meno incline/propenso agli errori

inoltre:

* Dare il nome corretto e significativo alle cose
* rimanere semplice il più possibile
* creare uno o più pattern che inducano familiarità

cercando quindi di:

* fare ciò che realmente serve
* limitare le funzionalità e le responsabilità
* individuare i confini del dominio
* stabilire sistemi di comunicazione (anche tra le persone!)

quello che ottengo è:

* affidabilità
* portabilità
* riutilizzabile
* testabile
* componibile

vedi anche principi: SOLID, KISS, YAGNI (ya ain't gonna need it), DRY, Basso accoppiamento, alta coesione.

## Mutazioni genetiche? No di stato!

Prendiamo un esempio tratto dalla seguente [guida](https://mostly-adequate.gitbooks.io/mostly-adequate-guide/content/ch01.html#a-brief-encounter), in cui rappresentiamo con una classe uno stormo di gabbiani la cui dimensione, dello stormo, cambia a seconda se si uniscano ad altri o si riproducano (conjoin e breed):

```js
class Flock {
  constructor(n) {
    this.seagulls = n;
  }

  conjoin(other) {
    console.log('Conjoin: ', other.seagulls, this.seagulls)
    this.seagulls += other.seagulls;
    return this;
  }

  breed(other) {
    console.log('Breed:', other.seagulls, this.seagulls)
    this.seagulls = this.seagulls * other.seagulls;
    return this;
  }
}

const flockA = new Flock(4);
const flockB = new Flock(2);
const flockC = new Flock(0);
const result = flockA
  .conjoin(flockC) // 0 + 4 = 4
  .breed(flockB) // 2 * 4 = 8
  .conjoin(flockA.breed(flockB)/* 2*8=16 */) /* 16 + 16 = 32 in quanto ripasso flockA e la somma a se stessa*/
  .seagulls;

console.log('Result: ', result)
/*
Quello che volevo:
conjoin flockA e flockC: 4 + 0 = 4
breed flockB e il cojoin precedente = 4 * 2 = 8
breed flockA e flockB = 4 * 2 = 8
cojoin tra i breed precedenti = 8 + 8 = 16

Quello che ottengo:
Faccio un conjoin con un flockC 0
Faccio un breed con un flockB di 2 ora ho uno stormo di 8
Faccio un conjoin tra uno stormo che viene calcolato a partire dall'attuale flockA e un breed con un flockB che è 16
ed il conjoin finale si fa tra questo 16 calcolato e flockA che è 16 e pertanto ho 32
*/
/*

*/
```

Lo scopo è quello di osservare la difficoltà che ho dovuto mettere per iscritto nei commenti per capire come il flow dell'applicazione muti lo stato interno della mia classe.
Con un approccio a funzioni anziché a classi avrei avuto:

```js
const conjoin = (flockX, flockY) => flockX + flockY;
const breed = (flockX, flockY) => flockX * flockY;

const flockA = 4;
const flockB = 2;
const flockC = 0;
const result =
    conjoin(breed(flockB, conjoin(flockA, flockC)), breed(flockA, flockB));
```

Il risultato è quello sperato, ma abbiamo ancora confusione dovuto al fatto di avere funzioni annidate. Quello che abbiamo fatto in realtà è stato richiamare semplici funzioni di addizione e moltiplicazione.

Questo ci porta a riprendere alcune proprietà matematiche di base:

```
// proprietà associativa
add(add(x, y), z) === add(x, add(y, z));

// proprietà commutativa
add(x, y) === add(y, x);

// identità
add(x, 0) === x;

// proprietà distributiva della moltiplicazione
multiply(x, add(y,z)) === add(multiply(x, y), multiply(x, z));
```

Ora riprendiamo il nostro esempio con funzioni e sostituiamo i nomi con `add` e `multiply`:

```js
const add = (flockX, flockY) => flockX + flockY;
const multiply = (flockX, flockY) => flockX * flockY;

const flockA = 4;
const flockB = 2;
const flockC = 0;
const result =
    add(multiply(flockB, add(flockA, flockC)), multiply(flockA, flockB));
```

e sostituiamo a seconda della proprietà:

```
add(flockA, flockC) = identità

add(multiply(flockB, identità, multiply(flockA, flockB)) = distributiva

ottengo:
multiply(flockB, add(identità, z))

ora sostituisco identità con flockA e z con flockA e non flockB! Attenzione, flockB qui è la x.
```

ed ecco la mia nuova chiamata:

```js
const result = multiply(flockB, add(flockA, flockA));
```

Il punto di questo esempio è che è possibile scrivere programmi concisi e leggibili sfruttando semplici proprietà matematiche. Ciò richiederà un pò di disciplina e approcciarsi in maniera differente evitando il "tutto va bene" della programmazione imperativa.

## Function as First Class Citizen

Probabilmente siamo abituati ad utilizzare le funzioni come dati, sia in qualità di argomenti che di risultato di una esecuzione. La chiave di comprensione è che una funzione, se usata male, aggiunge uno strato indiretto che complica anziché aggiungere valore al codice. Siamo abituati a invocare le funzioni e tornare dati, ma un altro modo di usare le funzioni è non invocarle:

```js
const func = msg => console.log(msg)
func('ciao')
```

equivale a:

```js
const func = console.log
func('ciao')
```

Vediamo un altro esempio

```js
const ajaxCall = fn => arg => fn(arg);
const callback = () => console.log('asd');

const getServerStuff = callback => ajaxCall(json => callback(json));
getServerStuff(callback)();
```

equivale a:

```js
const getServerStuff = ajaxCall;
getServerStuff2(callback)();
```

## Programmazione Funzionale

La Programmazione Funzionale si basa su alcuni concetti chiave che vedrò più avanti, per ora la più semplice definzione è che la PF fa uso di... funzioni e comunque predilige quest'ultime per controllare il flusso della mia applicazione.
Per prima cosa, sappi che ogni funzione è una coppia distinta input,output cioè una mappatura di un valore di ingresso in un valore di uscita. In maniera deterministica quindi ho un Dominio di Ingresso e un Insieme di possibili valori associati: 1 Input - 1 Output.

```
const toRomanNumbers = {"1":"I", "2": "II", "3": "III", "4": "IV", "5": "V" }
```

questo vuol dire che per lo stesso input, avrò sempre lo stesso output. Sempre!

Un'altra cosa che posso osservare è che ogni input ho sempre un valore, cioè è "totale" ovvero ogni elemento di dominio è associato ad un valore di uscita.

Infine, una funzione del "mondo del funzionale" non deve creare effetti collaterali, cioè non fa side-effects. Cosa significa? Un effetto collaterale è qualcosa di indesiderato e che insiste su un "sistema esterno" alla funzione, e che non concorre per l'ottenimento dell'output. Un ottimo esempio è la differenza tra due funzioni simili, una in cui ho una console.log e una che ritorna il messaggio di log come parte di una struttura dati. La prima fa side effect, la seconda no. E perché mai una console.log è side effects? In realtà per side effects è una qualsiasi variazione del mondo esterno, giusto? Anche la variazione di un testo di log nel monitor è considerato un side effect di qualcosa di esterno che non concorre per la generazione del mio output.

```js
const somma_con_se = (x, y) => {
  console.log(`Aggiungo ${x} a ${y}`);
  return x + y;
}

const somma_senza_se = (x, y) => {
  return {risultato: x + y, log: `Aggiungo ${x} a ${y}`};
}
```

Pertanto se vado a modificare il mio input, faccio side effect. Farei side effect anche se lanciassi eccezioni in caso di errore. E se dovessi ad esempio salvare un dato nel db? Anziché farlo nella funzione posso tornare una funzione che eseguita salverà il dato:

```js
const signUp = (attrs) => {
  return () => {
    const user = saveUser(attrs);
    welcomeUser(user);
  }
}
```

Sai dirmi due metodi degli Array simili ma che si comportano in maniera differente in termini di Side Effect?

* Splice fa side effect
* Slice non fa side effect

Esempi:

```js
const compleanno = utente => {
  utente.eta += 1;
  return utente;
}
// fa side effect, muta l'utente, ripasso l'utente ho un output sempre diverso, ma l'utente è sempre quello

const esclama = parola => parola.toUpperCase().concat('!');
// non fa side effect, concat non mi muta la stringa di ingresso

const testoIntestazione = header_selector => querySelector(header_selector).text();
// non muto l'ingresso, però quello che succede è che prendo il valore dal "mondo esterno" e quindi potrei non avere questa proprietà deterministica della funzione, e quindi...fa side effect

const parseQuery = () => location.search.substring(1).split('&').map(x => x.split('='));
// eeehhh??? Ok ok la risposta è semplice, se la location cambia l'output cambia quindi stesso input potrei avere output diverso
// se avessi passato direttamente il search?
const parseQuery = (search) => search.substring(1).split('&').map(x => x.split('='));
// parseQuery('login.php&user=Lorenzo&role=admin') il risultato è [ [ 'ogin.php' ], [ 'user', 'Lorenzo' ], [ 'role', 'admin' ] ]
// parseQuery('login.php&user=Lorenzo&role=admin') il risultato è [ [ 'ogin.php' ], [ 'user', 'Lorenzo' ], [ 'role', 'admin' ] ]

// non fa side effect perché: substring non modifica search, split mi crea suddivide la stringa in 2 array e la map mi ritorna un array, ma nessuno modifica search!
```

Ovviamente le modifiche interne alla funzione sono accettate e non generano side effect.

Questo concetto di stesso input, stesso output e non modifica del mondo esterno mi porta al termine di funzioni PURE e funzioni IMPURE.

Un altro concetto alla base di ciò che andrò a studiare più avanti sono le **Funzioni di Ordine Superiore** o High Order Function e cioè funzioni che accettano come parametri di ingresso funzioni o che restituiscono funzioni.

Quali HoF conosci? Pensa agli Array. Le funzioni come: map, filter e reduce accettano delle funzioni dette predicato che applicano, il predicato, all'i-esinmo elemento dell'Array:

```js
const array = [1, 2, 3];
const double = val => val * 2;
const doubled = array.map(double); // ottengo  [2, 4, 6]
const isOdd = val => val % 2 !== 0;
const odds = array.filter(isOdd);
const evens = array.filter(!isOdd); // sbem: errore!!! false is not a function
const evens = array.filter(x => !isOdd(x));
```

Quello che succede con le HoF è una trasformazione, una valutazione di un predicato o una riduzione o accumulazione. Prendo un flusso e ottengo un altro flusso lavorato, sintetizzato, collassatto.

Ho un problema: mi vengono ritornati due numeri sotto forma di array: [2,3].
Ho una funzione che somma due numeri dati come singoli parametri: const somma = (x, y) => x + y;
come posso fare in modo che posso passare a somma direttamente l'array senza modificare la signature di somma?

```js
const wrapper = func => ([x, y]) => func(x, y);
wrapper(add)([2,3]);
```

Tra le funzioni, quella che è interessante vedere è le funzioni **Curry** o **Curried Function**, cioè la possibilità di creare funzioni che accettano un solo argomento.
Le curried function sono il comportamento di default nel linguaggio Haskell, dove, di base, tutte le funzioni accettano un parametro e possiamo ricondurre tutte le funzioni di n parametri a funzioni di 1 parametro. Ecco perché in Haskell trovo qualcosa di simile a:

```h
/* funzione che prende due parametri di tipo a e ritorna un valore di tipo a */
a -> a -> a
/* funzione che prende due parametri: una funzione e un valore a e che ritorna un valore di tipo a. La funzione passata per parametro prende a sua volta un parametro di tipo a */
(a -> a) -> a -> a
```

La particolarità sta in questa **applicazione parziale** che mi crea di fatto delle funzioni on-the-fly e nel poter memorizzare una computazione di funzione e usarla più volte, durante l'esecuzione del mio codice, mettendo in atto la tecnica di Memoizzazione (memoization, cioè come se, una volta calcolato un valore, me lo appuntassi su un post-it).
Posso creare la mia semplice curry che trasforma funzioni con 2 argormenti in singoli argomenti:

```js
const add = (x, y) => x + y;

const curry = func => x => y => func(x, y);

const curriedAdd = curry(add);

const addOne = curriedAdd(1);

const addTwo = curriedAdd(2);

addOne(addTwo(2));
```

Posso definire una funzione di uncurry:

```js
const uncurry = func => (x, y) => func(x, y);

uncurry(addOne)(2);
```

ma `addOne` è `curriedAdd(1)` che è `curry(add)(1)` e pertanto posso scrivere `uncurry(curry(add)(1))(2)`.

Posso fare la curry di qualsiasi funzione con qualasi numero di parametri e trasformarla in una versione curry a singolo parametro. Questo processo è detto **Currying**.

Proviamo a pensarla. Per fare la curry di una funzione con n parametri, quello che faccio è passare alla mia curry la funzione e gli n parametri. Questi indicano l'**arietà** della funzione, cioè il numero di parametri.

```js
const curry = fn => {
  const arieta = fn.length;
}
```

adesso ritorno una funzione anonima che accetta un parametro:

```js
const curry = fn => {
  const arieta = fn.length;
  return param => { }
}
```

cosa devo fare adesso? Diciamo che devo ricorsivamente ritornare questa funzione che accetta parametri fino al caso base in cui l'arietà sia <= 1, in questo caso devo eseguire la mia fn:

```js
const curry = fn => {
  const arieta = fn.length;
  return param => {
    if (arieta <= 1) return fn(param);
  }
}
```

ed ora il passo ricorsivo, ma prima devo passare ad ogni step il decremento della mia arietà:

```js
const curry = (fn, n) => {
  const arieta = n || fn.length;
  return param => {
    if (arieta <= 1) return fn(param);
  }
}
```

ed ecco il passo ricorsivo:

```js
curry((...rest) => fn(param, ...rest), arieta-1);
```

e la mia curry finale:

```js
const curry = (fn, n) => {
  const arieta = n || fn.length;
  return param => {
    if (arieta <= 1) return fn(param);
    else return curry((...rest) => fn(param, ...rest), arieta-1);
  }
}
```

e l'utilizzo:

```js
const add = (x, y, z) => x + y + z;

curry(add)(2)(3)(4)
```


Vediamo un altro esempio:

```js
const modulo = x => y => y % 2 !== x

const isDispari = modulo(0);
const isPari = modulo(1);
const filter = f => array => array.filter(f);

const getDispari = filter(isDispari);
const getPari = filter(isPari);
getDispari([0,1,2,3,4,5,6,7,8,9,10]);
getPari([0,1,2,3,4,5,6,7,8,9,10]);
```

e se avessi avuto questo codice, come scrivo la getPari sfruttando solo la isDispari:

```js
const modulo = x => y => y % x
const isDispari = modulo(2);
const filter = f => array => array.filter(f);
const getDispari = filter(isDispari);
```

SOL: `const getPari = filter(x => !isDispari(x));`

# L'uso di funzioni negli oggetti letterali

Vediamo che posso creare facilmente un oggetto letterale e un metodo che posso utilizzare per ottenere il nome completo di una persona:

const utente = {nome: 'Lorenzo', cognome: 'Franceschini'};
const nomeCompleto = (nome, cognome) => [nome, cognome].join(' ');

nomeCompleto(utente.nome, utente.cognome);

e in maniera più generica:

const unisciStringheConSpazio = (...args) => args.join(' ');

sapendo che il rest operator mi da proprio un array su cui posso fare il mio join:

unisciStringheConSpazio(utente.nome, utente.cognome);
unisciStringheConSpazio('Sig.', unisciStringheConSpazio(utente.nome, utente.cognome));

come vedi in maniera generica posso creare composizioni di funzioni e l'ordine non è importante.

Ricordi la somma di 2, anzi 3 o più numeri: (1+2)+3 = 1+(2+3) = (1+3)+2 ? Ecco l'addizione è associativa:

somma(somma(x, y), z) == somma(x, somma(y, z))

una funzione è commutativa quando posso scambiare l'ordine:

somma(x, y) == somma(y, x)

e posso trovare un valore identità per cui quella somma di x torna proprio x:

somma(x, identity) = x

e per la somma l'identità chi è? E' il valore 0: somma(x, 0) = x


