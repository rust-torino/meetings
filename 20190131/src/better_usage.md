# Usiamolo meglio!
Vediamo di implementare un `for_each`, così possiamo migliorare l'usabilità.

```rust
trait StreamingIterator {
#  type Item;
#
#  fn next(&mut self) -> Option<&mut Self::Item>;
  /* ... */

  fn for_each<F>(mut self, mut f: F)
  where
    Self: Sized,
    F: FnMut(&mut Self::Item),
  {
    while let Some(item) = self.next() {
      f(item);
    }
  }
}
#
#struct SliceIterMut<'a, T> {
#  iter: std::slice::IterMut<'a, T>,
#  cur: Option<&'a mut T>,
#}
#
#impl<'a, T> StreamingIterator for SliceIterMut<'a, T> {
#  type Item = &'a mut T;
#
#  fn next(&mut self) -> Option<&mut &'a mut T> {
#    self.cur = self.iter.next(); 
#    self.cur.as_mut()
#  }
#}
#
#trait SliceIterExt<T> {
#  fn streaming_iter(&mut self) -> SliceIterMut<'_, T>;
#}
#
#impl<T> SliceIterExt<T> for [T] {
#  fn streaming_iter(&mut self) -> SliceIterMut<'_, T> {
#    SliceIterMut {
#      iter: self.iter_mut(),
#      cur: None,
#    }
#  }
#}

fn main() {
  let mut v = vec![1, 2, 3, 4, 5];

  v.streaming_iter().for_each(|x| {
    **x *= 2;
  });

  println!("{:?}", v);
}
```

Decisamente meglio! Purtroppo non è possibile utilizzare il più classico `for x in v.streaming_iter()`, poiché `SliceIterMut` non implementa `Iterator`.

Comunque la prima vera problematica consiste nel fatto che è necessario re-implementare tutte le funzionalità di `Iterator` nel tratto `StreamingIterator`.

Per fortuna esiste la crate `streaming_iterator` che ci aiuta ad avere il tratto già scritto.
