# did:btcr2 Regtest Test Vectors

`did:btcr2` test vectors registered on a local regtest network. The state of that network is in `did-btcr2.polar.zip`.

## Connecting to the regtest network

1. Unzip `did-btcr2.polar.zip`.
2. Change directory into the extracted folder: `cd did-btcr2-electrs.polar`
3. Start the containers: `docker-compose up`
4. Make sure the network runs with electrs:
   - Open `http://localhost:3000/blocks` in a browser. The page shows a JSON list of blocks.
   - Or run `curl localhost:3000/blocks` in a terminal.

If you get `curl: (56) Recv failure: Connection reset by peer`, or the browser cannot open localhost:3000:

1. Make sure the containers run (step 3 above).
2. Find the bitcoind container id: `docker ps` and look for the `polarlightning` container.
3. Open a shell in that container: `docker exec -it <CONTAINER_ID> bash`
4. Mine 6 blocks:
   ```sh
   bitcoin-cli \
     -regtest \
     -rpcuser=polaruser \
     -rpcpassword=polarpass \
     generatetoaddress 6 \
     $(bitcoin-cli -regtest -rpcuser=polaruser -rpcpassword=polarpass getnewaddress)
   ```
5. Wait about 30 seconds for the sync, then do step 4 above again.

You can also drag the zip file into the [Lightning Polar](https://lightningpolar.com/) app and start the network from there. Polar may drop the electrs and ipfs parts of the compose file. If so, copy them from the `docker-compose.yml` in the zip into the Polar compose file (`~/.polar/networks`).

Configure your resolver to query the electrs API at `http://localhost:3000`. The Bitcoin Core RPC of the stack is `http://localhost:18443` (user `polaruser`, password `polarpass`).

The stack also runs a Kubo (IPFS) node. It holds every CAS object of the vectors: the genesis documents, the signed updates, and the CAS Announcement Maps that a scenario delivers out of band. The node runs offline, so it serves the pinned blocks only. Configure your resolver to read CAS objects from the gateway `http://127.0.0.1:8080` (`/ipfs/<cid>?format=raw`). The Kubo RPC API is `http://127.0.0.1:5001`.

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
- A sub-vector with `minConf` equal to the confirmation count of one signal holds only at `recordedTip`. The chain of the Polar export is at that tip. A block that you mine changes the result.

## Positive vectors

| Scenario | Type | Delivery | Expected | Path |
|----------|------|----------|----------|------|
| 01-k1-base | k1 | no update | versionId 1 | [`k1/qgp45a3y/`](./k1/qgp45a3y/) |
| 02-k1-sidecar-update | k1 | 1 update sidecar | versionId 2 | [`k1/qgph7nre/`](./k1/qgph7nre/) |
| 03-x1-base | x1 | genesis sidecar | versionId 1 | [`x1/qf5zrqc4/`](./x1/qf5zrqc4/) |
| 04-x1-sidecar-update | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/q2z78yxz/`](./x1/q2z78yxz/) |
| 05-x1-no-beacon | x1 | genesis cas | versionId 1 | [`x1/qghp0w22/`](./x1/qghp0w22/) |
| 06-x1-cas-3-updates | x1 | genesis cas, 3 updates cas | versionId 4, deactivated | [`x1/qfgeftze/`](./x1/qfgeftze/) |
| 07-k1-sidecar-deactivate | k1 | 1 update sidecar | versionId 2, deactivated | [`k1/qgpx06u2/`](./k1/qgpx06u2/) |
| 08-x1-cas-update-deactivate | x1 | genesis cas, 2 updates cas | versionId 3, deactivated | [`x1/qtg5vcwk/`](./x1/qtg5vcwk/) |
| 09a-x1-cas-update-announcement | x1 | genesis cas, 1 update cas, announcement cas | versionId 2 | [`x1/qg5kgjm0/`](./x1/qg5kgjm0/) |
| 09b-x1-cas-update-announcement-paired | x1 | genesis cas, 1 update cas, announcement cas | versionId 2 | [`x1/qfmlfxut/`](./x1/qfmlfxut/) |
| 10a-x1-sidecar-update-cas-announcement | x1 | genesis sidecar, 1 update sidecar, announcement sidecar | versionId 2 | [`x1/qgxluz9h/`](./x1/qgxluz9h/) |
| 10b-x1-sidecar-update-cas-announcement-paired | x1 | genesis sidecar, 1 update sidecar, announcement sidecar | versionId 2 | [`x1/qtxu0aj9/`](./x1/qtxu0aj9/) |
| 11a-x1-cas-update-smt-proof | x1 | genesis cas, 1 update cas, SMT proof sidecar | versionId 2 | [`x1/qfgm2swr/`](./x1/qfgm2swr/) |
| 11b-x1-cas-update-smt-proof-paired | x1 | genesis cas, 1 update cas, SMT proof sidecar | versionId 2 | [`x1/qfzppzx5/`](./x1/qfzppzx5/) |
| 12a-x1-sidecar-update-smt-proof | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar | versionId 2 | [`x1/qfwwah7z/`](./x1/qfwwah7z/) |
| 12b-x1-sidecar-update-smt-proof-paired | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar | versionId 2 | [`x1/qf9ruh87/`](./x1/qf9ruh87/) |
| 13-k1-update-p2wpkh | k1 | 1 update sidecar | versionId 2 | [`k1/qgpseq0v/`](./k1/qgpseq0v/) |
| 14-k1-update-p2tr | k1 | 1 update sidecar | versionId 2 | [`k1/qgpw4847/`](./k1/qgpw4847/) |
| 15-x1-beacon-rotation | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qfaqdrxu/`](./x1/qfaqdrxu/) |
| 16-x1-beacon-add-then-use | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qt04c7dn/`](./x1/qt04c7dn/) |
| 17-x1-vm-add-rotate-authentication | x1 | genesis sidecar, 2 updates sidecar | versionId 3 | [`x1/qg935lwg/`](./x1/qg935lwg/) |
| 18-x1-embedded-invocation-key | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/qtrhj3w0/`](./x1/qtrhj3w0/) |
| 19-x1-relative-ids | x1 | genesis sidecar, 1 update sidecar | versionId 2 | [`x1/qtk24dpv/`](./x1/qtk24dpv/) |
| 20-k1-cas-update | k1 | 1 update cas | versionId 2 | [`k1/qgphrh53/`](./k1/qgphrh53/) |
| 21-k1-deactivate-then-update | k1 | 2 updates sidecar | versionId 2, deactivated | [`k1/qgpgm6kn/`](./k1/qgpgm6kn/) |
| 22-x1-three-updates-resolution-options | x1 | genesis sidecar, 3 updates sidecar | versionId 4 | [`x1/qg4zny9h/`](./x1/qg4zny9h/) |
| 23-k1-duplicate-signal | k1 | 2 updates sidecar, 1 duplicate signal | versionId 3 | [`k1/qgp0enf0/`](./k1/qgp0enf0/) |
| 24-k1-removed-beacon-signal | k1 | 2 updates sidecar | versionId 2 | [`k1/qgpz0cp4/`](./k1/qgpz0cp4/) |
| 25a-x1-smt-update-no-nonce | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar, no nonce | versionId 2 | [`x1/qfqxmcf0/`](./x1/qfqxmcf0/) |
| 25b-x1-smt-nonce-no-update | x1 | genesis sidecar, no update, SMT proof sidecar | versionId 1 | [`x1/qtcszm9j/`](./x1/qtcszm9j/) |
| 25c-x1-smt-empty-index | x1 | genesis sidecar, no update, SMT proof sidecar, no nonce | versionId 1 | [`x1/q2tyuy6t/`](./x1/q2tyuy6t/) |
| 26-k1-signal-below-current-height | k1 | 2 updates sidecar | versionId 2 | [`k1/qgpqx326/`](./k1/qgpqx326/) |

### 01-k1-base

Base resolution: k1 (key-type) DID with default 3 singleton beacons, no updates. Mirrors danubetech example 1.

DID: `did:btcr2:k1qgp45a3ycrpc54uqmq0z9zt2qnhtvlwxnhrv0phdkkzq9teq5j35y4stunrnq`

### 02-k1-sidecar-update

k1 (key-type) DID with default 3 singleton beacons, 1 sidecar update adding a DIDCommMessaging service. Mirrors danubetech example 2.

DID: `did:btcr2:k1qgph7nrekhzerkmsktp8l7rdtpxh2mw45xp6e90sjvxszpz6au0grssegjx6z`

### 03-x1-base

Base resolution: x1 (external-type) DID with default 3 singleton beacons, no updates. Genesis document delivered via sidecar. Mirrors danubetech example 3.

DID: `did:btcr2:x1qf5zrqc4fem7l65n0lpckw7yrdjelv4699lyvu8rn92q2rkws8ge7k7qm4n`

### 04-x1-sidecar-update

x1 (external-type) DID with default 3 singleton beacons, 1 sidecar update adding a DecentralizedWebNode service. Mirrors danubetech example 4.

DID: `did:btcr2:x1q2z78yxz3gy7pu25awwxlf4vgffrzkcatt909seqsucuw9ar47rr5jdfy8k`

### 05-x1-no-beacon

Base resolution: x1 (external-type) DID with NO beacon services in the document, no updates. Genesis doc delivered via sidecar. Mirrors danubetech example 5 (CAS-delivered genesis); our vector uses sidecar delivery so the vector is self-contained.

DID: `did:btcr2:x1qghp0w22wfyfkuekq75ddh5yrgu4fm20xc40np9djzptpafskklpgl4jf7j`

### 06-x1-cas-3-updates

x1 DID with default 3 singleton beacons, 3 updates delivered via CAS (didcomm, dwn, then deactivate). Mirrors danubetech example 6.

DID: `did:btcr2:x1qfgeftzejym8u9wype970senp4l82tktag2gal5d3v8kvpnf2k08jwrnudj`

### 07-k1-sidecar-deactivate

k1 (key-type) DID with default 3 singleton beacons, 1 sidecar update deactivating the DID. Mirrors danubetech example 7.

DID: `did:btcr2:k1qgpx06u2yw3404yajkf4xv339xu3zc078lj55q3ew2p3kvr3823yn3swgvhfu`

### 08-x1-cas-update-deactivate

x1 DID with default 3 singleton beacons, 1 CAS update (add didcomm) + CAS deactivate. Mirrors danubetech example 8.

DID: `did:btcr2:x1qtg5vcwkxhusl65yh2h5wls67scs57czrrv9dzgpu2zh0525plyhjnuqr73`

### 09a-x1-cas-update-announcement

x1 DID with default 3 singleton beacons, 1 CAS update + CAS Announcement Map. Mirrors danubetech example 9a (paired with 09b in danubetech's tree).

DID: `did:btcr2:x1qg5kgjm0ms2e8uq949lytmjgpe7l7leuqmjq6szq99kgl5yj5wlxxxjsajl`

### 09b-x1-cas-update-announcement-paired

x1 DID with default 3 singleton beacons, 1 CAS update + CAS Announcement Map. Mirrors danubetech example 9b (paired with 09a in danubetech's tree).

DID: `did:btcr2:x1qfmlfxutk2u5qpa8gl5x63r2zf8a4gh4wyevu4gurcksgksgce9f5ywm75v`

### 10a-x1-sidecar-update-cas-announcement

x1 DID with default 3 singleton beacons + dedicated CASBeacon, 1 sidecar update. CAS Announcement delivered via sidecar. Mirrors danubetech example 10a.

DID: `did:btcr2:x1qgxluz9hrdy9l46a87pw4nfgp9l5658lp5k3hvgrtnc2y6j67gfaxv6ev4d`

### 10b-x1-sidecar-update-cas-announcement-paired

x1 DID with default 3 singleton beacons + dedicated CASBeacon, 1 sidecar update adding DIDCommMessaging. CAS Announcement delivered via sidecar. Mirrors danubetech example 10b (paired with 10a in danubetech's tree).

DID: `did:btcr2:x1qtxu0aj9a8y5ru2as7zulqcchlxj7ay8su57k9wwxakqj4pmcchk5ftpw24`

### 11a-x1-cas-update-smt-proof

x1 DID with default 3 singleton beacons, 1 CAS update + SMT Proof (sidecar). Mirrors danubetech example 11a (paired with 11b in danubetech's tree).

DID: `did:btcr2:x1qfgm2swrsxeal7rjs8tm4dgsqatq5qt46c5cyd68c0fg695n2zv3kjavh2c`

### 11b-x1-cas-update-smt-proof-paired

x1 DID with default 3 singleton beacons, 1 CAS update + SMT Proof (sidecar). Mirrors danubetech example 11b (paired with 11a in danubetech's tree).

DID: `did:btcr2:x1qfzppzx5pq3hs2d5t4texy3p47kgv8mlcvadvx7lshu3ug47h4fcuj6l6tv`

### 12a-x1-sidecar-update-smt-proof

x1 DID with default 3 singleton beacons + dedicated SMTBeacon, 1 sidecar update. SMT Proof delivered via sidecar. Mirrors danubetech example 12a (paired with 12b in danubetech's tree).

DID: `did:btcr2:x1qfwwah7zf75x9lw4kugpfkukzpagpgfn4pz5pxp9yurvx88yp0wfk3d8qy8`

### 12b-x1-sidecar-update-smt-proof-paired

x1 DID with default 3 singleton beacons + dedicated SMTBeacon, 1 sidecar update adding DWN. SMT Proof delivered via sidecar. Mirrors danubetech example 12b (paired with 12a in danubetech's tree).

DID: `did:btcr2:x1qf9ruh87defxl2au4ct0ru72zrhxj8mffwf2tzachtu56jmsz2mqvesqvzl`

### 13-k1-update-p2wpkh

k1 DID with the default 3 singleton beacons, 1 sidecar update anchored at the #initialP2WPKH beacon.

DID: `did:btcr2:k1qgpseq0vxhuqzu8cj4j8f5q35jg7s5epym0z07fh3pltkgxma6c26hcwp2a9j`

### 14-k1-update-p2tr

k1 DID with the default 3 singleton beacons, 1 sidecar update anchored at the #initialP2TR beacon.

DID: `did:btcr2:k1qgpw4847amlkkkyys6jynhummypj6u9w2f2ew3lw3eqej2df5fj6nkgq097j6`

### 15-x1-beacon-rotation

x1 DID; update 1 replaces the endpoint of the #initialP2PKH singleton beacon with a new address (beacon rotation); update 2 is anchored at the rotated beacon.

DID: `did:btcr2:x1qfaqdrxu007dltfgc4g6cdf344n7u35djtzn4tmvn0uqmap8g9gh5rr0vrz`

### 16-x1-beacon-add-then-use

x1 DID; update 1 adds a fourth singleton beacon #newBeacon; update 2 is anchored at #newBeacon, so the resolver needs a second discovery round.

DID: `did:btcr2:x1qt04c7dnwmvz9nd72w0w3cdtyahglxrfq62rnrfs4n59dq2gf0scqfad42g`

### 17-x1-vm-add-rotate-authentication

x1 DID; update 1 adds the verification method #key-1, replaces authentication with it, and adds it to capabilityInvocation; update 2 is signed with #key-1.

DID: `did:btcr2:x1qg935lwg9dl37eg227u8dzrqe4sng797ycrjtdu6kt9nqcq2hjzd5wdhgz6`

### 18-x1-embedded-invocation-key

x1 DID whose capabilityInvocation embeds the initial verification method as an object; verificationMethod is empty. 1 sidecar update signed with the embedded method.

DID: `did:btcr2:x1qtrhj3w0m8y4ztp5077swct0hg6jfs0dp67wpn9ekwf0tq5yfxpwjn9leez`

### 19-x1-relative-ids

x1 DID whose genesis document spells every id as a relative DID URL (#initialKey, #initialP2PKH). 1 sidecar update.

DID: `did:btcr2:x1qtk24dpv4afp9kv7s4lw044x3pytxexq8rjxwtazud0g9z0ghz43cepamkp`

### 20-k1-cas-update

k1 DID with the default 3 singleton beacons, 1 update delivered through the CAS (no sidecar).

DID: `did:btcr2:k1qgphrh53zup2a5p3ecnfk365hacre49rxej9ezepgdcv05sdhupdlgqyx9pne`

### 21-k1-deactivate-then-update

k1 DID; update 1 deactivates the DID; update 2 is a later anchored update that a resolver must not apply. Resolves to version 2, deactivated; version 3 does not exist.

DID: `did:btcr2:k1qgpgm6kn4wqgd3unxtsht9rh34pdqc74tczdlwttp3kc7w9jsekc6lgfrhjdr`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

### 22-x1-three-updates-resolution-options

x1 DID with 3 sidecar updates, each anchored in its own block. The sub-vectors cover versionId, versionTime (before the first update, between the first and the second, at the second), an unreachable versionId, both options, minConf above the chain depth, and minConf equal to the confirmation count of the second anchor (regtest only: the case holds at the recorded tip).

DID: `did:btcr2:x1qg4zny9hzqfyvxtp6wr33k9n33kwkxujrs3n9z6k3mlq7ah93wwtqp3v2et`

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
| `resolve/10/` | `{"minConf":"depth:2"}` | versionId 3 |

### 23-k1-duplicate-signal

k1 DID; update 1 (version 2) and update 2 (version 3) are anchored at #initialP2WPKH; entry 3 announces the signed update of version 2 again at the same beacon, in a later block (a duplicate signal). Resolves to version 3. The sub-vector asks for a versionTime after the block of version 3 and before the block of the duplicate: a resolver confirms the duplicate before the versionTime test, so the duplicate does not end the resolution at version 2.

DID: `did:btcr2:k1qgp0enf0aafye8cnr7nm0x0r8sweyyj08cn0werj6ncs6ygsd2s4clsc59mmt`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionTime":"before:3"}` | versionId 3 |

### 24-k1-removed-beacon-signal

k1 DID; update 1 removes the #initialP2PKH beacon service (version 2); update 2 is announced at the removed #initialP2PKH address and adds a service. A resolver ignores the signal of a beacon address that the current document does not carry (Process Next Update, step 4): resolves to version 2, and version 3 does not exist.

DID: `did:btcr2:k1qgpz0cp4jlpknkqyc7j30ht27cq272t6vww0xg5mqj9sxrpxjx5kf9s2mwvps`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

### 25a-x1-smt-update-no-nonce

x1 DID in the SMT cohort smt-25 with 1 sidecar update. The tree entry has no nonce, so the leaf value is the updateId.

DID: `did:btcr2:x1qfqxmcf0tnarx4j59s3sfzxs82qrsgnr4vgutpymyhl4s7t9qj6z59jz09t`

### 25b-x1-smt-nonce-no-update

x1 DID in the SMT cohort smt-25 with no update. The tree entry has a nonce and no updateId, so the leaf value is hash(hash(nonce)). The proof announces no update, so the DID resolves to version 1.

DID: `did:btcr2:x1qtcszm9j8az6ku8pnzp66qk86hune0za5p420e5fl9y7xdm50zq05s4jk76`

### 25c-x1-smt-empty-index

x1 DID in the SMT cohort smt-25 with no update and no nonce. The tree has no entry for the DID, so the proof is the proof of an empty index. The DID resolves to version 1.

DID: `did:btcr2:x1q2tyuy6ttnjj5pmdktrgclld3cep0p0xf92t0wat563js7lcaalgc5rjnew`

### 26-k1-signal-below-current-height

k1 DID; update 1 adds the beacon #lateBeacon and is anchored in round 2. Update 2 is anchored at #lateBeacon in round 1, one block before update 1. The signal is below current_block_height, so a resolver ignores it. The sub-vector asks for version 3 and expects NOT_FOUND.

DID: `did:btcr2:k1qgpqx3263emcj93twykjstes7mzkgxrrryu57yz4awkha53prde8utg4kff5m`

| Sub-vector | Options | Expected |
|------------|---------|----------|
| `resolve/01/` | `{"versionId":"3"}` | error `NOT_FOUND` |

## Negative vectors

| Scenario | Type | Delivery | Expected | Path |
|----------|------|----------|----------|------|
| n01-k1-invalid-did-checksum | k1 | no update | error `INVALID_DID` | [`k1/qgp0hy8c/`](./k1/qgp0hy8c/) |
| n02-x1-invalid-did-padding | x1 | genesis sidecar | error `INVALID_DID` | [`x1/qfrgktt6/`](./x1/qfrgktt6/) |
| n03-k1-invalid-did-network-nibble | k1 | no update | error `INVALID_DID` | [`k1/qcp0cg86/`](./k1/qcp0cg86/) |
| n04-x1-genesis-hash-mismatch | x1 | genesis sidecar | error `INVALID_DID` | [`x1/qgaglc0d/`](./x1/qgaglc0d/) |
| n05-x1-missing-update-data | x1 | genesis sidecar, 1 update withheld | error `MISSING_UPDATE_DATA` | [`x1/qfuuz6h4/`](./x1/qfuuz6h4/) |
| n10-k1-invalid-update-context-member | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgp040ju/`](./k1/qgp040ju/) |
| n11-k1-invalid-update-context-order | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpejq0v/`](./k1/qgpejq0v/) |
| n12-k1-invalid-update-proof-context | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpnkuln/`](./k1/qgpnkuln/) |
| n13-k1-invalid-update-capability-action | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpf5yjw/`](./k1/qgpf5yjw/) |
| n14-k1-invalid-update-capability-encoding | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpw65qy/`](./k1/qgpw65qy/) |
| n15-k1-invalid-update-proof-purpose | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgp6fp4d/`](./k1/qgp6fp4d/) |
| n16-x1-invalid-update-unauthorized-method | x1 | genesis sidecar, 1 update sidecar | error `INVALID_DID_UPDATE` | [`x1/qty0lp74/`](./x1/qty0lp74/) |
| n17-k1-invalid-update-unknown-method | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgp5wcmx/`](./k1/qgp5wcmx/) |
| n18-k1-invalid-update-proof-value | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgp2ht79/`](./k1/qgp2ht79/) |
| n19-k1-invalid-update-source-hash | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpmreat/`](./k1/qgpmreat/) |
| n20-k1-invalid-update-target-hash | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgp3e09g/`](./k1/qgp3e09g/) |
| n21-k1-invalid-update-version-skip | k1 | 1 update sidecar | error `LATE_PUBLISHING_ERROR` | [`k1/qgpxl5uu/`](./k1/qgpxl5uu/) |
| n22-k1-invalid-update-patch-missing-path | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgp5fh0e/`](./k1/qgp5fh0e/) |
| n23-k1-invalid-update-patch-changes-id | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpl0zen/`](./k1/qgpl0zen/) |
| n24-k1-invalid-update-patch-invalid-document | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpq3zd0/`](./k1/qgpq3zd0/) |
| n25-k1-invalid-update-created-after-block | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpq4wrg/`](./k1/qgpq4wrg/) |
| n26-k1-invalid-update-expires-before-mediantime | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgp33y4v/`](./k1/qgp33y4v/) |
| n27-k1-invalid-update-expires-before-created | k1 | 1 update sidecar | error `INVALID_DID_UPDATE` | [`k1/qgpp9e44/`](./k1/qgpp9e44/) |
| n28-k1-late-publishing | k1 | 2 updates sidecar | error `LATE_PUBLISHING_ERROR` | [`k1/qgpepnx0/`](./k1/qgpepnx0/) |
| n29-x1-smt-proof-hash | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar (`hash` changed) | error `INVALID_SIGNAL_DATA` | [`x1/qgncuznq/`](./x1/qgncuznq/) |
| n30-x1-smt-proof-root-id | x1 | genesis sidecar, 1 update sidecar, SMT proof sidecar (`id` changed) | error `MISSING_UPDATE_DATA` | [`x1/qf0zm452/`](./x1/qf0zm452/) |
| n31-x1-smt-proof-withheld | x1 | genesis sidecar, 1 update sidecar, no SMT proof | error `MISSING_UPDATE_DATA` | [`x1/qttq27ml/`](./x1/qttq27ml/) |

### n01-k1-invalid-did-checksum

Negative: the resolve input carries a k1 identifier with a wrong Bech32m checksum.

DID: `did:btcr2:k1qgp0hy8cwcgrj7cw8ufawj8gmmfqsaew83sh8f40wfzntspety3ep9cz42djq`

### n02-x1-invalid-did-padding

Negative: the resolve input carries an x1 identifier with non-zero Bech32m padding bits.

DID: `did:btcr2:x1qfrgktt67zycyup5hcxnhy4hayjyjszg39zkenhunsxc6mtlkc7chpda5q3`

### n03-k1-invalid-did-network-nibble

Negative: the resolve input carries a k1 identifier whose network nibble is the reserved value 6.

DID: `did:btcr2:k1qcp0cg8668q6wwdgtxh3lceakmc0l43wrjzc2pa8suhej0ywpspf3gqzjlp0l`

### n04-x1-genesis-hash-mismatch

Negative: the sidecar genesis document does not hash to the genesis bytes of the x1 identifier.

DID: `did:btcr2:x1qgaglc0dpu0mhktpgzfcna9qxemcr99yrhq72hex4kx4jhcumxj5cc4yygv`

### n05-x1-missing-update-data

Negative: x1 DID with 1 anchored update that is in neither the sidecar nor the CAS.

DID: `did:btcr2:x1qfuuz6h4mxgj04g6w8hg4uhk3caf9uzrwfvjapxcxe29tujry0hw5dhg2t8`

### n10-k1-invalid-update-context-member

Negative: k1 DID with 1 anchored update; the update @context has a wrong last member.

DID: `did:btcr2:k1qgp040juug4k0ml7306utyz3sx7zg04t22wzlpmvmey5we48gk8ytfsmr5a2s`

### n11-k1-invalid-update-context-order

Negative: k1 DID with 1 anchored update; two members of the update @context are swapped.

DID: `did:btcr2:k1qgpejq0v0svp3gccwh20v2mgsaa8rfmq5ktelyzsjuq0hqs90fj3kys5zjflx`

### n12-k1-invalid-update-proof-context

Negative: k1 DID with 1 anchored update; the proof @context differs from the update @context.

DID: `did:btcr2:k1qgpnkulnyxrsw73csr5htq3vqqhp6lr6r0fsetrmk0jw2xf6nn98eucdlch3e`

### n13-k1-invalid-update-capability-action

Negative: k1 DID with 1 anchored update; proof.capabilityAction is Read.

DID: `did:btcr2:k1qgpf5yjw78ulg88tt4mu56nv9uca0eqn4m3udr3agur3qdk0za8mmwqhk02m4`

### n14-k1-invalid-update-capability-encoding

Negative: k1 DID with 1 anchored update; proof.capability carries the DID without percent-encoding.

DID: `did:btcr2:k1qgpw65qy7gszey64424hmygs4dm99e0dzqayxqgsmpvp699erydsxaq6xucna`

### n15-k1-invalid-update-proof-purpose

Negative: k1 DID with 1 anchored update; proof.proofPurpose is assertionMethod.

DID: `did:btcr2:k1qgp6fp4d4kfhag9zh2ey2fgqjtgnzqlj7xujc4a65ds5a0zurjmagvsrye745`

### n16-x1-invalid-update-unauthorized-method

Negative: x1 DID with a second verification method #key-1 listed for authentication only; 1 anchored update signed with #key-1.

DID: `did:btcr2:x1qty0lp74nvyt75dnx32p3pr5s4ja0nt3uc8r05u6cdxwxmrc706exfupn82`

### n17-k1-invalid-update-unknown-method

Negative: k1 DID with 1 anchored update; proof.verificationMethod names a method the document does not contain.

DID: `did:btcr2:k1qgp5wcmx75cg6e68s6mv8ell4x333t2d0qknvkyls6jpz5k48u6wlac5ez3uf`

### n18-k1-invalid-update-proof-value

Negative: k1 DID with 1 anchored update; one character of proof.proofValue differs.

DID: `did:btcr2:k1qgp2ht79cm5ls3hccw38csqsmxw54t4kxm9kxakyv3melkffmz63eag9atl3m`

### n19-k1-invalid-update-source-hash

Negative: k1 DID with 1 anchored update; sourceHash is not the hash of the current document.

DID: `did:btcr2:k1qgpmreat5m9tvmr9z784v9aqlr9l0dx9lmc3v4zm5wnxq2vjaswzvxg52dwdf`

### n20-k1-invalid-update-target-hash

Negative: k1 DID with 1 anchored update; targetHash is not the hash of the patched document.

DID: `did:btcr2:k1qgp3e09g3m0y64xp3kqhzs09kfra4202vmuftq2n7nuhs3qeuysx8hslgzduh`

### n21-k1-invalid-update-version-skip

Negative: k1 DID with 1 anchored update; targetVersionId skips a version, so a version is missing (late publishing).

DID: `did:btcr2:k1qgpxl5uu5dqgef2r73syfq5sq6z97zjgp4l3vqcg2j8atk784plu80qgql87z`

### n22-k1-invalid-update-patch-missing-path

Negative: k1 DID with 1 anchored update whose patch removes a path that does not exist.

DID: `did:btcr2:k1qgp5fh0eg6edzah0fkzxlrl36ql2746rkal60mzelj7adasfvwjdlhsqw8ljd`

### n23-k1-invalid-update-patch-changes-id

Negative: k1 DID with 1 anchored update whose patch replaces the document id.

DID: `did:btcr2:k1qgpl0zenjt5zrhzm9r9wq7nmjjth7lwsxzy9mkllxwyaljx5teh4p6sgkd6hz`

### n24-k1-invalid-update-patch-invalid-document

Negative: k1 DID with 1 anchored update whose patched document does not conform (the verification method type is not Multikey).

DID: `did:btcr2:k1qgpq3zd0f7kgruwy3ra3hkwh72kyfw25prujvcvpjw43rc6ylndkptsld8e5m`

### n25-k1-invalid-update-created-after-block

Negative: k1 DID with 1 anchored update; proof.created is after the header time of the block.

DID: `did:btcr2:k1qgpq4wrg3f75ekkpedexmvxg3yeyeppgd0ct7mpcx0awjmrelewgp5qt2fut8`

### n26-k1-invalid-update-expires-before-mediantime

Negative: k1 DID with 1 anchored update; proof.expires is before the mediantime of the block.

DID: `did:btcr2:k1qgp33y4vgpe4thxha0l2qu6pa2lej98v6z65x6hj35c7a39xad3nw7g3s05kq`

### n27-k1-invalid-update-expires-before-created

Negative: k1 DID with 1 anchored update; proof.expires is before proof.created.

DID: `did:btcr2:k1qgpp9e44h4nla8n03smnlgzpz7tvhxjf9c3uv7420dxzm0zphl5l3xsuafz9v`

### n28-k1-late-publishing

Negative: k1 DID with two different updates from version 1 to version 2, anchored at two beacons (late publishing).

DID: `did:btcr2:k1qgpepnx06y54nf8cnftdcq6wkptysutl6h3vdew88nt668jm6efhemqrr5jyp`

### n29-x1-smt-proof-hash

Negative: x1 DID in the SMT cohort smt-25 with 1 sidecar update; one byte of the first entry of hashes in the sidecar proof differs.

DID: `did:btcr2:x1qgncuznqmakha4yqsrzfnz8hzvphxgygvpu8dw4zlj0pnhdexd8p6qx3gfw`

### n30-x1-smt-proof-root-id

Negative: x1 DID in the SMT cohort smt-25 with 1 sidecar update; the id of the sidecar proof is not the signal root. A resolver finds proofs by id, so the sidecar has no proof for the signal.

DID: `did:btcr2:x1qf0zm452ltcpyq38mxxfz9n5kn6zvx6rph5wnnmk8cllakt9nrpjq4p6lta`

### n31-x1-smt-proof-withheld

Negative: x1 DID in the SMT cohort smt-25 with 1 sidecar update; the sidecar holds no SMT proof.

DID: `did:btcr2:x1qttq27mlpynx896eccn46l4m8hxa9q0qs7qv35fk7gmw9nxszxm7gp8mvj0`

