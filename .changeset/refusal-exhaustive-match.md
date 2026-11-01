---
---

fix(relay): replace Account::refusal wildcard with exhaustive Refusal matching

`Account::refusal()` previously matched on `u8` and ended in `_ => None`, using a wildcard to
represent "no refusal". If a third refusal cause were added, `refuse()` and `close_refused()` were
compiler-forced to handle it, but `Account::refusal()` would silently swallow the new cause into
`None`. As a result, the connection loop's iteration check in `relay.rs` and the read's refusal
guard in `subscribe.rs` would take `None` as "nothing to do", leaving the socket open and stored
reads continuing to produce and discard frames.

`Refusal` now models the unrefused state as an explicit variant (`Refusal::Clear`), and
`Account::refusal()` as well as `FrameSink::refusal()` return `Refusal` directly. Readers in
`relay.rs`, `subscribe.rs`, and `sink.rs` now match exhaustively over `Refusal`, forcing the
compiler to require every reader to decide whenever a new variant is introduced. Any unknown
refusal code in `Account::refusal` panics rather than being silently swallowed as unrefused.

The TypeScript tree is untouched: no `src/` or `test/` file changes, so the published package has no
user-visible change.
