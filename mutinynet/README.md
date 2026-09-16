# did:btcr2 Mutinynet Test Vectors

Live `did:btcr2` test vectors anchored on **Mutinynet** (a custom signet with 30 second blocks).

- **On-chain:** every update signal is an `OP_RETURN` on Mutinynet. Explorer: https://mutinynet.com. Esplora REST API: `https://mutinynet.com/api`.
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
| 01-k1-base | k1 | no update | versionId 1 | [`k1/q5pp6jm6/`](./k1/q5pp6jm6/) |
| 02-k1-sidecar-update | k1 | 1 update sidecar | versionId 2 | [`k1/q5pqhkks/`](./k1/q5pqhkks/) |
| 03-x1-base | x1 | genesis sidecar | versionId 1 | [`x1/q498tj8r/`](./x1/q498tj8r/) |
| 04-x1-sidecar-update | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/q4typvtp/`](./x1/q4typvtp/) |
| 05-x1-no-beacon | x1 | genesis cas | versionId 1 | [`x1/q5rlsadq/`](./x1/q5rlsadq/) |
| 06-x1-cas-3-updates | x1 | genesis cas, 3 updates cas | versionId 4, deactivated | [`x1/q5mp6cxq/`](./x1/q5mp6cxq/) |
| 07-k1-sidecar-deactivate | k1 | 1 update sidecar | versionId 2, deactivated | [`k1/q5p9uafd/`](./k1/q5p9uafd/) |
| 08-x1-cas-update-deactivate | x1 | genesis cas, 2 updates cas | versionId 3, deactivated | [`x1/q5k5xn0z/`](./x1/q5k5xn0z/) |
| 09a-x1-cas-update-announcement | x1 | genesis cas, 1 update cas, announcement cas | versionId 2 | [`x1/q5kssq8u/`](./x1/q5kssq8u/) |
| 09b-x1-cas-update-announcement-paired | x1 | genesis cas, 1 update cas, announcement cas | versionId 2 | [`x1/q4u560pr/`](./x1/q4u560pr/) |
| 10a-x1-sidecar-update-cas-announcement | x1 | genesis sidecar, 1 update sidecar, announcement sidecar | versionId 2 | [`x1/qhem7zpy/`](./x1/qhem7zpy/) |
| 10b-x1-sidecar-update-cas-announcement-paired | x1 | genesis sidecar, 1 update sidecar, announcement sidecar | versionId 2 | [`x1/q52dx36q/`](./x1/q52dx36q/) |
| 13-k1-update-p2wpkh | k1 | 1 update sidecar | versionId 2 | [`k1/q5p08ynf/`](./k1/q5p08ynf/) |
| 14-k1-update-p2tr | k1 | 1 update sidecar | versionId 2 | [`k1/q5p8svrz/`](./k1/q5p8svrz/) |
| 15-x1-beacon-rotation | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/q5rp7phe/`](./x1/q5rp7phe/) |
| 16-x1-beacon-add-then-use | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qk58te4e/`](./x1/qk58te4e/) |
| 17-x1-vm-add-rotate-authentication | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qkj4a5xu/`](./x1/qkj4a5xu/) |
| 18-x1-embedded-invocation-key | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/q4nw2tdl/`](./x1/q4nw2tdl/) |
| 19-x1-relative-ids | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/q5pzvvxz/`](./x1/q5pzvvxz/) |
| 20-k1-cas-update | k1 | 1 update cas | versionId 2 | [`k1/q5pj0t23/`](./k1/q5pj0t23/) |
| 21-k1-deactivate-then-update | k1 | 2 updates sidecar | versionId 2, deactivated | [`k1/q5pzmjfx/`](./k1/q5pzmjfx/) |
| 22-x1-three-updates-resolution-options | x1 | genesis sidecar, 3 updates sidecar | versionId 4 | [`x1/qhfjzym7/`](./x1/qhfjzym7/) |
| 23-k1-duplicate-signal | k1 | 2 updates sidecar, 1 duplicate signal | versionId 3 | [`k1/q5p97uqz/`](./k1/q5p97uqz/) |
| 24-k1-removed-beacon-signal | k1 | 2 updates sidecar | versionId 2 | [`k1/q5paaduz/`](./k1/q5paaduz/) |

### 01-k1-base

Base resolution: k1 (key-type) DID with default 3 singleton beacons, no updates. Mirrors danubetech example 1.

DID: `did:btcr2:k1q5pp6jm68c6nd0kjd2vpnyq78tw0nyz65ae925zvjun587dpemghe4shg7ra5`

### 02-k1-sidecar-update

k1 (key-type) DID with default 3 singleton beacons, 1 sidecar update adding a DIDCommMessaging service. Mirrors danubetech example 2.

DID: `did:btcr2:k1q5pqhkks8486gcw8fv4aj2wmmxzw252ys6csmzmtxr86tr8uljl60ps5yth0w`

### 03-x1-base

Base resolution: x1 (external-type) DID with default 3 singleton beacons, no updates. Genesis document delivered via sidecar. Mirrors danubetech example 3.

DID: `did:btcr2:x1q498tj8rxtga9gjh2huwejt0w4ag4wauvjx2u93u5s2r6vhatvmw7gqjqzx`

### 04-x1-sidecar-update

x1 (external-type) DID with default 3 singleton beacons, 1 sidecar update adding a DecentralizedWebNode service. Mirrors danubetech example 4.

DID: `did:btcr2:x1q4typvtplw0u68kwsk0ny258u6q5acx5453va5yddkps6l085c0nzwmpsvr`

### 05-x1-no-beacon

Base resolution: x1 (external-type) DID with NO beacon services in the document, no updates. Genesis doc delivered via sidecar. Mirrors danubetech example 5 (CAS-delivered genesis); our vector uses sidecar delivery so the vector is self-contained.

DID: `did:btcr2:x1q5rlsadqkdafrsmpp67y5uvpw277ydjj4sujj8g2489m7dp3n7y0s2nwxlz`

### 06-x1-cas-3-updates

x1 DID with default 3 singleton beacons, 3 updates delivered via CAS (didcomm, dwn, then deactivate). Mirrors danubetech example 6.

DID: `did:btcr2:x1q5mp6cxq4avt6mapnutxyelfuvyg2gxt46hxpkfq236kxskp9ss7c3sz6gg`

### 07-k1-sidecar-deactivate

k1 (key-type) DID with default 3 singleton beacons, 1 sidecar update deactivating the DID. Mirrors danubetech example 7.

DID: `did:btcr2:k1q5p9uafdl72pvc4nhl8em9z0ydslj7grz8cqyswt99c2nvxsrjrnhngftmpun`

### 08-x1-cas-update-deactivate

x1 DID with default 3 singleton beacons, 1 CAS update (add didcomm) + CAS deactivate. Mirrors danubetech example 8.

DID: `did:btcr2:x1q5k5xn0ztf2q3vw5qs7cj4jv9g5rd774rjywh8955u074epaeaatw6agr8j`

### 09a-x1-cas-update-announcement

x1 DID with default 3 singleton beacons, 1 CAS update + CAS Announcement Map. Mirrors danubetech example 9a (paired with 09b in danubetech's tree).

DID: `did:btcr2:x1q5kssq8u30w5eual99ss69cknd0y2qesdeythl5tgackgy87nulzq0l5lh4`

### 09b-x1-cas-update-announcement-paired

x1 DID with default 3 singleton beacons, 1 CAS update + CAS Announcement Map. Mirrors danubetech example 9b (paired with 09a in danubetech's tree).

DID: `did:btcr2:x1q4u560prehr04dpjpvyrnnrqkhhvthu0vqheke3c25klfvxynnmuxcxpaa4`

### 10a-x1-sidecar-update-cas-announcement

x1 DID with default 3 singleton beacons + dedicated CASBeacon, 1 sidecar update. CAS Announcement delivered via sidecar. Mirrors danubetech example 10a.

DID: `did:btcr2:x1qhem7zpyek2zlmgpzafggwzztytvj7kzh2cpzu479r4d75dmu02zkmw29ka`

### 10b-x1-sidecar-update-cas-announcement-paired

x1 DID with default 3 singleton beacons + dedicated CASBeacon, 1 sidecar update adding DIDCommMessaging. CAS Announcement delivered via sidecar. Mirrors danubetech example 10b (paired with 10a in danubetech's tree).

DID: `did:btcr2:x1q52dx36qyfpxkald3dmz32vufm8n5z4dzsf92q3dzsxt8m7sqylcvknl37x`

### 13-k1-update-p2wpkh

k1 DID with the default 3 singleton beacons, 1 sidecar update anchored at the #initialP2WPKH beacon.

DID: `did:btcr2:k1q5p08ynfqe6vfxevtz9tzcx675m8gzc7u7g3gesw8tqzdjw990x6zyclsqwn9`

### 14-k1-update-p2tr

k1 DID with the default 3 singleton beacons, 1 sidecar update anchored at the #initialP2TR beacon.

DID: `did:btcr2:k1q5p8svrzjd2yuw40adkpjtfs88ffx89wjk8vph6hxjecq85uf2xcthqx8ta75`

### 15-x1-beacon-rotation

x1 DID; update 1 replaces the endpoint of the #initialP2PKH singleton beacon with a new address (beacon rotation); update 2 is anchored at the rotated beacon.

DID: `did:btcr2:x1q5rp7phec5m3argcn3cd56hturnf3khhg77jtatnh0tad5zw58qvueynzwy`

### 16-x1-beacon-add-then-use

x1 DID; update 1 adds a fourth singleton beacon #newBeacon; update 2 is anchored at #newBeacon, so the resolver needs a second discovery round.

DID: `did:btcr2:x1qk58te4eu8lewrks6zmaa0uux2af9hwkpem8gad7twe3v50ff38julf2tj5`

### 17-x1-vm-add-rotate-authentication

x1 DID; update 1 adds the verification method #key-1, replaces authentication with it, and adds it to capabilityInvocation; update 2 is signed with #key-1.

DID: `did:btcr2:x1qkj4a5xu70hrzymaqg47aeupxjuq5p6hsrvdy5hdzthryer2aqp075scyrt`

### 18-x1-embedded-invocation-key

x1 DID whose capabilityInvocation embeds the initial verification method as an object; verificationMethod is empty. 1 sidecar update signed with the embedded method.

DID: `did:btcr2:x1q4nw2tdlfcctnshvfcjnjq7f5mgaem3cjup2dusn00m5q02vym3s6pxhn95`

### 19-x1-relative-ids

x1 DID whose genesis document spells every id as a relative DID URL (#initialKey, #initialP2PKH). 1 sidecar update.

DID: `did:btcr2:x1q5pzvvxz42epmdh6km4j0svrtaqml74huyrm7xjj75lmspkjmx6kg3k4wc8`

### 20-k1-cas-update

k1 DID with the default 3 singleton beacons, 1 update delivered through the CAS (no sidecar).

DID: `did:btcr2:k1q5pj0t23nnlqdd72007acs2j6ql78rwae2h2mzp3a867llwz3sadv5qul5uzk`

### 21-k1-deactivate-then-update

k1 DID; update 1 deactivates the DID; update 2 is a later anchored update that a resolver must not apply. Resolves to version 2, deactivated; version 3 does not exist.

DID: `did:btcr2:k1q5pzmjfx5q7hss5vhcydfxdlcfe2f7ly89hatcfq8vkw43zw83ch52c0ja9xc`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

### 22-x1-three-updates-resolution-options

x1 DID with 3 sidecar updates, each anchored in its own block. The sub-vectors cover versionId, versionTime (before the first update, between the first and the second, at the second), an unreachable versionId, both options, and minConf above the chain depth.

DID: `did:btcr2:x1qhfjzym7ah9q7zadrummn48wqs5ulrpufull05whrlz7lhemtank2vpvv50`

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

DID: `did:btcr2:k1q5p97uqzjlumn2d6zmv0wa8aqhrasyxxhlmdhzkfa8hf5f0vj5am6gqah3p5x`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionTime":"before:3"}` | versionId 3 |

### 24-k1-removed-beacon-signal

k1 DID; update 1 removes the #initialP2PKH beacon service (version 2); update 2 is announced at the removed #initialP2PKH address and adds a service. A resolver ignores the signal of a beacon address that the current document does not carry (Process Next Update, step 4): resolves to version 2, and version 3 does not exist.

DID: `did:btcr2:k1q5paaduz7faxp708a0r7tppmcx96naa6g3k4rsny9wnujq804mqhdrqtljr5a`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

## Negative vectors

| Scenario | Type | Delivery | Expected | Path |
|----------|------|----------|----------|------|
| n01-k1-invalid-did-checksum | k1 | no update | error `INVALID_DID` | [`k1/q5pl0pht/`](./k1/q5pl0pht/) |
| n02-x1-invalid-did-padding | x1 | genesis sidecar | error `INVALID_DID` | [`x1/qkvngz35/`](./x1/qkvngz35/) |
| n03-k1-invalid-did-network-nibble | k1 | no update | error `INVALID_DID` | [`k1/qcpqm8pm/`](./k1/qcpqm8pm/) |
| n04-x1-genesis-hash-mismatch | x1 | genesis sidecar | error `INVALID_DID` | [`x1/qhtj39nc/`](./x1/qhtj39nc/) |
| n05-x1-missing-update-data | x1 | genesis sidecar, 1 update withheld | error `MISSING_UPDATE_DATA` | [`x1/qhwufjvy/`](./x1/qhwufjvy/) |
| n10-k1-invalid-update-context-member | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5ppjlgm/`](./k1/q5ppjlgm/) |
| n11-k1-invalid-update-context-order | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5petkk0/`](./k1/q5petkk0/) |
| n12-k1-invalid-update-proof-context | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pqp5tn/`](./k1/q5pqp5tn/) |
| n13-k1-invalid-update-capability-action | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pueuxw/`](./k1/q5pueuxw/) |
| n14-k1-invalid-update-capability-encoding | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pggxe7/`](./k1/q5pggxe7/) |
| n15-k1-invalid-update-proof-purpose | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5p0w6a9/`](./k1/q5p0w6a9/) |
| n16-x1-invalid-update-unauthorized-method | x1 | genesis sidecar, 1 update sidecar | error `INVALID_DID_UPDATE` | [`x1/q4d3qyze/`](./x1/q4d3qyze/) |
| n17-k1-invalid-update-unknown-method | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pyz053/`](./k1/q5pyz053/) |
| n18-k1-invalid-update-proof-value | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pduhpu/`](./k1/q5pduhpu/) |
| n19-k1-invalid-update-source-hash | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5puvng8/`](./k1/q5puvng8/) |
| n20-k1-invalid-update-target-hash | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pqss3y/`](./k1/q5pqss3y/) |
| n21-k1-invalid-update-version-skip | k1 | 1 update sidecar | error `LATE_PUBLISHING_ERROR` | [`k1/q5pt9ln3/`](./k1/q5pt9ln3/) |
| n22-k1-invalid-update-patch-missing-path | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pkk6xt/`](./k1/q5pkk6xt/) |
| n23-k1-invalid-update-patch-changes-id | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5p4s0y9/`](./k1/q5p4s0y9/) |
| n24-k1-invalid-update-patch-invalid-document | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pe44p3/`](./k1/q5pe44p3/) |
| n25-k1-invalid-update-created-after-block | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5py5sz0/`](./k1/q5py5sz0/) |
| n26-k1-invalid-update-expires-before-mediantime | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5p9zf8s/`](./k1/q5p9zf8s/) |
| n27-k1-invalid-update-expires-before-created | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/q5pmrprx/`](./k1/q5pmrprx/) |
| n28-k1-late-publishing | k1 | 2 updates sidecar | error `LATE_PUBLISHING_ERROR` | [`k1/q5ptfnef/`](./k1/q5ptfnef/) |

### n01-k1-invalid-did-checksum

Negative: the resolve input carries a k1 identifier with a wrong Bech32m checksum.

DID: `did:btcr2:k1q5pl0phtl880rcejyml83jz77wllrdgs798hwttgszkkys0aquju20c4vt99q`

### n02-x1-invalid-did-padding

Negative: the resolve input carries an x1 identifier with non-zero Bech32m padding bits.

DID: `did:btcr2:x1qkvngz35q6m29szyc5pwk6n3543je49guq0sqmeuyyujfneeu7ve4jfv07a`

### n03-k1-invalid-did-network-nibble

Negative: the resolve input carries a k1 identifier whose network nibble is the reserved value 6.

DID: `did:btcr2:k1qcpqm8pmwacx2590xjkaj727dcc8z9gmp7jlrgc2u4duehqa8rvrpfqr6hltl`

### n04-x1-genesis-hash-mismatch

Negative: the sidecar genesis document does not hash to the genesis bytes of the x1 identifier.

DID: `did:btcr2:x1qhtj39nc6q8yampkdsa3t6z5wldrju4xmkr2tvfz2unf70m0eqk0sxmqvqp`

### n05-x1-missing-update-data

Negative: x1 DID with 1 anchored update that is in neither the sidecar nor the CAS.

DID: `did:btcr2:x1qhwufjvyunn93ua3zla7n6u5u9zec7grkq9vuvpwzqpfmecpfz36k8mjd07`

### n10-k1-invalid-update-context-member

Negative: k1 DID with 1 anchored update; the update @context has a wrong last member.

DID: `did:btcr2:k1q5ppjlgm7kc3062weg82zefm7gv35937h03vn4prhvam5a7g8eg6tcsd56a8z`

### n11-k1-invalid-update-context-order

Negative: k1 DID with 1 anchored update; two members of the update @context are swapped.

DID: `did:btcr2:k1q5petkk0fvzw0mhvt8cnkh9l39lwdlktanhw6egnfz76etmad72xz0stgpyvu`

### n12-k1-invalid-update-proof-context

Negative: k1 DID with 1 anchored update; the proof @context differs from the update @context.

DID: `did:btcr2:k1q5pqp5tnnc9a66fg72xl4kqft6pjwp7nuvk6767zv2q7nj5wxltur0c4qdlvt`

### n13-k1-invalid-update-capability-action

Negative: k1 DID with 1 anchored update; proof.capabilityAction is Read.

DID: `did:btcr2:k1q5pueuxw0yz4cuzr8le40tvynxsymxspph8rur8p9p2akm3lplef68g4ztm92`

### n14-k1-invalid-update-capability-encoding

Negative: k1 DID with 1 anchored update; proof.capability carries the DID without percent-encoding.

DID: `did:btcr2:k1q5pggxe7agdh4dhj8haklct27vktr9wffld88v92y2kvffshx5uzj2qu9qg9z`

### n15-k1-invalid-update-proof-purpose

Negative: k1 DID with 1 anchored update; proof.proofPurpose is assertionMethod.

DID: `did:btcr2:k1q5p0w6a9tllxq79y7729ct2c9yrsmpumah0jq7c2ey7syuen385wyys3wxd8c`

### n16-x1-invalid-update-unauthorized-method

Negative: x1 DID with a second verification method #key-1 listed for authentication only; 1 anchored update signed with #key-1.

DID: `did:btcr2:x1q4d3qyzeg8utml3v2ukdsa79yg4fz7pp2c64ruvzg3sckkdp07yjyl6tmtl`

### n17-k1-invalid-update-unknown-method

Negative: k1 DID with 1 anchored update; proof.verificationMethod names a method the document does not contain.

DID: `did:btcr2:k1q5pyz053tr2fw58ksh9yd03tcdg7tq5my29tg89dl7373xnznmpqf5cmwkuq9`

### n18-k1-invalid-update-proof-value

Negative: k1 DID with 1 anchored update; one character of proof.proofValue differs.

DID: `did:btcr2:k1q5pduhpu3juz2zuqf3r7wpjv5wqs0wa3q07htczmlczyy762kv9epgcvscp97`

### n19-k1-invalid-update-source-hash

Negative: k1 DID with 1 anchored update; sourceHash is not the hash of the current document.

DID: `did:btcr2:k1q5puvng8hdyr9weh67np8jgrxykrpdmp93d6zns0d7njylrtkx4evpcrg5ycx`

### n20-k1-invalid-update-target-hash

Negative: k1 DID with 1 anchored update; targetHash is not the hash of the patched document.

DID: `did:btcr2:k1q5pqss3y0vng3z8nh8j7y4z9xjj3swxj074pgtds8w8dh6cvk3zrj9cgwnxy6`

### n21-k1-invalid-update-version-skip

Negative: k1 DID with 1 anchored update; targetVersionId skips a version, so a version is missing (late publishing).

DID: `did:btcr2:k1q5pt9ln3nppesqnr359z2u5s2560cnnhmmxl0l6ft7pyl0jltnm7clc5l424m`

### n22-k1-invalid-update-patch-missing-path

Negative: k1 DID with 1 anchored update whose patch removes a path that does not exist.

DID: `did:btcr2:k1q5pkk6xt4gnzgzvq2gtm60tqdz959k7q7xs08c2ps57ccrufexslfegtuzc4c`

### n23-k1-invalid-update-patch-changes-id

Negative: k1 DID with 1 anchored update whose patch replaces the document id.

DID: `did:btcr2:k1q5p4s0y99ysey27vjq5feremlljppjjdvya3xdndynsekrgt46shezcpyjhm6`

### n24-k1-invalid-update-patch-invalid-document

Negative: k1 DID with 1 anchored update whose patched document does not conform (the verification method type is not Multikey).

DID: `did:btcr2:k1q5pe44p3ey720th3eyk3hzlpuyvkgqupeetj995qkrqnjxlu7yuxhjc78mykd`

### n25-k1-invalid-update-created-after-block

Negative: k1 DID with 1 anchored update; proof.created is after the header time of the block.

DID: `did:btcr2:k1q5py5sz0y95squtux2u7lcfqvk6r4l02pnkuyccu5crzm0x9rg3d96qecha6v`

### n26-k1-invalid-update-expires-before-mediantime

Negative: k1 DID with 1 anchored update; proof.expires is before the mediantime of the block.

DID: `did:btcr2:k1q5p9zf8suw8g820as33sz40ye68tddfc4s88j7wppaqdyh33cq3ukws60pu42`

### n27-k1-invalid-update-expires-before-created

Negative: k1 DID with 1 anchored update; proof.expires is before proof.created.

DID: `did:btcr2:k1q5pmrprxxyvfn20v273mpv0kzq27dmk7y2pl3uchftpcs3dt6ymu6ggt3ppnw`

### n28-k1-late-publishing

Negative: k1 DID with two different updates from version 1 to version 2, anchored at two beacons (late publishing).

DID: `did:btcr2:k1q5ptfnefs7994nc4yjtmy495930nxvwvy8mr0wqv6d7u70sewyt3c5sxzjx32`

