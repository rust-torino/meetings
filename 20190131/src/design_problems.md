# Problemi di design

Osservando bene il codice (sotto è presente nuovamente) è possibile notare che abbiamo utilizzato un riferimento mutabile ad `Item` in modo tale da _agganciare_ la vita dell'oggetto ritornato a quella dell'iteratore stesso.
Tuttavia, da ciò che abbiamo detto, è anche necessario che `Item` sia un riferimento mutabile affinché la cosa abbia senso.
Questo comporta uno scomodo doppio livello di indirezione.

Per rendere il tutto più chiaro facciamo un esempio:
```rust
trait StreamingIterator {
  type Item;

  fn next(&mut self) -> Option<&mut Self::Item>;
}

struct SliceIterMut<'a, T> {
  iter: std::slice::IterMut<'a, T>,
  cur: Option<&'a mut T>,
}

impl<'a, T> StreamingIterator for SliceIterMut<'a, T> {
  type Item = &'a mut T;

  fn next(&mut self) -> Option<&mut &'a mut T> {
    self.cur = self.iter.next(); 
    self.cur.as_mut()
  }
}

#fn main() {}
```

L'approccio funziona, ma non è certamente elegantissimo...
