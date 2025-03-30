## libmeg

A highly embeddable library which implements Matrix Event Graph (MEG) structures.

### Goals
 - A library which can be embedded into mobile applications and in web browsers via WASM.
 - Implements the essence of [Matrix](https://matrix.org) rooms: the event graph CRDT underlying all rooms.
 - With the overall aim of being used in servers as well as in P2P applications. For example, instead of
   embedding [an entire Dendrite server in the browser](https://matrix.org/blog/2020/06/02/introducing-p2p-matrix/)
   or [re-implementing Matrix federation](https://github.com/rocketchat/homeserver), this
   library should form a critical (but incomplete) building block which can be used for these scenarios.

### What this includes
 - Room metadata tracking (room version/type, members, join rules, etc)
 - State resolution (and therefore auth rules)
 - Graph queries (e.g `/get_missing_events` and `/backfill`)
 - Resolving holes in the DAG (e.g processing `/get_missing_events` responses)
 - Auth and State calculations before/after an event (e.g for `/state`, `/state_ids` and `/event_auth`)
 - Proto-event creation (e.g for `/make_xxx` endpoints)
 - Forwards extremity tracking (for creating events)
 - Persistent storage required for the above (by default via SQLite but you can bring your own).

Sub-goals:
 - [Deterministic Simulation Testing](https://journal.resonatehq.io/p/deterministic-simulation-testing)
   for robustness. This is achieved by presenting an API surface which is completely deterministic, and
   therefore can be exhaustively tested. This is why this library lacks any kind of concurrency or network requests,
   as that would introduce non-determinism.
 - Better [timeline event ordering](https://artificialworlds.net/blog/2024/12/04/message-order-in-matrix/) algorithms.

Stretch goals:
 - An experimental playing ground for new protocol designs e.g set reconciliation, DAG chunking (
  [Sedimentree-style](https://github.com/inkandswitch/keyhive/blob/ec6e15d288f989712de42ce85d68f8983bb2a270/design/sedimentree.md))

### What this does not include
 - Any network requests (e.g HTTP)
 - Signature checks
 - Signing events (and thus creating events)
 - Any concurrency: the caller must bring their own concurrency primitives.
 - Persisting full events: only critical event metadata is persisted.

As a result of this, an additional library is required in order to:
 - Create events
 - Remember complete event JSON

When this project is sufficiently developed, the additional library will be written to provide the missing pieces
for Matrix federation.

For a rough guide for what this library does/doesn't do:
 - It mostly implements [`_persist_events_batch`](https://github.com/element-hq/synapse/blob/3c188231c76ee8c05a6a40d12ccfdebada86b406/synapse/storage/controllers/persist_events.py#L560)`
 - ..and [`federation_event.py`](https://github.com/element-hq/synapse/blob/a4c476305e20609a71770818c8fd3eb38eff704a/synapse/handlers/federation_event.py)
 - ..and [`event_federation.py`](https://github.com/element-hq/synapse/blob/3c188231c76ee8c05a6a40d12ccfdebada86b406/synapse/storage/databases/main/event_federation.py#L4) for storage.


 ### Development plan

 NOTE: This project is only worked on part-time so development will be slow.

 - Rapidly prototype in Go, shamelessly stealing from gomatrixserverlib for state resolution etc
 - Implement some deterministic simulation tests to validate that it works
 - Verify with some Chaos rooms.

If there's interest and momentum:

 - Implement the additional library to make it federation appropriate.
 - Performance test, with the possibility of rewriting it in Rust (particularly if Synapse wants to steal this).