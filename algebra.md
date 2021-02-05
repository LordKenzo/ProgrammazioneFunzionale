# Concetti di base

In questo documento, vado a riassumere alcuni concetti di base che definiscono cos'è:

* Struttura Algebrica;
* Operazione binaria;
* Magma o Grupoid o Quasi Group;
* Loop;
* Semigroup;
* Monoid;
* Group.

Consideriamo un insieme S non vuoto.
Consideriamo * una operazione e generalmente si intende una operazione binaria una qualunque applicazione `f` sull'insieme `S` definita dal prodotto cartesiano `SxS` e che ritorna un valore `v` appartenente ancora ad `S`.

## Struttura algebrica

E' un insieme ordinato o tupla ordinata in cui ho un insieme S non vuoto e un numero di operazioni n binarie definite per tutti gli elementi di S:

In generale:

* Ho un insieme S non vuoto spesso anche detto Dominio di elementi dello stesso tipo;
* una collezione di operazioni sugli elementi con arietà finita (tipicamente binaria);
* un insieme finito di assiomi che devono essere soddisfatti dalle operazioni.

Un Algebra è lo studio delle strutture algebriche.

## Operazione Binaria

Dato un insieme `S` non vuoto allora ho una funzioni che indico con `*` tale per cui ho che il prodotto cartesiano nell'insieme `S` produca un valore dell'insieme `S`:

SxS -> S
Per ogni a,b appartenenti a S allora a*b apparterrà ancora ad S

## Magma, Grupoid o Quasi Group

Dato un insieme `S` non vuoto e `*` una operazione binaria su `S` l'applicazione di questa operazione binaria mi restituisce un risultato ancora appartenente all'insieme dei valori ammissibli in S, questa proprietà è detta **Closure** o Chiusa e la struttura (S, *) è una struttura algebrica chiamata Magma/Grupoid o Quasi Group e sarà il nostro punto di partenza.

## Loop

A partire da un Quasi Group, se per ogni `a` appartenente ad `S`, esiste `e` appartenente ad `S` tale che `a*e=e=e*a` ho l'identità, cioè l'**esistenza di un elemento identità** ed il Grupoid è detto Loop.

## Semigroup

A partire da un Quasi Group, la mia struttura algebrica (S, *) con `S` insieme non vuoto e `*` operazione binaria allora `(a*b)*c=a*(b*c)` per ogni `a,b,c` appartenenti ad `S` in cui l'operazione binaria soddisfa la **proprietà associativa**, quindi la struttura la struttura algebrica soddisfa questa proprietà ed è detta Semigroup.

## Monoid

A partire da un Semigroup in cui ho una operazione `*` che soddisfa la proprietà presente in Magma della chiusura, la proprietà associativa presente nel Semigroup, soddisfi inoltre l'esistenza di un elemento `e` appartenente ad `S` tale che per ogni elemento `a` di S ho che `a*e=e*a=a` quindi esiste un `e` di `S` chiamato **identità**.

Tra Loop e Monoid la differenza è che quest'ultimo è anche Semigroup e quindi soddisfa la proprietà associativa, cosa che non è richiesta nel Loop.

## Group

Dato un insieme S non vuoto e una operazione binaria `*` in cui il prodotto cartesiano `SxS->S` e quindi a partire da un Magma/Grupoid o Quasi Group, ho che la struttura algebrica `(S, *)` deve soddisfare 4 proprietà:

* Per ogni `a,b` appartenenti ad `S` allora `a*b` appartiene ancora ad `S` (Closure);
* `a*(b*c) = a*(b*c)` per ogni `a,b,c` appartenenti ad `S` (Associativa);
* Per ogni `e` appartenente ad `S` tale che `a*e=a=e*a` (Esistenza Identità);

e la quarta:

* Per ogni `a` appartenente ad `S`, esiste `b` appartenente ad `S` tale che `a*b=e=b*a` e cioè l'esistenza di un inverso per ogni elemento di `S`.

Quindi il Group è un Monoide in cui ogni elemento ha un inverso.