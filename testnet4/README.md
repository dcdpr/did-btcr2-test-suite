# did:btcr2 Testnet4 Test Vectors

Live `did:btcr2` test vectors anchored on **testnet4** (about 10 minute blocks).

- **On-chain:** every update signal is an `OP_RETURN` on testnet4. Explorer: https://mempool.space/testnet4. Esplora REST API: `https://mempool.space/testnet4/api`.
- **CAS:** the genesis documents, signed updates, and CAS Announcement Maps that a vector delivers through the CAS are pinned on IPFS (CIDv1, raw codec, sha2-256) and retrievable from a public gateway.
- **Sidecar:** everything else, SMT proofs included, rides in the resolution options of `resolve/input.json`.

## Layout

Each vector set lives under `{k1|x1}/{hash}/`:

- `create/input.json`, `create/output.json`: the create operation.
- `update/input.json`, `update/output.json`: the update operation; `update/NN/` for a set with more than one update. `signingMaterial` is the secret key of the signer.
- `resolve/input.json`, `resolve/output.json`: the resolve operation with the sidecar; `resolve/NN/` for a sub-vector with resolution options.
- `other.json`: the keys and the genesis document.
- `signals.json` (a set with a Beacon Signal on the chain): one entry per signal with the update it commits to (`update` is the `update/NN/` number; `duplicate` marks a second signal of the same update in a later block), the beacon id, the address, the `txid`, the block height, hash, time, and `mediantime`, and the signal bytes. A cohort member records the shared signal with the cohort id and members.

## How to compare

- Take the inputs and produce your own outputs. The signed bytes of an update are not compared: BIP340 signing is randomized. Your signed update must verify and must resolve to the recorded document.
- `resolve/output.json` is the DID Resolution result. Compare `didDocument`, `didDocumentMetadata.versionId`, and `didDocumentMetadata.deactivated`. Compare `didDocumentMetadata.confirmations` as "at least the recorded value": it grows with the chain. `updated` is the header time of the block of the last applied update.
- A negative vector records `didResolutionMetadata.error` with the DID Resolution error code. Compare the code only; `errorMessage` is the text of this implementation.
- Resolution applies a beacon signal at six confirmations (the specification default). A resolve before that depth returns an earlier version.
- `signals.json` is what your signal discovery must find at the beacon addresses. Compare the `txid`, the block, and the signal bytes; a resolver that reads no chain can take the signals from the file.

## Positive vectors

| Scenario | Type | Delivery | Expected | Path |
|----------|------|----------|----------|------|
| 01-k1-base | k1 | no update | versionId 1 | [`k1/qsprrscr/`](./k1/qsprrscr/) |
| 02-k1-sidecar-update | k1 | 1 update sidecar | versionId 2 | [`k1/qspz5wep/`](./k1/qspz5wep/) |
| 03-x1-base | x1 | genesis sidecar | versionId 1 | [`x1/qsv3urwd/`](./x1/qsv3urwd/) |
| 04-x1-sidecar-update | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/q359fq2n/`](./x1/q359fq2n/) |
| 05-x1-no-beacon | x1 | genesis cas | versionId 1 | [`x1/qnh3j3xa/`](./x1/qnh3j3xa/) |
| 06-x1-cas-3-updates | x1 | genesis cas, 3 updates cas | versionId 4, deactivated | [`x1/qjdupajy/`](./x1/qjdupajy/) |
| 07-k1-sidecar-deactivate | k1 | 1 update sidecar | versionId 2, deactivated | [`k1/qsps5avm/`](./k1/qsps5avm/) |
| 08-x1-cas-update-deactivate | x1 | genesis cas, 2 updates cas | versionId 3, deactivated | [`x1/q37ws22m/`](./x1/q37ws22m/) |
| 09a-x1-cas-update-announcement | x1 | genesis cas, 1 update cas, announcement cas | versionId 2 | [`x1/qjg3rvgl/`](./x1/qjg3rvgl/) |
| 09b-x1-cas-update-announcement-paired | x1 | genesis cas, 1 update cas, announcement cas | versionId 2 | [`x1/qny5xjfv/`](./x1/qny5xjfv/) |
| 10a-x1-sidecar-update-cas-announcement | x1 | genesis sidecar, 1 update sidecar, announcement sidecar | versionId 2 | [`x1/q3zsv63l/`](./x1/q3zsv63l/) |
| 10b-x1-sidecar-update-cas-announcement-paired | x1 | genesis sidecar, 1 update sidecar, announcement sidecar | versionId 2 | [`x1/qs7clm0n/`](./x1/qs7clm0n/) |
| 13-k1-update-p2wpkh | k1 | 1 update sidecar | versionId 2 | [`k1/qspxpjr0/`](./k1/qspxpjr0/) |
| 14-k1-update-p2tr | k1 | 1 update sidecar | versionId 2 | [`k1/qspaj3wh/`](./k1/qspaj3wh/) |
| 15-x1-beacon-rotation | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qnenf7q8/`](./x1/qnenf7q8/) |
| 16-x1-beacon-add-then-use | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qsvpyve8/`](./x1/qsvpyve8/) |
| 17-x1-vm-add-rotate-authentication | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qsukh94m/`](./x1/qsukh94m/) |
| 18-x1-embedded-invocation-key | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/qjmhfkyx/`](./x1/qjmhfkyx/) |
| 19-x1-relative-ids | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/qjszxrzd/`](./x1/qjszxrzd/) |
| 20-k1-cas-update | k1 | 1 update cas | versionId 2 | [`k1/qsprptnr/`](./k1/qsprptnr/) |
| 21-k1-deactivate-then-update | k1 | 2 updates sidecar | versionId 2, deactivated | [`k1/qsp472vc/`](./k1/qsp472vc/) |
| 22-x1-three-updates-resolution-options | x1 | genesis sidecar, 3 updates sidecar | versionId 4 | [`x1/qsryv830/`](./x1/qsryv830/) |
| 23-k1-duplicate-signal | k1 | 2 updates sidecar, 1 duplicate signal | versionId 3 | [`k1/qspm6rmq/`](./k1/qspm6rmq/) |
| 24-k1-removed-beacon-signal | k1 | 2 updates sidecar | versionId 2 | [`k1/qspqm6j2/`](./k1/qspqm6j2/) |

### 01-k1-base

Base resolution: k1 (key-type) DID with default 3 singleton beacons, no updates. Mirrors danubetech example 1.

DID: `did:btcr2:k1qsprrscrk9fsazdegjtc7p6lups88ms3ee2hq2afaycvnmvqt2p9y9qatcxd5`

### 02-k1-sidecar-update

k1 (key-type) DID with default 3 singleton beacons, 1 sidecar update adding a DIDCommMessaging service. Mirrors danubetech example 2.

DID: `did:btcr2:k1qspz5wepjwqgtcg5e9qasyvmwhy9fdd6sdx8pz89zu4hcqeacz9s62s8rp8r9`

### 03-x1-base

Base resolution: x1 (external-type) DID with default 3 singleton beacons, no updates. Genesis document delivered via sidecar. Mirrors danubetech example 3.

DID: `did:btcr2:x1qsv3urwdx8pnx52shpvqjnmg7rxsrngh95h85ef0n4e6xuzxkqptx2ga3aq`

### 04-x1-sidecar-update

x1 (external-type) DID with default 3 singleton beacons, 1 sidecar update adding a DecentralizedWebNode service. Mirrors danubetech example 4.

DID: `did:btcr2:x1q359fq2nww7hgypedz5qnfmylanzw2p62z7lzm8agctugthkgp9uwprq8hr`

### 05-x1-no-beacon

Base resolution: x1 (external-type) DID with NO beacon services in the document, no updates. Genesis doc delivered via sidecar. Mirrors danubetech example 5 (CAS-delivered genesis); our vector uses sidecar delivery so the vector is self-contained.

DID: `did:btcr2:x1qnh3j3xa57gnf6lkshvztp4tngeqhsy3n7qhfc9n995ww99q6ne2qyr97z7`

### 06-x1-cas-3-updates

x1 DID with default 3 singleton beacons, 3 updates delivered via CAS (didcomm, dwn, then deactivate). Mirrors danubetech example 6.

DID: `did:btcr2:x1qjdupajytz9csrtz3tjnaql06tg7jq80yv45vj36kj70pmf7ycz0z9js25l`

### 07-k1-sidecar-deactivate

k1 (key-type) DID with default 3 singleton beacons, 1 sidecar update deactivating the DID. Mirrors danubetech example 7.

DID: `did:btcr2:k1qsps5avmw05fjke7jz2dvufdmntzetrxnk4q6t0qfdmhm7hz0ttljpge237ae`

### 08-x1-cas-update-deactivate

x1 DID with default 3 singleton beacons, 1 CAS update (add didcomm) + CAS deactivate. Mirrors danubetech example 8.

DID: `did:btcr2:x1q37ws22m4c04rze3d9vn4rkt2tzd6h5k9z74a229gjd4mp9z6gqt2c7hlvp`

### 09a-x1-cas-update-announcement

x1 DID with default 3 singleton beacons, 1 CAS update + CAS Announcement Map. Mirrors danubetech example 9a (paired with 09b in danubetech's tree).

DID: `did:btcr2:x1qjg3rvgledjrugvv0snk7xg80chyywfpyfss27pw99h5kug03u46qnmqqy6`

### 09b-x1-cas-update-announcement-paired

x1 DID with default 3 singleton beacons, 1 CAS update + CAS Announcement Map. Mirrors danubetech example 9b (paired with 09a in danubetech's tree).

DID: `did:btcr2:x1qny5xjfv5cl4veelkpvrdv2qjrwxh6mzdqdwvak90adymzh79pmdskwad2c`

### 10a-x1-sidecar-update-cas-announcement

x1 DID with default 3 singleton beacons + dedicated CASBeacon, 1 sidecar update. CAS Announcement delivered via sidecar. Mirrors danubetech example 10a.

DID: `did:btcr2:x1q3zsv63l5vv7rfejlk7smxjpanr8djjnszejcj3lvsl77sk4fp6fq37kkel`

### 10b-x1-sidecar-update-cas-announcement-paired

x1 DID with default 3 singleton beacons + dedicated CASBeacon, 1 sidecar update adding DIDCommMessaging. CAS Announcement delivered via sidecar. Mirrors danubetech example 10b (paired with 10a in danubetech's tree).

DID: `did:btcr2:x1qs7clm0nsmwxdw9a57efe924u4pk7zl93p8g8eekfwp5cfrcap4ry9k605a`

### 13-k1-update-p2wpkh

k1 DID with the default 3 singleton beacons, 1 sidecar update anchored at the #initialP2WPKH beacon.

DID: `did:btcr2:k1qspxpjr0xd97l92kqrh8pl4pjgjqvdxr6pyaxxqyr7g0t9h648kct5q7uqvsz`

### 14-k1-update-p2tr

k1 DID with the default 3 singleton beacons, 1 sidecar update anchored at the #initialP2TR beacon.

DID: `did:btcr2:k1qspaj3wh7ztzvtw8qgcq0jm7dw2d44xp0k9397x0zkyewq0jr7r4crgefh3q4`

### 15-x1-beacon-rotation

x1 DID; update 1 replaces the endpoint of the #initialP2PKH singleton beacon with a new address (beacon rotation); update 2 is anchored at the rotated beacon.

DID: `did:btcr2:x1qnenf7q8f07k6qq30xx6qez6laurmqes9cjpezjshpj4mw8aaaknc7azjf6`

### 16-x1-beacon-add-then-use

x1 DID; update 1 adds a fourth singleton beacon #newBeacon; update 2 is anchored at #newBeacon, so the resolver needs a second discovery round.

DID: `did:btcr2:x1qsvpyve87n97xm7mxswheuwegmyd9naefzk7s6kspt6cq46az3jk7tfsg32`

### 17-x1-vm-add-rotate-authentication

x1 DID; update 1 adds the verification method #key-1, replaces authentication with it, and adds it to capabilityInvocation; update 2 is signed with #key-1.

DID: `did:btcr2:x1qsukh94mtqxv5e72hh2rhmyutu3jfcypvrr4m2p3mpe38k0328rcxxhleny`

### 18-x1-embedded-invocation-key

x1 DID whose capabilityInvocation embeds the initial verification method as an object; verificationMethod is empty. 1 sidecar update signed with the embedded method.

DID: `did:btcr2:x1qjmhfkyxuvj5el2wsvkx6cyp8k4luhqxxwt85egq8rlzhke6ys4026ck7ml`

### 19-x1-relative-ids

x1 DID whose genesis document spells every id as a relative DID URL (#initialKey, #initialP2PKH). 1 sidecar update.

DID: `did:btcr2:x1qjszxrzdu5j2pm6ay24q8qkypcxp27y0j6kze9q4uj37mdqq8jdrk8kkgfz`

### 20-k1-cas-update

k1 DID with the default 3 singleton beacons, 1 update delivered through the CAS (no sidecar).

DID: `did:btcr2:k1qsprptnrrmlzjwadmqem204tgftcsulu3n70hpgavqrma6tlta5g39sgw566g`

### 21-k1-deactivate-then-update

k1 DID; update 1 deactivates the DID; update 2 is a later anchored update that a resolver must not apply. Resolves to version 2, deactivated; version 3 does not exist.

DID: `did:btcr2:k1qsp472vc5tgquf0hfns4x9cnt0h2jqvgdjftufqy5ar0k7cu32xanwsuacycj`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

### 22-x1-three-updates-resolution-options

x1 DID with 3 sidecar updates, each anchored in its own block. The sub-vectors cover versionId, versionTime (before the first update, between the first and the second, at the second), an unreachable versionId, both options, and minConf above the chain depth.

DID: `did:btcr2:x1qsryv830c4av869cfpl2mz87zzk28uv4ylkn3cntzahfu0xdnvtyg66nlz7`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"1"}` | versionId 1 |
| `resolve/02/` | `{"versionId":"2"}` | versionId 2 |
| `resolve/03/` | `{"versionId":"3"}` | versionId 3 |
| `resolve/04/` | `{"versionId":"9"}` | error `NOT_FOUND` |
| `resolve/05/` | `{"versionTime":"before:1"}` | versionId 1 |
| `resolve/06/` | `{"versionTime":"after:1"}` | versionId 2 |
| `resolve/07/` | `{"versionTime":"at:2"}` | versionId 3 |
| `resolve/08/` | `{"versionId":"2","versionTime":"before:1"}` | error `INVALID_OPTIONS` |
| `resolve/09/` | `{"minConf":1000000}` | versionId 1 |

### 23-k1-duplicate-signal

k1 DID; update 1 (version 2) and update 2 (version 3) are anchored at #initialP2WPKH; entry 3 announces the signed update of version 2 again at the same beacon, in a later block (a duplicate signal). Resolves to version 3. The sub-vector asks for a versionTime after the block of version 3 and before the block of the duplicate: a resolver confirms the duplicate before the versionTime test, so the duplicate does not end the resolution at version 2.

DID: `did:btcr2:k1qspm6rmqtrv03wvjz6hung7wnf4vrhjqgcuhv3hy3920a2t8fjqh3ssedy3nn`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionTime":"before:3"}` | versionId 3 |

### 24-k1-removed-beacon-signal

k1 DID; update 1 removes the #initialP2PKH beacon service (version 2); update 2 is announced at the removed #initialP2PKH address and adds a service. A resolver ignores the signal of a beacon address that the current document does not carry (Process Next Update, step 4): resolves to version 2, and version 3 does not exist.

DID: `did:btcr2:k1qspqm6j2u5pe5s4ucchgs63cdmmyprh8vahc2tq0x8l0rm0zytqd2wqvld589`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

## Negative vectors

| Scenario | Type | Delivery | Expected | Path |
|----------|------|----------|----------|------|
| n01-k1-invalid-did-checksum | k1 | no update | error `INVALID_DID` | [`k1/qspg7yhr/`](./k1/qspg7yhr/) |
| n02-x1-invalid-did-padding | x1 | genesis sidecar | error `INVALID_DID` | [`x1/qjkr5ukn/`](./x1/qjkr5ukn/) |
| n03-k1-invalid-did-network-nibble | k1 | no update | error `INVALID_DID` | [`k1/qcp9e042/`](./k1/qcp9e042/) |
| n04-x1-genesis-hash-mismatch | x1 | genesis sidecar | error `INVALID_DID` | [`x1/qnr6s8cg/`](./x1/qnr6s8cg/) |
| n05-x1-missing-update-data | x1 | genesis sidecar, 1 update withheld | error `MISSING_UPDATE_DATA` | [`x1/qnwp673e/`](./x1/qnwp673e/) |
| n10-k1-invalid-update-context-member | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qsp3k0pq/`](./k1/qsp3k0pq/) |
| n11-k1-invalid-update-context-order | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qsp854zk/`](./k1/qsp854zk/) |
| n12-k1-invalid-update-proof-context | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qsptz2u9/`](./k1/qsptz2u9/) |
| n13-k1-invalid-update-capability-action | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspmajv6/`](./k1/qspmajv6/) |
| n14-k1-invalid-update-capability-encoding | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qsp9820e/`](./k1/qsp9820e/) |
| n15-k1-invalid-update-proof-purpose | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspqqxgv/`](./k1/qspqqxgv/) |
| n16-x1-invalid-update-unauthorized-method | x1 | genesis sidecar, 1 update sidecar | error `INVALID_DID_UPDATE` | [`x1/qjw0t0e4/`](./x1/qjw0t0e4/) |
| n17-k1-invalid-update-unknown-method | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qsp2x348/`](./k1/qsp2x348/) |
| n18-k1-invalid-update-proof-value | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspq6yml/`](./k1/qspq6yml/) |
| n19-k1-invalid-update-source-hash | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspk7udk/`](./k1/qspk7udk/) |
| n20-k1-invalid-update-target-hash | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspqn994/`](./k1/qspqn994/) |
| n21-k1-invalid-update-version-skip | k1 | 1 update sidecar | error `LATE_PUBLISHING_ERROR` | [`k1/qspmsf53/`](./k1/qspmsf53/) |
| n22-k1-invalid-update-patch-missing-path | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qsp622hd/`](./k1/qsp622hd/) |
| n23-k1-invalid-update-patch-changes-id | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspzu0kw/`](./k1/qspzu0kw/) |
| n24-k1-invalid-update-patch-invalid-document | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspf02mv/`](./k1/qspf02mv/) |
| n25-k1-invalid-update-created-after-block | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qsp8g0tw/`](./k1/qsp8g0tw/) |
| n26-k1-invalid-update-expires-before-mediantime | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspxna3u/`](./k1/qspxna3u/) |
| n27-k1-invalid-update-expires-before-created | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qspurjp5/`](./k1/qspurjp5/) |
| n28-k1-late-publishing | k1 | 2 updates sidecar | error `LATE_PUBLISHING_ERROR` | [`k1/qspe8u25/`](./k1/qspe8u25/) |

### n01-k1-invalid-did-checksum

Negative: the resolve input carries a k1 identifier with a wrong Bech32m checksum.

DID: `did:btcr2:k1qspg7yhraawy4kdd7denm4kp8prhqhc57xzf9r84vp6936fk9wufddcvlhpwq`

### n02-x1-invalid-did-padding

Negative: the resolve input carries an x1 identifier with non-zero Bech32m padding bits.

DID: `did:btcr2:x1qjkr5uknj298edvf46n4y9jskf5w02wwc7lz9vvw5twpssflz8khe2y6cft`

### n03-k1-invalid-did-network-nibble

Negative: the resolve input carries a k1 identifier whose network nibble is the reserved value 6.

DID: `did:btcr2:k1qcp9e042jyjqen2gv8tncj2uh0jjn09txlc7qu070272m6y5usanyxge9eqa9`

### n04-x1-genesis-hash-mismatch

Negative: the sidecar genesis document does not hash to the genesis bytes of the x1 identifier.

DID: `did:btcr2:x1qnr6s8cg0ljl9ql252jqezvh6rv3thgav37grw5xa0ehpx6gheycwhgxuh0`

### n05-x1-missing-update-data

Negative: x1 DID with 1 anchored update that is in neither the sidecar nor the CAS.

DID: `did:btcr2:x1qnwp673eqen450mtqs0yd6egjmav324764eqpzdsdnxyspnyq4f4xpmfeem`

### n10-k1-invalid-update-context-member

Negative: k1 DID with 1 anchored update; the update @context has a wrong last member.

DID: `did:btcr2:k1qsp3k0pqt3ekmdfs88cdajc435rw0xyfhxlwhtk2nuqs5vx3kj00s9s35fkkz`

### n11-k1-invalid-update-context-order

Negative: k1 DID with 1 anchored update; two members of the update @context are swapped.

DID: `did:btcr2:k1qsp854zkyhp323d7nj7q5xxqqw2wv6mvngl2pwjlcz0muqm98ezkllgwu8zuu`

### n12-k1-invalid-update-proof-context

Negative: k1 DID with 1 anchored update; the proof @context differs from the update @context.

DID: `did:btcr2:k1qsptz2u92e9089ll4hrcf4mez5esraut4kgg2qt3su46e4phsugtuqc4lt579`

### n13-k1-invalid-update-capability-action

Negative: k1 DID with 1 anchored update; proof.capabilityAction is Read.

DID: `did:btcr2:k1qspmajv6k52vl8p8nag7nvnz8f62we39xftn5nxuzl3x7t7rdrddsgshzjej7`

### n14-k1-invalid-update-capability-encoding

Negative: k1 DID with 1 anchored update; proof.capability carries the DID without percent-encoding.

DID: `did:btcr2:k1qsp9820e6rhygl2hs7a755nt3v0yh9xlkes77p9jttfyp3me5c2ters6t49xh`

### n15-k1-invalid-update-proof-purpose

Negative: k1 DID with 1 anchored update; proof.proofPurpose is assertionMethod.

DID: `did:btcr2:k1qspqqxgvhgwj4esjjr8yhv2hkg58npda9x67lxye5thzhp4xprffd6szgj2my`

### n16-x1-invalid-update-unauthorized-method

Negative: x1 DID with a second verification method #key-1 listed for authentication only; 1 anchored update signed with #key-1.

DID: `did:btcr2:x1qjw0t0e45uc8mtq2mwk45rv734yn3a6kxr6xtxwrrya5dswr8c6jjgl9qc8`

### n17-k1-invalid-update-unknown-method

Negative: k1 DID with 1 anchored update; proof.verificationMethod names a method the document does not contain.

DID: `did:btcr2:k1qsp2x3482yltp0mgjtw0sc7jvpnwm3twq7xc93ljg66vt390hmfzj3qu37euy`

### n18-k1-invalid-update-proof-value

Negative: k1 DID with 1 anchored update; one character of proof.proofValue differs.

DID: `did:btcr2:k1qspq6yml7hpze5fpwy6xwt29tx74jev9nesfd67shhdha9r0rjxcd3cwx62he`

### n19-k1-invalid-update-source-hash

Negative: k1 DID with 1 anchored update; sourceHash is not the hash of the current document.

DID: `did:btcr2:k1qspk7udktxfk8xz6uvlp5mxw27ljuaf4fh7r2gyd906lrmk6ct6206scq4s7d`

### n20-k1-invalid-update-target-hash

Negative: k1 DID with 1 anchored update; targetHash is not the hash of the patched document.

DID: `did:btcr2:k1qspqn9946vnm3n0g3rc4p84apvfx6s06w8av363dxk4g9k7cnmkvg2qd5dg3k`

### n21-k1-invalid-update-version-skip

Negative: k1 DID with 1 anchored update; targetVersionId skips a version, so a version is missing (late publishing).

DID: `did:btcr2:k1qspmsf53fsyt4f0t49pysgsr3vqqcqevf7jmajgtp4wjjcj60v2xx3s943yup`

### n22-k1-invalid-update-patch-missing-path

Negative: k1 DID with 1 anchored update whose patch removes a path that does not exist.

DID: `did:btcr2:k1qsp622hdjxe7jmme89k8p8dnv5pqepjlg4u9pq6glu7ags6hegf99lqnxr08r`

### n23-k1-invalid-update-patch-changes-id

Negative: k1 DID with 1 anchored update whose patch replaces the document id.

DID: `did:btcr2:k1qspzu0kw9ufrz5xh8y36felcd2cch0lgppfdrgjslxynv22syt4ydlg4x5c0z`

### n24-k1-invalid-update-patch-invalid-document

Negative: k1 DID with 1 anchored update whose patched document does not conform (the verification method type is not Multikey).

DID: `did:btcr2:k1qspf02mvzrlsu7shtf6cxvpqk0enehdk3xm89lqv5w8q9rn0enkqynqt8ss0t`

### n25-k1-invalid-update-created-after-block

Negative: k1 DID with 1 anchored update; proof.created is after the header time of the block.

DID: `did:btcr2:k1qsp8g0tw80tkzl5nu2fwsn2qdetjr5nk9qfxjkzj70ccvmwg0rpl6eqznzpu2`

### n26-k1-invalid-update-expires-before-mediantime

Negative: k1 DID with 1 anchored update; proof.expires is before the mediantime of the block.

DID: `did:btcr2:k1qspxna3u5peke5rgl7xf5ay5yz6u6rs58y349syxp924hdawzgy8yhg57na58`

### n27-k1-invalid-update-expires-before-created

Negative: k1 DID with 1 anchored update; proof.expires is before proof.created.

DID: `did:btcr2:k1qspurjp50njc35xxkaumgfepukhxjjevq89fpfg56r5vazef7w6m5ggczsfff`

### n28-k1-late-publishing

Negative: k1 DID with two different updates from version 1 to version 2, anchored at two beacons (late publishing).

DID: `did:btcr2:k1qspe8u25mdzxgg6n34a9ryukjf6pfp3kez8nm6af4x48nz6k6wxqezg0j222a`

