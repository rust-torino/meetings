# Streaming iterators
Proviamo a definire un tratto `StreamingIterator` in base alle esigenze evidenziate fino ad ora:
* Si devono ottenere riferimenti mutabili
* La _lifetime_ dell'oggetto ottenuto deve essere collegata all'iteratore stesso

Ecco una possible elementare definizione:
```rust
trait StreamingIterator {
  type Item;

  fn next(&mut self) -> Option<&mut Self::Item>;
}

#fn main() {}
```
