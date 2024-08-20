NIP-XX: Pubkey Migration Proof
------------------------------

`draft` `optional`

This NIP defines a preemptive method for proving ownership of a new pubkey when migrating from an old one. This allows users to strengthen their continuity between pubkeys and notify followers when moving to a new keypair in events such as private key compromise, new vanity npub, etc.

Here is how it works:

## Preimage Commitment Event

A user intending to enable future migration publishes a kind `18` event with the following structure:

```json
{
  "pubkey": <current pubkey>
  "kind": 18
  "content": "<32-byte lowercase hex-encoded SHA256 hash of a secret preimage>"
  "tags": [
    ["alt", "Pubkey migration preimage commitment"]
  ]
}
```

The preimage is some strong unguessable text. The user should store the preimage securely offline and ensure it can be hashed again to obtain the same SHA256 output. The hash of the preimage will be used to prove ownership of the new pubkey in the future, if needed.

## Pubkey Migration Event 

To migrate to a new pubkey, the user publishes a kind `19` event from the new pubkey with the following structure:

```json
{
  "pubkey": <new pubkey>
  "kind": 19
  "content": "<the plaintext preimage whose SHA256 hash was published in the kind 18 event>"
  "tags": [
    ["e", "<event id of the kind 18 preimage commitment event>"]
    ["p", "<old pubkey being migrated from>"]
    ["alt", "Pubkey migration - moving from <old pubkey> to <new pubkey>"]
  ]
}
```

The event MUST include:
- An "e" tag referencing the event id of the original kind 18 preimage commitment event
- A "p" tag with the old pubkey being migrated from

## Pubkey Burn Event

A pubkey can publish a kind 20 event to signal that no kind 18 events published afterward should be considered valid.

## Client Behavior

### Migrated Pubkeys

Clients can:
- ignore this NIP
- show users that a pubkey has migrated and provide a link to the new pubkey
- show users the new pubkey instead of the old pubkey

To obtain migration info for a pubkey in the UI, clients should do the following:

1. Filter kind 18 events authored by any given pubkey.
2. If one exists, client should query kind 19 events referencing the kind 18 event id. 
3. If a kind 19 is found, the preimage should be hashed with SHA256 and checked against the kind 18 content.
4. If they are equal then the kind 19 author is the new successor for the kind 18 author.
5. Show migration in UI.

Kind 19 events can appear in timelines to show users that someone they follow has migrated.

1. Include a filter in the timeline REQ for kind 19 events with a "#p" tag containing the user's follow list.
2. Verify kind 19 as above and display in UI.

## Security Considerations

This scheme only works reliably if the kind 18 is published before it is needed and the preimage is kept secret.

### What if a private key is compromised before publishing a kind 18?

The original owner of the private key should publish a kind 18 followed by a kind 20 ASAP. Then they can migrate to a new keypair.

### What if a private key is compromised and the attacker publishes a 

### What if a preimage is compromised?

The preimage thief can effectively steal the old pubkey's account by creating a new pubkey and publishing a valid kind 19 migration event.


- only the oldest kind 18 is considered valid
the kind 18 is published before it is needed. Multiple kind 18s SHOULD NOT be published, but in the event that a preimage is lost it may be necessary to do so.

Users should ensure the preimage is stored securely offline and not reused. The preimage should be sufficiently long and random to prevent guessing. It is recommended that the preimage include at least 256 bits of cryptographically secure entropy. It is critical that the stored version of the preimage can be easily and reliably digitized for hashing.

Typically only the oldest available kind 18 should be trusted, but the option to publish a legitimate secondary kind 18 must be available or only 1 could be published per pubkey.


This scheme does not protect against a compromised private key being used to publish false migration events. Users should migrate promptly if they suspect their key has been compromised.

This draft NIP outlines the basic structure and behavior for the pubkey ownership proof and migration scheme you described. It defines two new event kinds (18 and 19) for the preimage commitment and migration events respectively. The draft also includes some guidance on how clients and relays should handle these events.

You may want to adjust or expand on certain aspects, such as:
1. Specifying a recommended length/format for the preimage
2. Adding more detail on how clients should handle conflicts or multiple migration events
3. Discussing any potential privacy implications
4. Specifying how long relays should retain kind:18 events

## Relay Implementation

Relays must not delete kind 18 events??

