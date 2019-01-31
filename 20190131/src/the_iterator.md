# L'iteratore

Come è possibile implementare la funzione `iter_npcs_with_nearest_enemy`?

Ciò non è banale, in quanto si tratta di ritornare due riferimenti mutabili all'interno dello stesso slice. Ovviamente il punto è che l'implementazione deve essere corretta (_sound_) e, se possibile, safe.

Cominciamo con qualcosa di estremamente banale: la funzione prende come input lo slice e l'indice di un NPC e deve ritornare una coppia di riferimenti mutabili. La funzione chiamante si occuperà di iterare sugli indici.

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
fn make_turn(npcs: &mut [NPC]) {
    let len = npcs.len();
    (0..len).for_each(|npc_index| {
        let (npc, enemy) = get_npc_with_nearest_enemy(npcs, npc_index);
        npc.move_toward(enemy);
        npc.attack(enemy);
    });
}

fn get_npc_with_nearest_enemy(npcs: &mut [NPC], index: usize) -> (&mut NPC, &mut NPC) {
    let (before, after) = npcs.split_at_mut(index);
    let (npc, after) = after.split_first_mut().unwrap();

    let other_npc = before
        .iter_mut()
        .chain(after.iter_mut())
        .filter(|other_npc| other_npc.faction != npc.faction)
        .min_by_key(|other_npc| npc.distance(other_npc))
        .unwrap();

    (npc, other_npc)
}
#
#fn main() {
#    let mut npcs: Vec<_> = (0..30).map(|_| NPC::new()).collect();
#    (0..5).for_each(|_| {
#        make_turn(&mut npcs);
#    });
#}
```

Quindi possiamo applicare lo stesso concetto per creare un iteratore che funzioni secondo gli stessi principi, giusto? Sbagliato. Sfortunatamente ciò non funziona.

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

fn make_turn(npcs: &mut [NPC]) {
    iter_npcs_with_nearest_enemy(npcs)
        .for_each(|(npc, enemy)| {
            npc.move_toward(enemy);
            npc.attack(enemy);
        });
}

fn iter_npcs_with_nearest_enemy(npcs: &mut [NPC]) -> impl Iterator<Item = (&mut NPC, &mut NPC)> {
    let len = npcs.len();
    (0..len).map(move |index: usize| {
        let (before, after) = npcs.split_at_mut(index);
        let (npc, after) = after.split_first_mut().unwrap();

        let other_npc = before
            .iter_mut()
            .chain(after.iter_mut())
            .filter(|other_npc| other_npc.faction != npc.faction)
            .min_by_key(|other_npc| npc.distance(other_npc))
            .unwrap();

        (npc, other_npc)
    })
}
#
#fn main() {
#    let mut npcs: Vec<_> = (0..30).map(|_| NPC::new()).collect();
#    (0..5).for_each(|_| {
#        make_turn(&mut npcs);
#    });
#}
```

La problematica è stata discussa [sul forum Rust](https://users.rust-lang.org/t/confused-about-indexing-and-lifetime/23993), e senza implementare un iteratore apposito sembra che non sia possibile ottenere tale risultato.
