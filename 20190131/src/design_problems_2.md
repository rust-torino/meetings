# Problemi di design (2)
Proviamo ad utilizzare un oggetto per astrarre il secondo livello di referenzialità:

```rust
#use std::ops::{Deref, DerefMut};
struct RefMut<'a, T>(&'a mut T);

impl<T> Deref for RefMut<'_, T> {
  type Target = T;

  fn deref(&self) -> & Self::Target {
    self.0
  }
}

impl<T> DerefMut for RefMut<'_, T> {
  fn deref_mut(&mut self) -> &mut Self::Target {
    self.0
  }
}

impl<T> AsRef<T> for RefMut<'_, T> {
  fn as_ref(&self) -> &T {
    self.0
  }
}

impl<T> AsMut<T> for RefMut<'_, T> {
  fn as_mut(&mut self) -> &mut T {
    self.0
  }
}

trait StreamingIterator<'a> {
  type Item;

  fn next(&mut self) -> Option<&mut RefMut<'a, Self::Item>>;
}

struct SliceIterMut<'a, T> {
  iter: std::slice::IterMut<'a, T>,
  cur: Option<RefMut<'a, T>>,
}

impl<'a, T> StreamingIterator<'a> for SliceIterMut<'a, T> {
  type Item = T;

  fn next(&mut self) -> Option<&mut RefMut<'a, Self::Item>>
  {
    self.cur = self.iter.next().map(|t| RefMut(t)); 
    self.cur.as_mut()
  }
}

#fn main() {}
```

Oooookay?

Certamente non c'è più un doppio riferimento diretto, tuttavia c'è un bel po' di boilerplate e per giunta il tratto adesso richiede una _lifetime_...
Unico vantaggio oggettivo è che `Item` adesso non deve essere specificato in quanto riferimento mutabile, riuscendo in parte a soddisfare il concetto per cui uno streaming iterator di questo tipo deve tornare un "riferimento mutabile a qualcosa", dove _qualcosa_ è effettivamente `Item`.
