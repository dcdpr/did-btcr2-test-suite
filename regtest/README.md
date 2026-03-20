# did:btcr2 Regtest Test Vectors

This folder contains a set of test vectors for **did:btcr2** identifiers registered on a local regtest network. The state of that network is contained in the `did-btcr2.polar.zip` folder.

## Connecting to the regtest network

1. Unzip the contents of the `did-btcr2.polar.zip` folder.
2. Change directory into the extracted folder: `cd did-btcr2-electrs.polar`
3. Start the containers: `docker-compose up`
4. Verify the network is running with electrs.
  a. Go to `http://localhost:3000/blocks`, you should see a list of json objects representing bitcoin blocks
  b. Run `curl localhost:3000/blocks` in a terminal

If you get `curl: (56) Recv failure: Connection reset by peer` or cannot visit localhost:3000 in a browser, try the following debugging steps.

1. Ensure the containers are running (step 3 above)
2. Open a new terminal window
3. Determine the bitcoind container ID: `docker ps | awk -F '  ' '{ print $1, $2 }' | grep 'polar'` (or just `docker ps` and look for the CONTAINER ID of the polarlightning container)
4. Drop a shell into that bitcoind container: `docker exec -it <CONTAINER_ID> bash`
5. Once inside the bitcoind container, mine 6 blocks:
  ```sh
  bitcoin-cli \
    -regtest \
    -rpcuser=polaruser \
    -rpcpassword=polarpass \
    generatetoaddress 6 \
    $(bitcoin-cli -regtest -rpcuser=polaruser -rpcpassword=polarpass getnewaddress)
  ```
4. Wait 30s or so for it to finish syncing and retry the verification step (step 4) from above.

Alternatively, the zip file can be dragged into the [lighting polar](https://lightningpolar.com/) app, if it is installed. Then the network can be started through the apps interface. However, doing this may drop the electrs part of the docker file. If this happens, view the docker-compose file in the `did-btcr2.polar.zip` folder and copy across into the polar docker compose. Polar networks can be found under `~/.polar/networks`.

You should configure your resolver to query the electrs API at `http://localhost:3000`.

## Data

* [/regtest/k1/qgpakaw4](/regtest/k1/qgpakaw4) - no changes, simplest case
* [/regtest/k1/qgppexmy](/regtest/k1/qgppexmy) - replaces the #initialP2PKH SingletonBeacon serviceEndpoint
* [/regtest/k1/qgpy0hmm](/regtest/k1/qgpy0hmm) - adds #additionalP2PKH SingletonBeacon service

* [/regtest/x1/q2fz9mz6](/regtest/x1/q2fz9mz6) - no changes, simplest case
* [/regtest/x1/q26jeds9](/regtest/x1/q26jeds9) - adds #service-1 SingletonBeacon service
* [/regtest/x1/qfl7se8f](/regtest/x1/qfl7se8f) - adds #key-1 verificationMethod and replaces authentication with #key-1 