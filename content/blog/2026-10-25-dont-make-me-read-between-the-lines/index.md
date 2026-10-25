---
title: "Jon Gjengset's ''Don't Make Me Read Between the Lines''"
date: 2026-10-25T10:23:44-04:00
description: ''
draft: false
---

> This post is a summary of [Jon Gjengset](https://thesquareplanet.com/)'s talk at Rust Munich 2026
> called "[Don't make me read between the lines](https://www.youtube.com/watch?v=Ke3PYexY6XE)." I'm
> a big fan of Jon's work generally, and this talk about comments in code really resonated with me,
> so I wanted to create a written reference for myself and others. The ideas below are not my own;
> all credit goes to Jon.

## Comment Visibility

The syntax highlighting for comments in most editors looks something like this:

```rust 
let now = time::Instant::now();
let read = &mut self.read;

// here's the trick we're going to play:
// we're going to re-try the lookups starting with the _last_ key.
// if it hits, we move on to the second-to-last, and so on.
// the moment we miss again, we yield immediately, rather than continue.

while let Some(read_i) = self.pending.pop() {
    let key = self.keys.pop().expect("pending.len() == keys.len()");
```

The comments are rendered as almost invisible, and so we as readers are trained to ignore them. Jon
changes his editor config to instead render comments as bright orange:

```rust {noClasses=false, class="orange-comments"}
let now = time::Instant::now();
let read = &mut self.read;

// here's the trick we're going to play:
// we're going to re-try the lookups starting with the _last_ key.
// if it hits, we move on to the second-to-last, and so on.
// the moment we miss again, we yield immediately, rather than continue.

while let Some(read_i) = self.pending.pop() {
    let key = self.keys.pop().expect("pending.len() == keys.len()");
```

There are a couple reasons to do this:

1. It makes you pay attention to comments.
2. It makes it painfully obvious if a comments is not useful.

In general, loud comments make you write better ones. You read comments more often, and remove what
isn't useful.

## Comments Worth Writing

Below is an _exhaustive_ list of the types of comments worth writing. Jon admits that he will
probably regret using the word "exhaustive" at some point, but the following list is an earnest
attempt to enumerate all comments he considers helpful. By extension, anything _not_ in this list is
_not_ worth writing.

### Good TODOs

TODOs tell the next reader of the code what they should be doing and why. Ideally, they are linked
to an issue tracker somewhere and/or assigned to someone.

```rust {noClasses=false, class="orange-comments"}
// TODO(malte): below is ugly, but the only way to get the sharding
// width at this point; the sharder parent does not currently have 
// the information. Change this once we support per-subgraph sharding.
```

> Yes, it's that [Malte](https://cs.brown.edu/people/malte/), a professor I TA-ed for at Brown
> University! Malte and Jon worked on
> [Noria](https://github.com/mit-pdos/noria/blame/465184ee4b57a2791d41b0f12fb8f71c45bae6b6/server/src/controller/migrate/routing.rs#L114-L117)
> together at MIT.

### Cite Your Sources

If you're implementing an algorithm, a data format, or some weird corner case from some library
you're using, include a link to the spec, research paper, Wikipedia page, etc. inline. This helps
prevent the future reader from needing to rediscover the source(s) of your change.

Also, make sure those links are permalinks. On GitHub, you can pick "Copy permalink" on one or more
lines of code. You can also use [web archive](https://web.archive.org/).

```rust {noClasses=false, class="orange-comments"}
// https://web.archive.org/web/20170404144156/https://dev.mysql.com./doc/internals/en/null-bitmap.html
// NULL-bitmap-byte = ((field-pos + offset) / 8)
// NULL-bitmap-bit = ((field-pos + offset) % 8)
if let Some(nullmap) = self.nullmap {
    let byte = self.col as usize / 8;
```

### Correctness Arguments

The code itself cannot explain why it is correct; it only explains the "how." It's very challenging
to prove the correctness of an algorithm or the safety of something in Rust, for example, with code
alone. So, add a comment!

```rust {noClasses=false, class="orange-comments"}
// Safety: we can guarantee there's only one Domain<Global>,
// because Global itself is not constructable outside of this crate
// (#[non-exhaustive] + no public constructor), and we only ever
// construct one Domain from it internally, as a single static.
unsafe impl Singleton for Global {}
```

Ideally, these kinds of safety arguments also come with other kinds of checkable annotations in your
code, e.g., `unreachable!()`, `expect()`, `compile_fail`, etc. in Rust.

### Time-Consuming Changes

If it took you more than 30 minutes to write a small chunk of code, it will likely take the next
reader a long time to understand it. Save them some time by explaining your thought process.

### Unexplained Choices

The reasons for some choices are not obvious to the next reader. For example, magic numbers and type
definitions might have subtle motivations, or they might have been chosen randomly. An explanation
in a comment helps the next reader understand whether these choices can be changed or undone without
breaking something.

### Code That Cannot Be Simplified

If you have attempted to simplify some code and failed, you should explain why the code cannot be
simplified in a comment so another person does not come around, get the same simplification idea,
and reach the same conclusion.

```rust {noClasses=false, class="orange-comments"}
// Buffered upquery responses waiting for more replay pieces.
//
// Stored as a BTreeMap so that when we iterate, we get all replays
// of one tag together - letting us avoid looking up the same tag
// more than once. A HashMap would silently break that.
replay_pieces: BTreeMap<(Tag, Vec<DataType>, usize), ReplayPieces>,
```

### Algorithm Narration

Also known as [literate programming](https://en.wikipedia.org/wiki/Literate_programming). A natural
language explanation of particularly thorny code can be a livesaver for people coming after.

```rust {noClasses=false, class="orange-comments"}
// we now know that we have the following:
//
//  grandp[src_col] -> p[col] -> n[col] ---> nchildren[][]
//                     :
//                     +----> pchildren[col][]
// 
// we want to move the sharder "before" p. this requires us to:
// ...
```

### The Road Not Taken

For code that you decided _not_ to write, there is nothing in the codebase that explains why that
code is missing. For example, you might perform the same action in three places in the codebase, but
exclude the auth check from one. In that case, you need to write a comment that explains why that
missing auth check is intentional and correct.

### Architectural Decisions

Some architectural and implementation decisions require choosing between many options and
trade-offs. After choosing a certain option among many, it helps to explain why you chose that
option, which trade-offs were at play, and which alternatives you considered and rejected. These
decisions are sometimes documented outside the source tree, but those artifacts can get lost, so the
closer to the code, the better.

```rust {noClasses=false, class="orange-comments"}
// In the context of replay routing, facing unbounded buffering 
// under load, we chose to drop late replay pieces and re-request,
// and rejected buffering everything or blocking upstream, to 
// keep memory bounded, accepting occasional duplicate work 
// because correctness matters more than a little extra work.
```

This is sometimes known as a [Y-statement](https://medium.com/olzzio/y-statements-10eb07b5a177), and
Helsing has a tool called [yadr](https://github.com/helsing-ai/yadr) which helps parse inline
[ADR](https://adr.github.io/)s.

## How to Write Comments

1. Avoid abbreviations of domain-specific language within reason. Acronyms are hard to remember, so
   it often helps to spell things out. Sometimes there is language used repeatedly within a file or
   project, in which case it's appropriate to have a glossary somewhere.
2. Avoid words like "obviously," "just," and "trivially," because chances are, if someone is reading
   your comment, they do not believe it to be obvious. Force yourself to explain thoroughly.
3. Ensure comments are correct. Typos, ambiguity, and stale links are worse than no comment at all.
4. Don't mandate comment perfection. Comments can be improved over time, so don't bikeshed wording
   too much during code review.

## Conclusion

Code only shows the _how_. Comments show the _why_, like context you can't recover, decisions and
the road not taken, and intent the code can't state. So, make comments **impossible to ignore** by
changing your editor syntax highlighting so each comment has to earn its place.

> I recently changed my comment color to orange in Neovim
> ([ref](https://github.com/csirianni/dotfiles/blob/47a1739a82c651b2d897e86a29c365aac9fba009/.config/nvim/lua/plugins/theme.lua#L3-L4)).
> I also use the Neovim plugin [todo-comments](https://github.com/folke/todo-comments.nvim) to make
> TODOs especially obvious.
