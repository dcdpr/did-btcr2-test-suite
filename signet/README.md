# did:btcr2 Signet Test Vectors

Live `did:btcr2` test vectors anchored on **signet** (about 10 minute blocks).

- **On-chain:** every update signal is an `OP_RETURN` on signet. Explorer: https://mempool.space/signet. Esplora REST API: `https://mempool.space/signet/api`.
- **CAS:** the genesis documents, signed updates, and CAS Announcement Maps that a vector delivers through the CAS are pinned on IPFS (CIDv1, raw codec, sha2-256) and retrievable from a public gateway.
- **Sidecar:** everything else, SMT proofs included, rides in the resolution options of `resolve/input.json`.

## Layout

Each vector set lives under `{k1|x1}/{hash}/`:

- `create/input.json`, `create/output.json`: the create operation.
- `update/input.json`, `update/output.json`: the update operation; `update/NN/` for a set with more than one update. `signingMaterial` is the secret key of the signer.
- `resolve/input.json`, `resolve/output.json`: the resolve operation with the sidecar; `resolve/NN/` for a sub-vector with resolution options.
- `other.json`: the keys and the genesis document.
- `signals.json` (a set with a Beacon Signal on the chain): one entry per signal with the update it commits to (`update` is the `update/NN/` number; `duplicate` marks a second signal of the same update in a later block), the beacon id, the address, the `txid`, the block height, hash, time, and `mediantime`, the signal bytes, and `recordedTip` (the chain tip height when the outputs were recorded). A cohort member records the shared signal with the cohort id and members. A cohort member with no update has no `update` member: the signal commits to no update of the DID.

## How to compare

- Take the inputs and produce your own outputs. The signed bytes of an update are not compared: BIP340 signing is randomized. Your signed update must verify and must resolve to the recorded document.
- `resolve/output.json` is the DID Resolution result. Compare `didDocument`, `didDocumentMetadata.versionId`, and `didDocumentMetadata.deactivated`. Compare `didDocumentMetadata.confirmations` as "at least the recorded value": it grows with the chain. `updated` is the header time of the block of the last applied update.
- A negative vector records `didResolutionMetadata.error` with the DID Resolution error code. Compare the code only; `errorMessage` is the text of this implementation.
- Resolution applies a beacon signal at six confirmations (the specification default). A resolve before that depth returns an earlier version.
- `signals.json` is what your signal discovery must find at the beacon addresses. Compare the `txid`, the block, and the signal bytes; a resolver that reads no chain can take the signals from the file.
- `recordedTip` pins the chain of a set. At that tip, each signal has `recordedTip - blockHeight + 1` confirmations, and each recorded `confirmations` is at least the recorded value.

## Positive vectors

| Scenario | Type | Delivery | Expected | Path |
|----------|------|----------|----------|------|
| 01-k1-base | k1 | no update | versionId 1 | [`k1/qyp62qtt/`](./k1/qyp62qtt/) |
| 02-k1-sidecar-update | k1 | 1 update sidecar | versionId 2 | [`k1/qyp5h7kz/`](./k1/qyp5h7kz/) |
| 03-x1-base | x1 | genesis sidecar | versionId 1 | [`x1/q9kv6m73/`](./x1/q9kv6m73/) |
| 04-x1-sidecar-update | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/q98uadmd/`](./x1/q98uadmd/) |
| 05-x1-no-beacon | x1 | genesis cas | versionId 1 | [`x1/q83d7crh/`](./x1/q83d7crh/) |
| 06-x1-cas-3-updates | x1 | genesis cas, 3 updates cas | versionId 4, deactivated | [`x1/q82utypv/`](./x1/q82utypv/) |
| 07-k1-sidecar-deactivate | k1 | 1 update sidecar | versionId 2, deactivated | [`k1/qyphkqcy/`](./k1/qyphkqcy/) |
| 08-x1-cas-update-deactivate | x1 | genesis cas, 2 updates cas | versionId 3, deactivated | [`x1/qxtpqesc/`](./x1/qxtpqesc/) |
| 09a-x1-cas-update-announcement | x1 | genesis cas, 1 update cas, announcement cas | versionId 2 | [`x1/qxs3zu4m/`](./x1/qxs3zu4m/) |
| 09b-x1-cas-update-announcement-paired | x1 | genesis cas, 1 update cas, announcement cas | versionId 2 | [`x1/qxunyl82/`](./x1/qxunyl82/) |
| 10a-x1-sidecar-update-cas-announcement | x1 | genesis sidecar, 1 update sidecar, announcement sidecar | versionId 2 | [`x1/q93l6dxt/`](./x1/q93l6dxt/) |
| 10b-x1-sidecar-update-cas-announcement-paired | x1 | genesis sidecar, 1 update sidecar, announcement sidecar | versionId 2 | [`x1/q9nhcz2z/`](./x1/q9nhcz2z/) |
| 11a-x1-cas-update-smt-proof | x1 | genesis cas, 1 update cas, SMT proof sidecar | versionId 2 | [`x1/q9wkxze8/`](./x1/q9wkxze8/) |
| 11b-x1-cas-update-smt-proof-paired | x1 | genesis cas, 1 update cas, SMT proof sidecar | versionId 2 | [`x1/qxsfdgg9/`](./x1/qxsfdgg9/) |
| 12a-x1-sidecar-update-smt-proof | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar | versionId 2 | [`x1/q8sxjrau/`](./x1/q8sxjrau/) |
| 12b-x1-sidecar-update-smt-proof-paired | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar | versionId 2 | [`x1/qxqfmajn/`](./x1/qxqfmajn/) |
| 13-k1-update-p2wpkh | k1 | 1 update sidecar | versionId 2 | [`k1/qypdscmf/`](./k1/qypdscmf/) |
| 14-k1-update-p2tr | k1 | 1 update sidecar | versionId 2 | [`k1/qyprgq5l/`](./k1/qyprgq5l/) |
| 15-x1-beacon-rotation | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/q9j6lwt5/`](./x1/q9j6lwt5/) |
| 16-x1-beacon-add-then-use | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qxkut6n0/`](./x1/qxkut6n0/) |
| 17-x1-vm-add-rotate-authentication | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qx6ld2rx/`](./x1/qx6ld2rx/) |
| 18-x1-embedded-invocation-key | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/q84gyrkg/`](./x1/q84gyrkg/) |
| 19-x1-relative-ids | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/q99qwj9u/`](./x1/q99qwj9u/) |
| 20-k1-cas-update | k1 | 1 update cas | versionId 2 | [`k1/qypda5tj/`](./k1/qypda5tj/) |
| 21-k1-deactivate-then-update | k1 | 2 updates sidecar | versionId 2, deactivated | [`k1/qyplj6cu/`](./k1/qyplj6cu/) |
| 22-x1-three-updates-resolution-options | x1 | genesis sidecar, 3 updates sidecar | versionId 4 | [`x1/q8y5x5f3/`](./x1/q8y5x5f3/) |
| 23-k1-duplicate-signal | k1 | 2 updates sidecar, 1 duplicate signal | versionId 3 | [`k1/qypjcajt/`](./k1/qypjcajt/) |
| 24-k1-removed-beacon-signal | k1 | 2 updates sidecar | versionId 2 | [`k1/qypws7tm/`](./k1/qypws7tm/) |
| 25a-x1-smt-update-no-nonce | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar, no nonce | versionId 2 | [`x1/q8wpt5qu/`](./x1/q8wpt5qu/) |
| 25b-x1-smt-nonce-no-update | x1 | genesis sidecar, no update, SMT proof sidecar | versionId 1 | [`x1/q9pspvd9/`](./x1/q9pspvd9/) |
| 25c-x1-smt-empty-index | x1 | genesis sidecar, no update, SMT proof sidecar, no nonce | versionId 1 | [`x1/qy0glluz/`](./x1/qy0glluz/) |
| 26-k1-signal-below-current-height | k1 | 2 updates sidecar | versionId 2 | [`k1/qyphftn0/`](./k1/qyphftn0/) |

### 01-k1-base

Base resolution: k1 (key-type) DID with default 3 singleton beacons, no updates. Mirrors danubetech example 1.

DID: `did:btcr2:k1qyp62qttvcs2kaku8m02gq8zeqhslcg5tp27nh0awfnj5jf5ffrns5c7gsvmx`

### 02-k1-sidecar-update

k1 (key-type) DID with default 3 singleton beacons, 1 sidecar update adding a DIDCommMessaging service. Mirrors danubetech example 2.

DID: `did:btcr2:k1qyp5h7kz6jtdqwmyun3n7rkfe9t84xefez6pcyrzhu279luh4z3h5fq7qf88q`

### 03-x1-base

Base resolution: x1 (external-type) DID with default 3 singleton beacons, no updates. Genesis document delivered via sidecar. Mirrors danubetech example 3.

DID: `did:btcr2:x1q9kv6m73y2xhchg7dd4dhlfvl45dyyvnrxjdyfptnuudvvhgwz0zkj6v685`

### 04-x1-sidecar-update

x1 (external-type) DID with default 3 singleton beacons, 1 sidecar update adding a DecentralizedWebNode service. Mirrors danubetech example 4.

DID: `did:btcr2:x1q98uadmd2ygyfe48mmsdhj0yjr40ws9h56er49jhr8zsu7lzztdrq2gvxkn`

### 05-x1-no-beacon

Base resolution: x1 (external-type) DID with NO beacon services in the document, no updates. Genesis doc delivered via sidecar. Mirrors danubetech example 5 (CAS-delivered genesis); our vector uses sidecar delivery so the vector is self-contained.

DID: `did:btcr2:x1q83d7crha649qwnl72pcy5rqfz2tpyu6mexlrkm048r48x5ef3537jckd5m`

### 06-x1-cas-3-updates

x1 DID with default 3 singleton beacons, 3 updates delivered via CAS (didcomm, dwn, then deactivate). Mirrors danubetech example 6.

DID: `did:btcr2:x1q82utypv6fvp0f3m99dd7k4cvt6yj40y2xe4h5j9fhl9rpfmx6dfzy7jg2z`

### 07-k1-sidecar-deactivate

k1 (key-type) DID with default 3 singleton beacons, 1 sidecar update deactivating the DID. Mirrors danubetech example 7.

DID: `did:btcr2:k1qyphkqcy5m0znq3nyux4puxl2ewu9gqau8cds4d7h4xzyd78ff4zjwgxu3phw`

### 08-x1-cas-update-deactivate

x1 DID with default 3 singleton beacons, 1 CAS update (add didcomm) + CAS deactivate. Mirrors danubetech example 8.

DID: `did:btcr2:x1qxtpqesct5xh68a5da3enf9568tt65ju7kr0v2wl58uplrpl3ulc7rrsney`

### 09a-x1-cas-update-announcement

x1 DID with default 3 singleton beacons, 1 CAS update + CAS Announcement Map. Mirrors danubetech example 9a (paired with 09b in danubetech's tree).

DID: `did:btcr2:x1qxs3zu4mmjtjgtjtrx3a08e4fcj3m7ylewy7yv7x60lunltt50vtgzsc57a`

### 09b-x1-cas-update-announcement-paired

x1 DID with default 3 singleton beacons, 1 CAS update + CAS Announcement Map. Mirrors danubetech example 9b (paired with 09a in danubetech's tree).

DID: `did:btcr2:x1qxunyl8237vshu9mz0vakss2nvskdxvhytknzu675hq9s6rj6ruljhqpr8n`

### 10a-x1-sidecar-update-cas-announcement

x1 DID with default 3 singleton beacons + dedicated CASBeacon, 1 sidecar update. CAS Announcement delivered via sidecar. Mirrors danubetech example 10a.

DID: `did:btcr2:x1q93l6dxt2p29q9vzj2e980yt22p84z77zp2kfmql6h4jp3242c6evut822s`

### 10b-x1-sidecar-update-cas-announcement-paired

x1 DID with default 3 singleton beacons + dedicated CASBeacon, 1 sidecar update adding DIDCommMessaging. CAS Announcement delivered via sidecar. Mirrors danubetech example 10b (paired with 10a in danubetech's tree).

DID: `did:btcr2:x1q9nhcz2zs9w9m5y7n2wm8wevk0he3vs6wvqgrlruzp60p9uz77jqcshswn5`

### 11a-x1-cas-update-smt-proof

x1 DID with default 3 singleton beacons, 1 CAS update + SMT Proof (sidecar). Mirrors danubetech example 11a (paired with 11b in danubetech's tree).

DID: `did:btcr2:x1q9wkxze8zaylfsz806utfy8e54gu598d9u5mu3tvwdnam7v9kfp8gr848ke`

### 11b-x1-cas-update-smt-proof-paired

x1 DID with default 3 singleton beacons, 1 CAS update + SMT Proof (sidecar). Mirrors danubetech example 11b (paired with 11a in danubetech's tree).

DID: `did:btcr2:x1qxsfdgg9j5r8v85sqe4xhjtkjzvjdgjzcvvj96rgqcwu9hsdvfdtsk96aah`

### 12a-x1-sidecar-update-smt-proof

x1 DID with default 3 singleton beacons + dedicated SMTBeacon, 1 sidecar update. SMT Proof delivered via sidecar. Mirrors danubetech example 12a (paired with 12b in danubetech's tree).

DID: `did:btcr2:x1q8sxjrauhntq8ff2ad927w6jskweekuays34xycu2fsut005mn77wa4q9z3`

### 12b-x1-sidecar-update-smt-proof-paired

x1 DID with default 3 singleton beacons + dedicated SMTBeacon, 1 sidecar update adding DWN. SMT Proof delivered via sidecar. Mirrors danubetech example 12b (paired with 12a in danubetech's tree).

DID: `did:btcr2:x1qxqfmajnpsh6elpm9wt6pjqvl55gdrnzg5yvn5rftd7rghm28j8dyrt2y4k`

### 13-k1-update-p2wpkh

k1 DID with the default 3 singleton beacons, 1 sidecar update anchored at the #initialP2WPKH beacon.

DID: `did:btcr2:k1qypdscmf7s7ef09rv0qhrzfktkhft2ajldyz5d7639hlect7e8gcm5q23d9me`

### 14-k1-update-p2tr

k1 DID with the default 3 singleton beacons, 1 sidecar update anchored at the #initialP2TR beacon.

DID: `did:btcr2:k1qyprgq5l4k7d6j3pgq7ynmgltggja238h9h0m2aa43343adac48mh5qd40m25`

### 15-x1-beacon-rotation

x1 DID; update 1 replaces the endpoint of the #initialP2PKH singleton beacon with a new address (beacon rotation); update 2 is anchored at the rotated beacon.

DID: `did:btcr2:x1q9j6lwt5nq5q56quce694luurd49zhca63ldzzf5acemcuj5nawj275n7ea`

### 16-x1-beacon-add-then-use

x1 DID; update 1 adds a fourth singleton beacon #newBeacon; update 2 is anchored at #newBeacon, so the resolver needs a second discovery round.

DID: `did:btcr2:x1qxkut6n0n0gvkf9f6u5ajeyd7twashs5yg8e38cg7eehq2mz48rxcfafsgh`

### 17-x1-vm-add-rotate-authentication

x1 DID; update 1 adds the verification method #key-1, replaces authentication with it, and adds it to capabilityInvocation; update 2 is signed with #key-1.

DID: `did:btcr2:x1qx6ld2rxwml5y4ahpwe9d5nmp7xlw4pauecw6qnp6fvaud3c08avw3yf62d`

### 18-x1-embedded-invocation-key

x1 DID whose capabilityInvocation embeds the initial verification method as an object; verificationMethod is empty. 1 sidecar update signed with the embedded method.

DID: `did:btcr2:x1q84gyrkggd3759z9syj3aeewk8rdlwsfaa95kf38jv8ej2aylyfjkahta4n`

### 19-x1-relative-ids

x1 DID whose genesis document spells every id as a relative DID URL (#initialKey, #initialP2PKH). 1 sidecar update.

DID: `did:btcr2:x1q99qwj9umyetasrrcrxj8cuhsjzkj7z6uf0dx3ahtcmq8grx35g8u6fldxr`

### 20-k1-cas-update

k1 DID with the default 3 singleton beacons, 1 update delivered through the CAS (no sidecar).

DID: `did:btcr2:k1qypda5tj07vn65yhyly00t8879laup7dgf8nm52mahnax38ke939srs7wthpd`

### 21-k1-deactivate-then-update

k1 DID; update 1 deactivates the DID; update 2 is a later anchored update that a resolver must not apply. Resolves to version 2, deactivated; version 3 does not exist.

DID: `did:btcr2:k1qyplj6cu65rh6ejq9ssx86ftjaqf4v0jf25ay59qwmrm92wxhstvy5sw6py47`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

### 22-x1-three-updates-resolution-options

x1 DID with 3 sidecar updates, each anchored in its own block. The sub-vectors cover versionId, versionTime (before the first update, between the first and the second, at the second), an unreachable versionId, both options, and minConf above the chain depth.

DID: `did:btcr2:x1q8y5x5f35cw5jc278nqhaqnu8jlza4aa6glszzscp9sgavwkgx596d3vfs2`

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

DID: `did:btcr2:k1qypjcajtptruvwylxszx20zmgw3350p5vxh360e0hq48v7g539uq5ncr7fm29`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionTime":"before:3"}` | versionId 3 |

### 24-k1-removed-beacon-signal

k1 DID; update 1 removes the #initialP2PKH beacon service (version 2); update 2 is announced at the removed #initialP2PKH address and adds a service. A resolver ignores the signal of a beacon address that the current document does not carry (Process Next Update, step 4): resolves to version 2, and version 3 does not exist.

DID: `did:btcr2:k1qypws7tm5j3hp3tzs093hgjhf3udepew8jk08xkx3hk2hvhdpd70d0qcn6xs7`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

### 25a-x1-smt-update-no-nonce

x1 DID in the SMT cohort smt-25 with 1 sidecar update. The tree entry has no nonce, so the leaf value is the updateId.

DID: `did:btcr2:x1q8wpt5qush3tpm9m4estslk3gs6vqys52mkutzvg3ckwlm66hhakuuvphz2`

### 25b-x1-smt-nonce-no-update

x1 DID in the SMT cohort smt-25 with no update. The tree entry has a nonce and no updateId, so the leaf value is hash(hash(nonce)). The proof announces no update, so the DID resolves to version 1.

DID: `did:btcr2:x1q9pspvd9waez6wddkm39ztwty6ygqup0k0fzd82h0dw89kng4z8s2y46mye`

### 25c-x1-smt-empty-index

x1 DID in the SMT cohort smt-25 with no update and no nonce. The tree has no entry for the DID, so the proof is the proof of an empty index. The DID resolves to version 1.

DID: `did:btcr2:x1qy0glluzajpjwsr0tuthpv7m7k8nru5hck53muaj50wcxwfw03hgxud8pr6`

### 26-k1-signal-below-current-height

k1 DID; update 1 adds the beacon #lateBeacon and is anchored in round 2. Update 2 is anchored at #lateBeacon in round 1, one block before update 1. The signal is below current_block_height, so a resolver ignores it. The sub-vector asks for version 3 and expects NOT_FOUND.

DID: `did:btcr2:k1qyphftn050vfx0xy55ch6w9etarzwt6dtetdrcwgvv6hykvfxqsjw5cztmh8a`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

## Negative vectors

| Scenario | Type | Delivery | Expected | Path |
|----------|------|----------|----------|------|
| n01-k1-invalid-did-checksum | k1 | no update | error `INVALID_DID` | [`k1/qyps5h33/`](./k1/qyps5h33/) |
| n02-x1-invalid-did-padding | x1 | genesis sidecar | error `INVALID_DID` | [`x1/q9k9vxhx/`](./x1/q9k9vxhx/) |
| n03-k1-invalid-did-network-nibble | k1 | no update | error `INVALID_DID` | [`k1/qcpszaex/`](./k1/qcpszaex/) |
| n04-x1-genesis-hash-mismatch | x1 | genesis sidecar | error `INVALID_DID` | [`x1/q9nuqr6s/`](./x1/q9nuqr6s/) |
| n05-x1-missing-update-data | x1 | genesis sidecar, 1 update withheld | error `MISSING_UPDATE_DATA` | [`x1/qyzmtprn/`](./x1/qyzmtprn/) |
| n10-k1-invalid-update-context-member | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qyp527dr/`](./k1/qyp527dr/) |
| n11-k1-invalid-update-context-order | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qyp2ju95/`](./k1/qyp2ju95/) |
| n12-k1-invalid-update-proof-context | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypc0v9c/`](./k1/qypc0v9c/) |
| n13-k1-invalid-update-capability-action | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qyphxahc/`](./k1/qyphxahc/) |
| n14-k1-invalid-update-capability-encoding | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypkdal6/`](./k1/qypkdal6/) |
| n15-k1-invalid-update-proof-purpose | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qyp7s8n5/`](./k1/qyp7s8n5/) |
| n16-x1-invalid-update-unauthorized-method | x1 | genesis sidecar, 1 update sidecar | error `INVALID_DID_UPDATE` | [`x1/q9rxnv97/`](./x1/q9rxnv97/) |
| n17-k1-invalid-update-unknown-method | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qype9x7f/`](./k1/qype9x7f/) |
| n18-k1-invalid-update-proof-value | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypdef8c/`](./k1/qypdef8c/) |
| n19-k1-invalid-update-source-hash | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qyp3yvm3/`](./k1/qyp3yvm3/) |
| n20-k1-invalid-update-target-hash | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypp2qva/`](./k1/qypp2qva/) |
| n21-k1-invalid-update-version-skip | k1 | 1 update sidecar | error `LATE_PUBLISHING_ERROR` | [`k1/qyp0nl7q/`](./k1/qyp0nl7q/) |
| n22-k1-invalid-update-patch-missing-path | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypljzfn/`](./k1/qypljzfn/) |
| n23-k1-invalid-update-patch-changes-id | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypcaw4m/`](./k1/qypcaw4m/) |
| n24-k1-invalid-update-patch-invalid-document | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypxr0l9/`](./k1/qypxr0l9/) |
| n25-k1-invalid-update-created-after-block | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qyph2fjv/`](./k1/qyph2fjv/) |
| n26-k1-invalid-update-expires-before-mediantime | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypxn45q/`](./k1/qypxn45q/) |
| n27-k1-invalid-update-expires-before-created | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qypyl83s/`](./k1/qypyl83s/) |
| n28-k1-late-publishing | k1 | 2 updates sidecar | error `LATE_PUBLISHING_ERROR` | [`k1/qypv877a/`](./k1/qypv877a/) |
| n29-x1-smt-proof-hash | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar (`hash` changed) | error `INVALID_SIGNAL_DATA` | [`x1/qxrwycar/`](./x1/qxrwycar/) |
| n30-x1-smt-proof-root-id | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar (`id` changed) | error `MISSING_UPDATE_DATA` | [`x1/qyuvshka/`](./x1/qyuvshka/) |
| n31-x1-smt-proof-withheld | x1 | genesis sidecar, 1 update sidecar, no SMT proof | error `MISSING_UPDATE_DATA` | [`x1/qxvg5h46/`](./x1/qxvg5h46/) |

### n01-k1-invalid-did-checksum

Negative: the resolve input carries a k1 identifier with a wrong Bech32m checksum.

DID: `did:btcr2:k1qyps5h33rz9gz65cqj7rr2clssp64wrleuyjw5p664dmasdttg6gc7ss6fzzq`

### n02-x1-invalid-did-padding

Negative: the resolve input carries an x1 identifier with non-zero Bech32m padding bits.

DID: `did:btcr2:x1q9k9vxhx2p8pskyz8kksszd9u8y5j8qclzhhvewuqw2uvwl6ewqamscgcmj`

### n03-k1-invalid-did-network-nibble

Negative: the resolve input carries a k1 identifier whose network nibble is the reserved value 6.

DID: `did:btcr2:k1qcpszaexycatrzkcclaxcwqsm03382c4m837g6eq3afzfg85fyym8qq59upan`

### n04-x1-genesis-hash-mismatch

Negative: the sidecar genesis document does not hash to the genesis bytes of the x1 identifier.

DID: `did:btcr2:x1q9nuqr6s45cefydrefvujcy6sj4d6frc5gyuxzmn07xe2au50u39zm4lvzg`

### n05-x1-missing-update-data

Negative: x1 DID with 1 anchored update that is in neither the sidecar nor the CAS.

DID: `did:btcr2:x1qyzmtprn8rzs07nlfg0ex82n5fcpqcrjr304ysdxj5crdvsk04gpgjtk49x`

### n10-k1-invalid-update-context-member

Negative: k1 DID with 1 anchored update; the update @context has a wrong last member.

DID: `did:btcr2:k1qyp527drq8afc6jffc9ndmzvzw90kendr58p2ggjep4xsk8y5tg0yxq0gpzm9`

### n11-k1-invalid-update-context-order

Negative: k1 DID with 1 anchored update; two members of the update @context are swapped.

DID: `did:btcr2:k1qyp2ju952zdgpe305hghjcs6l4ypdg9zrzhtuxuqcqa2hq8wgcxlmscd86x37`

### n12-k1-invalid-update-proof-context

Negative: k1 DID with 1 anchored update; the proof @context differs from the update @context.

DID: `did:btcr2:k1qypc0v9c9ykelxpensp3fcsmwzvemsteqkmcccnhk73t4fjxxrt4u9s6e9s6g`

### n13-k1-invalid-update-capability-action

Negative: k1 DID with 1 anchored update; proof.capabilityAction is Read.

DID: `did:btcr2:k1qyphxahcy9w386x0erjpmtwjw7y6vgd7aexcmdjxecdjzkf3wyzlrjckwcvlk`

### n14-k1-invalid-update-capability-encoding

Negative: k1 DID with 1 anchored update; proof.capability carries the DID without percent-encoding.

DID: `did:btcr2:k1qypkdal6kw7tpky2t5ph0wsf5v2v5svzn5s2nktqk376rx2nwt3kyxsx489kf`

### n15-k1-invalid-update-proof-purpose

Negative: k1 DID with 1 anchored update; proof.proofPurpose is assertionMethod.

DID: `did:btcr2:k1qyp7s8n52kmzdjef24pnq6m6q5qhpks2ls9dp3p4pd8v32fy9xalyvge8hqgw`

### n16-x1-invalid-update-unauthorized-method

Negative: x1 DID with a second verification method #key-1 listed for authentication only; 1 anchored update signed with #key-1.

DID: `did:btcr2:x1q9rxnv9723u4c4w8rmhad04at9ypde99q5j2t3l66rqj5784avuawmljxvf`

### n17-k1-invalid-update-unknown-method

Negative: k1 DID with 1 anchored update; proof.verificationMethod names a method the document does not contain.

DID: `did:btcr2:k1qype9x7f2tfqs8p6m3t2s25h5cdmquaezn4p97tjacxspuc4p2ag53chrsv2v`

### n18-k1-invalid-update-proof-value

Negative: k1 DID with 1 anchored update; one character of proof.proofValue differs.

DID: `did:btcr2:k1qypdef8c9lgmfudfrzckpw863jjrvh4ygp6y99yzmqezwapk8reyuyqxkxr4j`

### n19-k1-invalid-update-source-hash

Negative: k1 DID with 1 anchored update; sourceHash is not the hash of the current document.

DID: `did:btcr2:k1qyp3yvm36yddzkstdnlphxc3n5lvle7wsy3dnhqzgycfpnfarhcgxdsgvdv82`

### n20-k1-invalid-update-target-hash

Negative: k1 DID with 1 anchored update; targetHash is not the hash of the patched document.

DID: `did:btcr2:k1qypp2qvacsjzfuww3vxn25t0fnuf6z9zxuyhax5eaehytg9tstrgzkcc7mkp8`

### n21-k1-invalid-update-version-skip

Negative: k1 DID with 1 anchored update; targetVersionId skips a version, so a version is missing (late publishing).

DID: `did:btcr2:k1qyp0nl7q0y52mqmrlej7ghla2t5t3dqgngrphy70y5fa96ep3mywm5su2xxf9`

### n22-k1-invalid-update-patch-missing-path

Negative: k1 DID with 1 anchored update whose patch removes a path that does not exist.

DID: `did:btcr2:k1qypljzfnqdv343t0zufeyfphpumav2tmxvzu87pm5jca4ly00gs0n4g0tsvq4`

### n23-k1-invalid-update-patch-changes-id

Negative: k1 DID with 1 anchored update whose patch replaces the document id.

DID: `did:btcr2:k1qypcaw4m3qwwhah2s4k6yg89efxtk70fx3sg22ccyfphzedktvc4usqa6ldfk`

### n24-k1-invalid-update-patch-invalid-document

Negative: k1 DID with 1 anchored update whose patched document does not conform (the verification method type is not Multikey).

DID: `did:btcr2:k1qypxr0l9lwdndqwx8ymdrzkze7c7rkjseedxmk4kqks4jnu3t3cdyhc8dhpf8`

### n25-k1-invalid-update-created-after-block

Negative: k1 DID with 1 anchored update; proof.created is after the header time of the block.

DID: `did:btcr2:k1qyph2fjvxy9gddagyxsketux0wzqje3hquhzqfcpzua8dw4gqkjq6msh7a6ea`

### n26-k1-invalid-update-expires-before-mediantime

Negative: k1 DID with 1 anchored update; proof.expires is before the mediantime of the block.

DID: `did:btcr2:k1qypxn45qnzrkt5rmx9wvupy2p4n3g86vt2ps68qvjc7xxjz8j9gw22g069shp`

### n27-k1-invalid-update-expires-before-created

Negative: k1 DID with 1 anchored update; proof.expires is before proof.created.

DID: `did:btcr2:k1qypyl83sytn85uk5544vchxg547fyadsp82cg42psuxg8wr2j65rycqc58eua`

### n28-k1-late-publishing

Negative: k1 DID with two different updates from version 1 to version 2, anchored at two beacons (late publishing).

DID: `did:btcr2:k1qypv877ay48akw53pc2kk3p6rv9l9rsd4pyj6fxfzmufa8t03lnm5dcykxjxs`

### n29-x1-smt-proof-hash

Negative: x1 DID in the SMT cohort smt-25 with 1 sidecar update; one byte of the first entry of hashes in the sidecar proof differs.

DID: `did:btcr2:x1qxrwycarfgnplgal08kk479vfs4t3cx5wmxkcjfqxqp2amh8f5xe7vd7npp`

### n30-x1-smt-proof-root-id

Negative: x1 DID in the SMT cohort smt-25 with 1 sidecar update; the id of the sidecar proof is not the signal root. A resolver finds proofs by id, so the sidecar has no proof for the signal.

DID: `did:btcr2:x1qyuvshka7ay69lj9ex2gcvyjupy7g7mc9gh03q9rx2lalgm9wpdt68cy7f5`

### n31-x1-smt-proof-withheld

Negative: x1 DID in the SMT cohort smt-25 with 1 sidecar update; the sidecar holds no SMT proof.

DID: `did:btcr2:x1qxvg5h46yn29wgx6dmzzfswwtlccwpdtkg524sp507c5dgzce2525cxed4h`

