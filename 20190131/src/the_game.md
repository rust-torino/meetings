# Il gioco

Per semplicità il gioco è 1D, ogni NPC ha una fazione, HP, potenza di attacco e posizione.

```rust
extern crate rand; // 0.6.3

use rand::{
    distributions::{Distribution, Uniform},
    seq::SliceRandom,
    thread_rng,
};

#[derive(Clone, Copy, Debug, PartialEq, Eq)]
enum Faction {
    Red,
    Blue,
}

#[derive(Clone, Debug, PartialEq, Eq)]
struct NPC {
    faction: Faction,
    hp: u8,
    power: u8,
    pos: u8,
    alive: bool,
}

impl NPC {
    pub fn new() -> Self {
        let mut rng = thread_rng();
        let faction = *[Faction::Red, Faction::Blue].choose(&mut rng).unwrap();
        let hp = Uniform::from(30..=100u8).sample(&mut rng);
        let power = Uniform::from(1..=7u8).sample(&mut rng);
        let pos = Uniform::from(0..=255u8).sample(&mut rng);

        Self {
            faction,
            hp,
            power,
            pos,
            alive: true,
        }
    }
}
#fn main() {
#    let _npc = NPC::new();
#}
```
