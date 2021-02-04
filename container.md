# Container

Definiamo un **container** come un contenitore di dati e metodi, metodi che lavorano sui dati salvati nel nostro contenitore.

Posso rappresentare il mio contenitore con una classe:

```js
class Container {
    constructor(data) {
        this.data = data;
    }
}

const obj = new Container(10);
```

Quello che ho costruito è semplicemente un contenitore con un costruttore e l'utilizzo di new per la creazione del mio oggetto.
A questo punto posso aggiungere alla mia classe un metodo statico per la creazione "agevolata", di comodo, senza l'uso del `new`, del mio oggetto.

```js
class Container {
    constructor(data) {
        this.data = data;
    }
    static of(data) {
        return new Container(data);
    }
}

const obj = Container.of(10);
```

Abbiamo detto che il nostro contenitore ha 2 cose fondamentali:

* la possibilità di gestire dati
* un insieme di metodi che lavorano su questi dati

aggiungiamo una terza proprietà fondamentale ed è quella di poter costruire il nostro oggetto e trasformare i dati, attraverso una **interfaccia Fluent**, e cioè la possibilità di richiamare i metodi attraverso la concatenazione (chaining).

A questo punto possiamo cominciare ad aggiungere al nostro contenitore la nostra prima funzionalità: la **map**, una funzione che preso un input (genericamente una lista) in ingresso restituisce un nuovo input su cui è stata applicata una trasformazione. L'input iniziale non viene modificato ma viene restituita una nuova forma del dato trattato.
La map è una delle prime High Order Function con cui solitamente entriami in contatto. Questo perché posso passare a map una funzione di trasformazione.

```js
const values = [1, 2, 3];
const res = values.map(n => n * 2);
// values = [1, 2, 3]
// res = [2, 4, 6]

const addTwo = x => x + 2;
console.log([1,2,3].map(addTwo)); // [3,4,5]
```

Quindi per creare questa funzionalità al mio container devo, ovviamente creare il metodo `map`, accettare un parametro che rappresenta la funzione di trasformazione applicata al mio data:

```js
class Container {
    // omissis
    map(func) {
        // apply func to my data
    }
}
```

La cosa interessante, per far si che io possa riapplicare uno dei metodi della mia classe, è ciò che ritorno dal mio metodo.

Prima ipotesi (errata):

```js
map(f) {
    this.data = f(this.data);
    return this;
}
```

Seconda ipotesi (corretta):

```js
map(f) {
    const newData = f(this.data);
    return Container.of(newData);
}
```

Questa infatti non muta il mio dato iniziale ma mi restituisce un nuovo contenitore a cui poter applicare in chaining successive trasformazioni. Nella prima ipotesi ottengo il chaining mutando la struttura di partenza.

```js
const obj = Container.of(10);
const newObj = obj.map(addTwo).map(x => x * 2);
```

In questa versione se passassi un array al posto di un intero, ottengo un valore errato.
Posso modificare velocemente il codice della mia map in questa maniera:

```js
map(f) {
    return Array.isArray(this.data)
      ? Container.of(this.data.map(f))
      : Container.of(f(this.data));
}
```

Per ora ignoriamo questa modifica e manteniamo la nostra classe semplicemente utilizzabile per valori singoli interi.

```js
class Container {

  constructor(data) {
    this.data = data;
  }
  static of(data) {
    return new Container(data);
  }
  map(f) {
      return Container.of(f(this.data));
  }
}
```

Bene, ma cosa succede ora se usassi il mio container in questo modo, cioè passando un valore `null`:

```js
const obj = Container.of(null).map(addTwo)
```

chiaro no?

```
TypeError: Cannot read property 'map' of null
```

## Maybe

il problema dei programmi è che falliscono, nel momento in cui li scriviamo in maniera fragile. Gestiamo l'eccezione in maniera specifica, quando ce lo ricordiamo e lo prevediamo. Ma come rendere questo Container migliore by design? Introduciamo il concetto di "potrebbe fallire" con il **Maybe**.

```js
class Maybe {
    constructor(data) {
        this.data = data;
    }
    static of(data) {
        return new Maybe(data);
    }
    map(func) {
        if(this.data === undefined || this.data === null) return this;
        return Maybe.of(f(this.data));
    }
}
```

In questo caso con `const obj = Maybe.of(null).map(addTwo)` il mio programma non fallisce, semplicemente il mio dato è `null`.
Posso migliorare la leggibilità della mia classe creando il metodo `isEmpty`:

```js
class Maybe {
    // omissis
    isEmpty() {
        return this.data === undefined || this.data === null;
    }
    map(func) {
        if(this.isEmpty()) return this;
        return Maybe.of(f(this.data));
    }
}
```

## Either

Con questo container realizziamo il branching del flusso di un algoritmo, tradotto in parole povere riusciamo a creare un parte then e una parte else, che si chiamano **Right** e **Left** e rappresentano rispettivamente il **successo** e l'**insuccesso** di una data operazione.

Costruisco queste strutture sulla falsa riga del Container iniziale:

```js
class Either {
    constructor(data) {
        this.data = data;
    }
    static of(data) {

    }
}

class Right extends Either {

}

class Left extends Either {

}
```

Cosa metto nell'of di Either? E cosa nelle classi Right e Left?

L'of di Either mi tornerà il caso di successo quindi una istanza di Right, mentre in Right e Left vado ad inserire le mie funzioni di map, in cui in caso di successo ritorno una istanza di Right, in caso di insuccesso ritorno l'istanza stessa che ha generato l'insucesso, in sostanza ignora la funzione passata:

```js
class Either {
    constructor(data) {
        this.data = data;
    }
    static of(data) {
        return new Right(data);
    }
}

class Right extends Either {
    map(func) {
        return Right.of(func(this.data));
    }
}

class Left extends Either {
    map(func) {
        return this;
    }
}
```

Vediamo un esempio di codice scritto senza l'utilizzo di Either, Right e Left:

```js
const body = {secret: '12324'};
const api = {secret: '1234', endpoint: '/user', data: {
  user: 'Lorenzo', age: 42
}};
const sendError = errorMsg => console.log(errorMsg);
const loadRoute = api => {
  console.log(api.endpoint);
  return api;
};
const sendData = api => {
  console.log(api.data);
  return api;
}

const accessAPI = (body, api) => {
  return (body.secret === api.secret) ? api : `L'API Key è errata: ${body.secret}`;
}

const result = accessAPI(body, api);

if(typeof result == 'string') {
  sendError(result);
} else {
  loadRoute(api);
  sendData(api);
}
```

ed ecco il codice completo con l'uso di Either:

```js
class Either {
    constructor(data) {
        this.data = data;
    }
    static of(data) {
        return new Right(data);
    }
}

class Right extends Either {
    map(func) {
        // equivalente: return Right.of(func(this.data));
        return new Right(func(this.data));
    }
}

class Left extends Either {
    map(func) {
        return this;
    }
}


// omissis

/*
Il codice ora ritorna un of di Right oppure un new Left
*/
const accessAPI = (body, api) => {
  return (body.secret === api.secret)
    ? Right.of(api)
    : new Left(`L'API Key è errata: ${body.secret}`);
}

accessAPI(body, api).map(loadRoute).map(sendData);
```

Se provassi a mettere una secret non valida non ottengo nessun messaggio. Per risolvere questo problema introduco una nuova funzione **Fold**, il suo scopo è quello di eseguire una Right o una Left a seconda se sono nel caso di successo o insuccesso.

```js
fold(left, right) {

}
```

Questa funzione verrà inserita sia in Left sia in Right:

```js
class Right extends Either {
    map(func) {
        // equivalente: return Right.of(func(this.data));
        return new Right(func(this.data));
    }
    fold(left, right) {
        right(this.data);
        return this;
    }
}

class Left extends Either {
    map(func) {
        return this;
    }
    fold(left, right) {
        left(this.data);
        return this;
    }
}
```

e userò il tutto in questo modo:

```js
accessAPI(body, api).map(loadRoute).fold(sendError, sendData);
accessAPI(body, api).fold(sendError, loadRoute).map(sendData);
```