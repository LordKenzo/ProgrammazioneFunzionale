# First Contact con FP-TS

[Disclaimer]: tutto ciò è frutto di studi e appunti ed è sicuramente soggetto ad errori parziali, totali o catastrofi, quindi prendere tutto con le pinze 🙅🏻‍♂️.

FP-TS è una [libreria](https://github.com/gcanti/fp-ts) scritta da Giulio Canti per scrivere programmazione funzionale tipizzata in JavaScript utilizzando il superset TypeScript.

Lo scopo di FP-TS è quello di permettere ai developers di utilizzare i principali patterns e astrazioni presenti nei linguaggi funzionali. Include i data types più popolari, le type classes e le astrazioni come **Option**, **Either**, **IO**, **Task**, **Functor**, **Applicative**, **Monad**.

FP-TS support gli Higher Kinded Types 🤯. Un **Functor** è un HKT, una sorta di astrazione rispetto ad un tipo astratto 💀.

Partendo dal principio, possiamo definire che i valori sono informazioni/dati che il nostro programma ha elaborato o che noi abbiamo impostato e assegnato alle nostre variabili. Possibili valori sono: "Hello", 100, true, 3.14, 'c'. Ognuno di essi appartiene ad un tipo: String, int, boolean, float, char (in Java).
I tipi specificano (a compile-time per i linguaggi statically-typed e a run-time per i linguaggi dynamically-typed) i vincoli sui valori che una espressione può assumere a run-time. Questi li definiamo con **tipi concreti**.
Abbiamo anche i generici/template, cioè tipi che dobbiamo definire tramite un parametro T: Array<T>, quindi a partire da un tipo concreto.
Qui viene il concetto di **type constructor** cioè una funzione che prende in input due tipi e torna un nuovo tipo, proprio come l'Array<T> prende l'Array, prende il T e produrrà il nuovo tipo Persone.
Siamo abituati a conoscere il concetto di **value constructor** meglio noti come **funzioni** e **metodi**.
Ora prendiamo il concetto di funzionalità/comportamento. Potrei definire delle funzionalità di uguaglianza (eq), di ordinamento (ord), di visualizzazione (show), ecc... queste mi definiscono un **TypeClass** ed è un concetto simile ai Traits di Scala e alle Interfacce di Java/TypeScript. Sono comportamenti che ritrovo in quei tipi appartenenti a quella type class. Per fare un paragone è come se avessi degli Archetipi di classe di un gioco RPG e delle Specializzazioni su quegli Archetipi o almeno credo 😅.

## Un primo incontro

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
type ConfirmRent {
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