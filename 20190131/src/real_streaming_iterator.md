# Il vero StreamingIterator
Proviamo con il solito esempio semplice: proviamo ad implementare uno `StreamingIterator` per una slice mutabile.

```rust
trait SliceMutStreamingIterExt<T> {
  fn streaming_iter_mut(&mut self) -> StreamIterMut<'_, T>;
}

struct StreamIterMut<'a, T> {
  iter: std::slice::IterMut<'a, T>,
  item: Option<&'a mut T>,
}

impl<T> SliceMutStreamingIterExt<T> for [T] {
  fn streaming_iter_mut(&mut self) -> StreamIterMut<'_, T> {
    StreamIterMut {
      iter: self.iter_mut(),
      item: None,
    } 
  }
}
#fn main() {}
```

Ok, adesso proviamo ad implementare `Iterator`:
```rust
# trait SliceMutStreamingIterExt<T> {
#   fn streaming_iter_mut(&mut self) -> StreamIterMut<'_, T>;
# }
# 
# struct StreamIterMut<'a, T> {
#   iter: std::slice::IterMut<'a, T>,
#   item: Option<&'a mut T>,
# }
# 
# impl<T> SliceMutStreamingIterExt<T> for [T] {
#   fn streaming_iter_mut(&mut self) -> StreamIterMut<'_, T> {
#     StreamIterMut {
#       iter: self.iter_mut(),
#       item: None,
#     } 
#   }
# }
#
impl<'a, T> Iterator for StreamIterMut<'a, T> {
  for <'self> type Item = &'self mut &'a mut T;

  fn next(&mut self) -> Option<Self::Item> {
    self.item = self.iter.next();
    self.item.as_mut()
  }
}
#fn main() {}
```

Ohi ohi. Ecco il problema. La sintassi che stiamo usando, paragonabile a quella degli Higher-Rank Trait Bound (HRTB), non esiste.
Ciò che stiamo cercando di scrivere è un tipo generico associato (GAT), una feature con tracking issue aperta da settembre 2017.
