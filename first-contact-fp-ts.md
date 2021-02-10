# First Contact con FP-TS

[Disclaimer]: tutto ciò è frutto di studi e appunti ed è sicuramente soggetto ad errori parziali, totali o catastrofi, quindi prendere tutto con le pinze 🙅🏻‍♂️. Ogni tanto proverò ad inserire qualche spunto preso dalla Teoria delle Categorie.

FP-TS è una [libreria](https://github.com/gcanti/fp-ts) scritta da Giulio Canti per scrivere programmazione funzionale tipizzata in JavaScript utilizzando il superset TypeScript.
Grazie a FP-TS possiamo fare asserzioni riguardanti le nostre strutture dati, inoltre ci fornisce i costrutti per esprimere un modello di codice come una sequenza che può anche fallire. Infine, unito a TypeScript abbiamo tutti i vantaggi di un codice Type Safe con una serie di problematiche catturate a compile time e codice più auto documentato.

Lo scopo di FP-TS è quello di permettere ai developers di utilizzare i principali patterns e astrazioni presenti nei linguaggi funzionali. Include i data types più popolari, le type classes e le astrazioni come **Option**, **Either**, **IO**, **Task**, **Functor**, **Applicative**, **Monad**.

FP-TS support gli Higher Kinded Types 🤯. Un **Functor** è un HKT, una sorta di astrazione rispetto ad un tipo astratto 💀.

Partendo dal principio, possiamo definire che i valori sono informazioni/dati che il nostro programma ha elaborato o che noi abbiamo impostato e assegnato alle nostre variabili. Possibili valori sono: "Hello", 100, true, 3.14, 'c'. Ognuno di essi appartiene ad un tipo: String, int, boolean, float, char (in Java).
I tipi specificano (a compile-time per i linguaggi statically-typed e a run-time per i linguaggi dynamically-typed) i vincoli sui valori che una espressione può assumere a run-time. Questi li definiamo con **tipi concreti**.
Abbiamo anche i generici/template, cioè tipi che dobbiamo definire tramite un parametro T: Array<T>, quindi a partire da un tipo concreto.
Qui viene il concetto di **type constructor** cioè una funzione che prende in input due tipi e torna un nuovo tipo, proprio come l'Array<T> prende l'Array, prende il T e produrrà il nuovo tipo Persone.
Siamo abituati a conoscere il concetto di **value constructor** meglio noti come **funzioni** e **metodi**.
Ora prendiamo il concetto di funzionalità/comportamento. Potrei definire delle funzionalità di uguaglianza (eq), di ordinamento (ord), di visualizzazione (show), ecc... queste mi definiscono un **TypeClass** ed è un concetto simile ai Traits di Scala e alle Interfacce di Java/TypeScript. Sono comportamenti che ritrovo in quei tipi appartenenti a quella type class. Per fare un paragone è come se avessi degli Archetipi di classe di un gioco RPG e delle Specializzazioni su quegli Archetipi o almeno credo 😅.

## Un primo incontro: Pipe e Flow

Utilizzando RunJS o qualsiasi altro compilatore online/editor locale, installiamo la libreria **fp-ts** `npm i fp-ts`:

```js
import { pipe, flow } from 'fp-ts/function';

function double(a: string): string {
  return a + a;
}

function length(a: string): number {
  return a.length;
}

const doubleLength = flow(double, length);

const result0 = length(double('foo'));
const result1 = pipe('foo', double, length);
const result2 = doubleLength('foo');
```

Qui vediamo che ottengo lo stesso risultato tramite una composizione di funzione e con FP-TS, con l'utilizzo di pipe, vado a simulare la sintassi del pipe operator `foo |> double |> length`, mentre con flow vado a comporre una mia terza funzione a partire da due funzioni.

L'operatore **pipe** ci permette quindi di concatenare sequenze di funzioni da sinistra a destra. La composizione in Teoria delle Categorie consiste in una sequenza di funzioni da destra verso sinistra (g°f significa g dopo f). La **composizione** è alla base della Teoria delle Categorie ed è l'essenza di una Categoria o, l'essenza della composizione è la Categoria.

> Una categoria consiste in oggetti e frecce che vanno tra loro. Una freccia parte da un oggetto e finisce in un altro oggetto o al medesimo oggetto di partenza. Le frecce sono dette morfismi e, nel caso della programmazione, sono riconducibli a delle funzioni. Gli oggetti sono Tipi.

L'operatore pipe prende:

* un valore iniziale arbitrario che sarà il primo argomento della funzione che lo segue;
* un numero n di funzioni con arietà unaria (singolo argomento) ed è per questo che nel codice ho scritto funzioni con singoli argomenti.

```ts
import { pipe, flow } from 'fp-ts/function';

// add è volutamente sovrastrutturata di tipi, vedi multiply per una gestione del tipo più snella che sfrutta l'inferenza di tipo
const add: (x: number) => (y: number) => number = (x: number) => (y: number) => x + y;

const multiply = (x: number) => (y: number) => x * y;

const add2 = (x: number): number => add(x)(2);
const multiply2 = (x: number): number=> multiply(x)(2);

// Anche qui potrei omettere il tipo e usare l'inferenza di tipo
const convertToString: (x: number) => string = (x: number) => `il valore ottenuto è ${x}`;

const res = pipe(10,add2, multiply2, convertToString);
```

Pertanto con pipe posso modellare facilmente una sequenza, l'importante è che il tipo finale sia compatibile con il tipo iniziale della funzione successiva. Questo è proprio il concetto che trovo tra elementi e frecce:

```
f :: A -> B
g :: B -> C
h :: C -> D

h ° (g ° f) = ( h ° g) ° f = h ° g ° f
```

L'operatore **flow** è analogo al pipe, con la differenza che il primo argomento sarà una funzione e non un valore arbitrario:

```ts
// equivalenti
// 1.
const res = pipe(10, flow(add2, multiply2, convertToString));
// 2.
const res = flow(add2, multiply2, convertToString)(10);
// 3.
const res = flow(add2, multiply2, convertToString);
res(10);
```

Se analizziamo le sequenze di pipe e flow ho uno schema del tipo:

```
v -> A -> B -> C -> D ...
A -> B -> C -> D ...
```

Vediamo un esempio in cui la flow è sicuramente da preverire a pipe. Supponiamo di avere una funzione di concatenazione con arietà 2 e che prende un numero e una funzione di trasformazione ,che a sua volta partendo da un numero restituisce una rappresentazione in stringa. Quello che ottengo sarà una tupla `[number, string]` e la funzione di trasformazione sarà proprio le due sequenze viste prima:

```ts
const concat = (
  // ho 2 parametri di ingresso
  a: number,
  transformer: (a: number) => string,
): [number, string] =>
  // prendo il valore a e lo passo alla mia sequenza:
  [a, transformer(a)];
```

ed ecco come la userò:

```ts
concat(10, (n) => pipe(n, add2, multiply2, convertToString)); // [10, 'il valore ottenuto è 6']
concat(10, flow(add2, multiply2, convertToString)); // [10, 'il valore ottenuto è 6']
```

Qui il parametro `n` deve far parte della mia funzione anonima e in generale è da evitare perché potrei fare "shadowing" di una variabile con lo stesso nome nell'outer scope. Inoltre è prolisso. Con la flow non ho bisogno della funzione anonima in quanto la flow ritorna una funzione lei stessa e pertanto `transformer(a)` è come se fosse `flow(...)(a)`.

## Prodotto e Somma di Tipi

Supponiamo ora di avere una specifica che preveda la costruzione di uno schema per il noleggio di una autovettura. Definiamo quindi delle regole e dei vincoli tali per cui devo tenere traccia di:

* tipo di pagamento: "onsite" - "online"
* email: solo se il pagamento è "online"
* age range: "18-24" "25-29" 30+

Definisco il mio DOMINIO con i valori possibili e analizzo i payload (validi, parzialmente validi e invalidi) che posso ottenere.

Il numero delle possibilità che ho di ricevere diversi payload è dato dal prodotto di possibilità che ho per singolo tipo. Esempio: supponiamo che ho un payload che riceve 3 valori: 2 boolean e 1 digit (da 0 a 9), ho pertanto: 2(true/false) x 2(true/false) x 10 (da 0..9) = 40 payload possibili.
Tornando al nostro caso potrei avere dei payload parzialmente invalidi, come nel caso di pagamento "onsite" e avere l'"email" o pagamento "online" e non avere l'"email".

In questo caso avrei 2 possibilità per la modalità di pagamento, 3 possibilità per l'age range e infinite possibilità per l'email più il caso di undefined nel momento in cui questa non è richiesta.

```ts
type PaymentMode = 'onsite' | 'online';
type AgeRange = '18-24' | '25-29' | '30+';

type FormType = {
  paymentMode?: PaymentMode;
  ageRange?: AgeRange;
  email?: string;
};
```

Definisco degli optional in quanto l'utente inzialmente dovrà compilare la form e non ho valori.
Definisco il tipo della mia API:

```ts
type ConfirmRent = {
  paymentMode: PaymentMode;
  ageRange: AgeRange;
  email?: string;
}
```

Ed ora vediamo alcuni possibili payload accettati della nostra API:

```ts
const payload1: ConfirmRent = {
  paymentMode: 'onsite'
  ageRange: '30+'
}
// parzialmente errato, funziona ma l'email non era richiesta
const payload3: ConfirmRent = {
  paymentMode: 'onsite'
  ageRange: '30+'
  email: 'email@email.com;
}
// completamente errato, manca un parametro obbligatorio come l'email
const payload4: ConfirmRent = {
  paymentMode: 'online'
  ageRange: '18-24'
}
```

Pertanto con la rappresentazione dei miei possibili stati tramite il prodotto degli stati parziali succede che posso ricevere stati incosistenti o invalidi.

Vediamo quindi un'altra rappresentazione.

In precedenza avevamo definito il nostro `ConfirmRent` come:

```ts
type ConfirmRent = {
  paymentMode: PaymentMode;
  ageRange: AgeRange;
  email?: string;
}
```

mentre ora lo definiamo come:

```ts
type OnlinePayment = {
  paymentMode: "online";
  ageRange: AgeRange;
  email: string;
}

type OnsitePayment = {
  paymentMode: "onsite";
  ageRange: AgeRange;
}

type ConfirmRent = OnlinePayment | OnsitePayment;
```

In questo modo ho una discriminante che mi notifica a compile time eventuali payload errati.
Ho sempre valori infiniti per la mia stringa, ma andiamo a restringere i casi possibili di payload by design grazie alla definizione del nuovo tipo tramite la somma dei tipi rappresentabili.
In [TypeScript](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html) questo viene rappresentato tramite le **Discriminated Union** in cui ho una proprietà letterale che mi definisce la "differenza", il discriminante, tra un tipo e l'altro in una unione:

```ts
type NetworkLoadingState = {
  state: "loading";
};

type NetworkFailedState = {
  state: "failed";
  code: number;
};

type NetworkSuccessState = {
  state: "success";
  response: {
    title: string;
    duration: number;
    summary: string;
  };
};

// Create a type which represents only one of the above types
// but you aren't sure which it is yet.
type NetworkState =
  | NetworkLoadingState
  | NetworkFailedState
  | NetworkSuccessState;
```

Gestirò il dato tramite una `switch` sul discriminante, in cui nelle `case` posso accedere alle proprietà relative ad uno specifico tipo.

```ts
function rentConfirmation(rent: ConfirmRent): string {
  switch(rent.paymentMode) {
    case 'online':
      return `La tua prenotazione è stata confermata all'indirizzo ${rent.email}`;
    case 'onsite':
      return `La tua prenotazione è confermata, pagherai al pickup`
  }
}
```

## Fold

Vediamo ora la funzione `fold`, una funzione generica che:

* accetta un parametro `match` che conterrà i miei possibili "stati" di pagamento: online e onsite
* ritorna una funzione che prende una conferma di prenotazione e ritorna il tipo generico

In questo modo, a seconda del mio stato, posso mappare il valore di ritorno, che può essere una stringa:

```ts
function fold<R>(match: {
  online: (onlinePayment: OnlinePayment) => R,
  onsite: (onsitePayment: OnsitePayment) => R
}): (rentSubmit: ConfirmRent) => R {
  return rentSubmit => {
    switch (rentSubmit.paymentMode) {
      case 'online':
        return match.online(rentSubmit);
      case 'onsite':
        return match.onsite(rentSubmit);
    }
  }
}

const payload1: ConfirmRent = {
  paymentMode: 'onsite',
  ageRange: '30+'
}
const payload2: ConfirmRent = {
  paymentMode: 'online',
  email: 'lorenzo@gmail.com',
  ageRange: '30+'
}

const rent = fold<string>({
  online: (onlinePayment: OnlinePayment) => `you pay online with email ${onlinePayment.email}`,
  onsite: (onlinePayment: OnsitePayment) => 'you pay on pickup',
});

const res1 = rent(payload1);
const res2 = rent(payload2);
```

Hai capito a cosa serve la **fold**? Praticamente ci permette di fare **Pattern Matching** cioè sostituire quei costrutti che facevi con **if/elseif/else** o **switch**.
Vediamo un esempio più semplice che "mima" l'handler di una richiesta di SignIn:

```ts
import { fold } from "fp-ts/lib/Either";

const signIn = (username: string, password: string) => {
  return (username ==='pippo' && password ==='iNks')
    ? true
    : throw new Error("bad credentials");;
}

const replyUnauthorized = (res: any) => {return (error) => `Non sei autorizzato: ${error}`; }

const replyToken = (res: any) => {return () => `Token generato`; }

const res = {
  body: {
    username: 'pippo',
    password: 'iNks'
  }
};

const {username, password} = res.body;


fold(replyUnauthorized(res), replyToken(res))(signIn(username, password))
```

Per capire meglio questo esempio risciviamo la signIn con **Either, Right e Left**:

```ts
import { right, left, Either } from "fp-ts/lib/Either"

const signIn = (username: string, password: string): Either<error, boolean> => {
  return (username ==='pippo' && password ==='inps')
    ? right(true)
    : left(new Error("bad credentials"))
}
```

## Option

Vediamo la definizione di [Option](https://gcanti.github.io/fp-ts/modules/Option.ts.html):

```ts
type Option<A> = None | Some<A>
```

è un [Container](./container.md) cioè un involucro che può contenere un valore di tipo A:

* una instanza di None
* una istanza di Some<A> che contiene il valore di tipo A
