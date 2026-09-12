# Validated WebSocket CI race pattern

## Symptom

The local game-server test passed, but GitHub Actions failed while asserting that both clients had received `entity_delta` immediately after waiting for `world_snapshot`.

Observed CI failure:

```text
Expected: entity_delta
Received: match_found, intent_ack, world_snapshot
```

## Producer timing

The server sends `world_snapshot` synchronously after `player_ready`, while `entity_delta` is emitted by an independent 50ms authoritative tick. Receiving the snapshot does not guarantee that the next tick has run.

## Minimal fix

Keep the existing bounded event helper and wait for the semantic event before collecting/asserting message types:

```ts
await Promise.all([
  waitFor(messagesA, 'world_snapshot'),
  waitFor(messagesB, 'world_snapshot'),
]);
await Promise.all([
  waitFor(messagesA, 'entity_delta'),
  waitFor(messagesB, 'entity_delta'),
]);

const typesA = messagesA.map((message) => parseMessage(JSON.parse(message)).type);
const typesB = messagesB.map((message) => parseMessage(JSON.parse(message)).type);
```

Do not replace this with an arbitrary `setTimeout`; event-driven waiting follows the protocol and survived the remote CI run.

## Evidence

- Narrow game-server test: 4 passed.
- Full local gates: format, lint, typecheck, test, build, and diff check passed.
- GitHub Actions run `31565844800`: completed successfully.
