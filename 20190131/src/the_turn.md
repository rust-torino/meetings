# Il turno

Ad ogni turno ogni NPC può muoversi verso l'NPC di fazione opposta più vicino nel caso esso sia a distanza maggiore di 1.

Fatto ciò proverà ad attaccare tale nemico nel caso la distanza sia minore o uguale di 1. L'attacco ha come effetto la riduzione degli HP dell'avversario pari alla propria potenza. Nel caso l'HP dell'avversario scenda al di sotto di 1, l'avversario muore.

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
impl NPC {
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
    pub fn attack(&self, other: &mut Self) {
        if other.hp <= self.power {
            other.hp = 0;
            other.alive = false;
        } else {
            other.hp -= self.power;
        }
    }

    pub fn move_toward(&mut self, other: &Self) {
        if self.distance(other) > 1 {
            if other.pos > self.pos {
                self.pos += 1;
            } else {
                self.pos -= 1;
            }
        }
    }

    pub fn distance(&self, other: &Self) -> u8 {
        (self.pos as i16 - other.pos as i16).abs() as u8
    }
}

fn make_turn(npcs: &mut [NPC]) {
    iter_npcs_with_nearest_enemy(npcs)
        .for_each(|(npc, enemy)| {
            npc.move_toward(enemy);
            npc.attack(enemy);
        });
}

fn iter_npcs_with_nearest_enemy(npcs: &mut [NPC]) -> impl Iterator<Item = (&mut NPC, &mut NPC)> {
    #if true {
    unimplemented!()
    #} else {
    #    let mut iter = npcs.iter_mut();
    #    std::iter::once((iter.next().unwrap(), iter.next().unwrap()))
    #}
}
#
#fn main() {
#    let mut npcs: Vec<_> = (0..30).map(|_| NPC::new()).collect();
#    (0..5).for_each(|_| {
#        make_turn(&mut npcs);
#    });
#}
```
