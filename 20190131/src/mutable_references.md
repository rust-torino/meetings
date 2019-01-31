# Riferimenti mutabili... a cosa?

Facciamo un passo indietro. Come funziona un normale `std::slice::IterMut`?

Concettualmente è semplice: ogni volta che chiamiamo la funzione `next` ci viene ritornato un riferimento mutabile **diverso**.

Possiamo quindi fare cose come queste:
```rust
let mut v = vec![1, 2, 3, 4, 5];
let v_of_mut_refs: Vec<&mut i32> = v.iter_mut().collect();
```

Ciò è possibile in quanto ogni riferimento esisterà **solo una volta**.

Ed il nostro iteratore? Restituisce sempre coppie di riferimenti mutabili... Oh oh! È possibile, attraverso la nostra implementazione, avere più di un riferimento mutabile allo stesso elemento, e questo in Rust è undefined behaviour.

La nostra implementazione non è corretta.
