# Il problema, di nuovo
Torniamo momentaneamente al problema iniziale e l'esigenza che abbiamo: dobbiamo far sì che possiamo iterare tra i vari elementi senza poter possedere contemporaneamente due elementi successivi.

Abbiamo creato quindi il concetto di `StreamingIterator`.

Abbiamo visto che avere un tratto iteratore diverso da `Iterator` crea molti inconvenienti.

Non possiamo semplicemente far sì che `StreamingIterator` sia a tutti gli effetti un `Iterator`?
