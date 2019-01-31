# Funziona!

Proviamo ad implementare la funzione `make_turn` come fatto in precedenza.

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
#impl NpcIterMutWithNearestExt for [NPC] {
#    fn iter_npc_mut_with_nearest(&mut self) -> NpcIterMutWithNearest {
#        let begin = self.as_mut_ptr();
#        // It is sound because we can point to the end of an allocated memory
#        let end = unsafe { begin.add(self.len()) };
#
#        NpcIterMutWithNearest {
#            begin,
#            end,
#            cur: begin,
#            _marker: std::marker::PhantomData,
#        }
#    }
#}
#
fn make_turn(npcs: &mut [NPC]) {
    npcs.iter_npc_mut_with_nearest().for_each(|(npc, enemy)| {
        npc.move_toward(enemy);
        npc.attack(enemy);
    });
}
#
#fn main() {
#    let mut npcs: Vec<_> = (0..30).map(|_| NPC::new()).collect();
#    (0..5).for_each(|_| {
#        make_turn(&mut npcs);
#    });
#}
```

Wow! Sembra tutto a posto! Sta funzionando!!!

**SIAMO SICURI??**
