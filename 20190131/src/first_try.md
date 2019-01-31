# Primo tentativo

Riassumiamo: dobbiamo ottenere un iteratore che permetta di prendere un riferimento mutabile a un `NPC` e il riferimento mutabile all'`NPC` più vicino.

Creiamo quindi un tratto per estendere le fuzionalità di uno slice di `NPN`.

```rust
##[derive(Clone, Copy, Debug, PartialEq, Eq)]
#enum Faction {
#    Red,
#    Blue,
#}
#
##[derive(Clone, Debug, PartialEq, Eq)]
#struct NPC {
#    faction: Faction,
#    hp: u8,
#    power: u8,
#    pos: u8,
#    alive: bool,
#}
#
trait NpcIterMutWithNearestExt {
    fn iter_npc_mut_with_nearest(&mut self) -> NpcIterMutWithNearest;
}
#
#struct NpcIterMutWithNearest<'a> {
#    begin: *mut NPC,
#    end: *mut NPC,
#    cur: *mut NPC,
#    _marker: std::marker::PhantomData<&'a mut NPC>,
#}
#fn main() {}
```

Abbiamo quindi bisogno di creare `NpcIterMutWithNearest`, il nostro iteratore.

```rust
##[derive(Clone, Copy, Debug, PartialEq, Eq)]
#enum Faction {
#    Red,
#    Blue,
#}
#
##[derive(Clone, Debug, PartialEq, Eq)]
#struct NPC {
#    faction: Faction,
#    hp: u8,
#    power: u8,
#    pos: u8,
#    alive: bool,
#}
#
struct NpcIterMutWithNearest<'a> {
    begin: *mut NPC,
    end: *mut NPC,
    cur: *mut NPC,
    _marker: std::marker::PhantomData<&'a mut NPC>,
}
#fn main() {}
```
Come possiamo vedere, la struttura contiene tre puntatori, in modo da mantenere l'informazione sull'inizio e la fine dello slice e sull'elemento corrente. Il `marker` serve per far sì che la _lifetime_ `'a` venga in qualche modo _salvata_ all'interno della struttura.

Adesso implementiamo il tratto `Iterator` per `NpcIterMutWithNearest`.
```rust
#extern crate rand; // 0.6.3
#
#use rand::{
#    distributions::{Distribution, Uniform},
#    seq::SliceRandom,
#    thread_rng,
#};
#
##[derive(Clone, Copy, Debug, PartialEq, Eq)]
#enum Faction {
#    Red,
#    Blue,
#}
#
##[derive(Clone, Debug, PartialEq, Eq)]
#struct NPC {
#    faction: Faction,
#    hp: u8,
#    power: u8,
#    pos: u8,
#    alive: bool,
#}
#
#impl NPC {
#    pub fn new() -> Self {
#        let mut rng = thread_rng();
#        let faction = *[Faction::Red, Faction::Blue].choose(&mut rng).unwrap();
#        let hp = Uniform::from(30..=100u8).sample(&mut rng);
#        let power = Uniform::from(1..=7u8).sample(&mut rng);
#        let pos = Uniform::from(0..=255u8).sample(&mut rng);
#
#        Self {
#            faction,
#            hp,
#            power,
#            pos,
#            alive: true,
#        }
#    }
#
#    pub fn attack(&self, other: &mut Self) {
#        if other.hp <= self.power {
#            other.hp = 0;
#            other.alive = false;
#        } else {
#            other.hp -= self.power;
#        }
#    }
#
#    pub fn move_toward(&mut self, other: &Self) {
#        if self.distance(other) > 1 {
#            if other.pos > self.pos {
#                self.pos += 1;
#            } else {
#                self.pos -= 1;
#            }
#        }
#    }
#
#    pub fn distance(&self, other: &Self) -> u8 {
#        (self.pos as i16 - other.pos as i16).abs() as u8
#    }
#}
#
/// Stable replacement for `std::ptr::offset_from`
fn ptr_distance<T>(a: *const T, b: *const T) -> isize {
    (b as isize - a as isize) / std::mem::size_of::<T>() as isize
}
#
#trait NpcIterMutWithNearestExt {
#    fn iter_npc_mut_with_nearest(&mut self) -> NpcIterMutWithNearest;
#}
#
#struct NpcIterMutWithNearest<'a> {
#    begin: *mut NPC,
#    end: *mut NPC,
#    cur: *mut NPC,
#    _marker: std::marker::PhantomData<&'a mut NPC>,
#}

impl<'a> Iterator for NpcIterMutWithNearest<'a> {
    type Item = (&'a mut NPC, &'a mut NPC);

    fn next(&mut self) -> Option<Self::Item> {
        if self.cur < self.end {
            // before, after and cur are disjoint
            let before = unsafe {
                std::slice::from_raw_parts_mut(
                    self.begin,
                    ptr_distance(self.begin, self.cur) as usize,
                )
            };
            // cur < end, therefore it is sound to increment it
            let next_ptr = unsafe { self.cur.add(1) };
            let after = unsafe {
                std::slice::from_raw_parts_mut(next_ptr, ptr_distance(next_ptr, self.end) as usize)
            };
            let cur = unsafe { &mut *self.cur };

            let enemy = before
                .iter_mut()
                .chain(after.iter_mut())
                .filter(|npc| npc.faction != cur.faction)
                .min_by_key(|npc| npc.distance(cur));

            self.cur = next_ptr;
            enemy.map(|enemy| (cur, enemy))
        } else {
            None
        }
    }
}
#
#fn main() {}
```
È possibile immediatamente notare la presenza di una funzione che mi permetta di valutare la distanza tra due puntatori in funzione della grandezza dell'oggetto a cui puntano, come l'operazione sottrazione tra puntatori C.

Come è possibile aspettarsi in questi casi, ci siamo limitati all'implementazione della funzione `next`.
Questa si occupa di controllare che il puntatore `cur` sia minore di `end` e, in questo caso, costruirà un riferimento mutabile partendo da `cur` e due slice. Questi saranno slice mutabili agli elementi precendi `cur` e a quelli successivi.
A questo punto viene semplicemente cercato l'`NPC` di fazione diversa a minor distanza, il puntatore `cur` viene incrementato e la funzione torna la coppia di riferimenti mutabili ottenuti. Ovviamente nel caso `cur` abbia raggiunto `end` viene ritornato `None` e l'iterazione termina.

Manca solo da implementare il tratto per gli slice di `NPC`, dettaglio semplice.
```rust
#extern crate rand; // 0.6.3
#
#use rand::{
#    distributions::{Distribution, Uniform},
#    seq::SliceRandom,
#    thread_rng,
#};
#
##[derive(Clone, Copy, Debug, PartialEq, Eq)]
#enum Faction {
#    Red,
#    Blue,
#}
#
##[derive(Clone, Debug, PartialEq, Eq)]
#struct NPC {
#    faction: Faction,
#    hp: u8,
#    power: u8,
#    pos: u8,
#    alive: bool,
#}
#
#impl NPC {
#    pub fn new() -> Self {
#        let mut rng = thread_rng();
#        let faction = *[Faction::Red, Faction::Blue].choose(&mut rng).unwrap();
#        let hp = Uniform::from(30..=100u8).sample(&mut rng);
#        let power = Uniform::from(1..=7u8).sample(&mut rng);
#        let pos = Uniform::from(0..=255u8).sample(&mut rng);
#
#        Self {
#            faction,
#            hp,
#            power,
#            pos,
#            alive: true,
#        }
#    }
#
#    pub fn attack(&self, other: &mut Self) {
#        if other.hp <= self.power {
#            other.hp = 0;
#            other.alive = false;
#        } else {
#            other.hp -= self.power;
#        }
#    }
#
#    pub fn move_toward(&mut self, other: &Self) {
#        if self.distance(other) > 1 {
#            if other.pos > self.pos {
#                self.pos += 1;
#            } else {
#                self.pos -= 1;
#            }
#        }
#    }
#
#    pub fn distance(&self, other: &Self) -> u8 {
#        (self.pos as i16 - other.pos as i16).abs() as u8
#    }
#}
#
#/// Stable replacement for `std::ptr::offset_from`
#fn ptr_distance<T>(a: *const T, b: *const T) -> isize {
#    (b as isize - a as isize) / std::mem::size_of::<T>() as isize
#}
#
#trait NpcIterMutWithNearestExt {
#    fn iter_npc_mut_with_nearest(&mut self) -> NpcIterMutWithNearest;
#}
#
#struct NpcIterMutWithNearest<'a> {
#    begin: *mut NPC,
#    end: *mut NPC,
#    cur: *mut NPC,
#    _marker: std::marker::PhantomData<&'a mut NPC>,
#}
#
#impl<'a> Iterator for NpcIterMutWithNearest<'a> {
#    type Item = (&'a mut NPC, &'a mut NPC);
#
#    fn next(&mut self) -> Option<Self::Item> {
#        if self.cur < self.end {
#            // before, after and cur are disjoint
#            let before = unsafe {
#                std::slice::from_raw_parts_mut(
#                    self.begin,
#                    ptr_distance(self.begin, self.cur) as usize,
#                )
#            };
#            // cur < end, therefore it is sound to increment it
#            let next_ptr = unsafe { self.cur.add(1) };
#            let after = unsafe {
#                std::slice::from_raw_parts_mut(next_ptr, ptr_distance(next_ptr, self.end) as usize)
#            };
#            let cur = unsafe { &mut *self.cur };
#
#            let enemy = before
#                .iter_mut()
#                .chain(after.iter_mut())
#                .filter(|npc| npc.faction != cur.faction)
#                .min_by_key(|npc| npc.distance(cur));
#
#            self.cur = next_ptr;
#            enemy.map(|enemy| (cur, enemy))
#        } else {
#            None
#        }
#    }
#}
#
impl NpcIterMutWithNearestExt for [NPC] {
    fn iter_npc_mut_with_nearest(&mut self) -> NpcIterMutWithNearest {
        let begin = self.as_mut_ptr();
        // It is sound because we can point to the end of an allocated memory
        let end = unsafe { begin.add(self.len()) };

        NpcIterMutWithNearest {
            begin,
            end,
            cur: begin,
            _marker: std::marker::PhantomData,
        }
    }
}
#
#fn main() {
#    let mut _npcs: Vec<_> = (0..30).map(|_| NPC::new()).collect();
#}
```

L'iteratore è fatto. Funzionerà?
