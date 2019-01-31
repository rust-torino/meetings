# Ma perché?

È giusto a questo punto chiedersi *perché* succede questo. Alla fine l'iteratore ha al suo interno un marker inserito per mantenere un solo accesso mutabile alla slice.

Rivediamo un secondo la dichiarazione della implementazione del nostro iteratore.
```rust
#struct NPC;
#struct NpcIterMutWithNearest<'a> {
#    begin: *mut NPC,
#    end: *mut NPC,
#    cur: *mut NPC,
#    _marker: std::marker::PhantomData<&'a mut NPC>,
#}
#
impl<'a> Iterator for NpcIterMutWithNearest<'a> {
    type Item = (&'a mut NPC, &'a mut NPC);

    fn next(&mut self) -> Option<Self::Item> {
        // ...
#        unimplemented!()
    }
}
#fn main() {}
```

Analizziamo bene queste poche righe di codice. `next` ritorna un `Option`, che quindi potrà contenere o meno il tipo `Item`.
`Item` a sua volta è una tupla di due riferimenti mutabili con _lifetime_ `'a`.

Tuttavia non viene mai specificata la _lifetime_ della struttura in questione! Ovviamente è presente in modo implicito in `&mut self` di `next`, ma comunque è completamente scorrelata da `Item`.

Questo è il vero motivo per il quale è possibile ottenere un vettore a tutti i riferimenti mutabili di una slice, e lo stesso per cui la nostra implementazione, apparentemente corretta, è completamente da buttare.

Quello che vogliamo è un valore di ritorno che abbia una _lifetime_ dipendente da quella dell'iteratore, in modo tale da non poter chiamare `next` prima di aver distrutto il valore di ritorno (in questo caso i due riferimenti mutabili).

Gli iteratori di cui abbiamo bisogno vengono chiamati **streaming iterators**.
