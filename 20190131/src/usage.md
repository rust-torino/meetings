# Usiamolo!
Proviamo a prendere la prima implementazione e usiamola.

```rust
#trait StreamingIterator {
#  type Item;
#
#  fn next(&mut self) -> Option<&mut Self::Item>;
#}
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
trait SliceIterExt<T> {
  fn streaming_iter(&mut self) -> SliceIterMut<'_, T>;
}

impl<T> SliceIterExt<T> for [T] {
  fn streaming_iter(&mut self) -> SliceIterMut<'_, T> {
    SliceIterMut {
      iter: self.iter_mut(),
      cur: None,
    }
  }
}

fn main() {
  let mut v = vec![1, 2, 3, 4, 5];

  {
    let mut iter = v.streaming_iter();
    while let Some(x) = iter.next() {
      **x *= 2;
    }
  }

  println!("{:?}", v);
}
```

Beh, per funzionare funziona. Brutto e scomodo da utilizzare, ma funziona.
