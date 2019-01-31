# Il vero problema

Quindi possiamo usare `StreamingIterator`! Giusto?

Sbagliato, sfortunatamente. O perlomeno, potete utilizzarlo per piccoli progetti, o per zone a _scatola chiusa_ di un progetto, dove sapete che non avrete bisogno di utilizzare la funzionalità altrove.

Il motivo, purtroppo, è semplice.
Facciamo un esempio pratico: voglio scrivere una funzione che prende come input un iterazione di numeri, aggiunga "1" al numero e passi il risultato come output.

Qualcosa del tipo:
```rust
#extern crate num;
#use std::ops::Add;
#use num::One;
#
fn add_one<I>(iter: I) -> impl Iterator<Item = I::Item>
where
  I::Item: Add,
{
  iter.map(|x| x + I::Item::one())
}
#
#fn main() {}
```

Ops, questo non compila. E comunque avremmo già un problema: ritorniamo un oggetto che implementi `Iterator`.
Potremmo far sì che ritorni un implementazione di `StreamingIterator`, ma le due cose si escludono a vicenda.

Proviamo a scrivere il codice funzionante, per capire quanto la cosa sia grave:
```rust
#extern crate num;
#use std::ops::Add;
#use num::One;
#
fn add_one<I>(iter: I) -> impl Iterator<Item = <I as Iterator>::Item>
where
  I: Iterator,
  <I as Iterator>::Item: num::One + Add<Output = <I as Iterator>::Item>,
{
  iter.map(|x| x + <I as Iterator>::Item::one())
}
#
#fn main() {}
```

Ok, è evidente che `Iterator` diventa pervasivo.
L'unico modo per scrivere entrambe la funzione per entrambi i tratti sarebbe scrivere il tratto `AddOne` e implementarlo per tutto ciò che implementa `Iterator` e `StreamingIterator`...

**Aiuto**
