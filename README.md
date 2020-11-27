# substrate-remote-signer-example
Too Simple Substrate Remote Signer (tssrs) protocol example implementation

⚠️ This is an example to show how to integrate a remote-signer into [Substrate](). This has severe security issues and can't and should _never_ be used in any production environment. This is for educational purposes only.

## trying it out

The main crate contains the actual code of the protocol, the two examples show a fully working integration. One acts as the sever, the other implements a default `node-template` with the specific remote-keystore-matching function. You can try it out as follows:

To set up let's create a local keystore we can reuse. Start the node and let it create the keystore, then wait until a few blocks have been created and kill it:

```bash
# create a local key store and add alice's key to it
cargo run -- key insert -d /tmp --keystore-path .local/remote-keystore --scheme Sr25519 --key-type aura --suri "bottom drive obey lake curtain smoke basket hold race lonely fit walk//Alice
```

Now we copy that over and feed it to the example server:
```bash
# now run the server exposing that local keystore
cargo run --manifest-path example-server/Cargo.toml -- --keystore-path .local/remote-keystore
```

Which should says something like:
```
Serving Remote Signer at http://127.0.0.1:33033
```

While this runs, start the node again in a second terminal, this time give it an empty local keystore and point it to the remote server

```bash
cargo run -- --dev -d .local/node --keystore-uri tssrs+http://localhost:33033/
```

You'll see it does author connect to the remote server and starts producing blocks:

```
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/node-tssrs-example --validator --dev -d .local/node --keystore-uri 'tssrs+http://localhost:33033/'`
2020-11-27 14:23:17  Running in --dev mode, RPC CORS has been disabled.
2020-11-27 14:23:17  Substrate Node
2020-11-27 14:23:17  ✌️  version 2.0.0-5f51e4c-x86_64-linux-gnu
2020-11-27 14:23:17  ❤️  by Anonymous, 2017-2020
2020-11-27 14:23:17  📋 Chain specification: Development
2020-11-27 14:23:17  🏷 Node name: fumbling-underwear-1857
2020-11-27 14:23:17  👤 Role: AUTHORITY
2020-11-27 14:23:17  💾 Database: RocksDb at .local/node/chains/dev/db
2020-11-27 14:23:17  ⛓  Native runtime: node-template-1 (node-template-1.tx1.au1)
2020-11-27 14:23:17  Using default protocol ID "sup" because none is configured in the chain specs
2020-11-27 14:23:17  🏷 Local node identity is: 12D3KooWRzUDhkKp2PZvfNMuEN85D7zFyj5Zy4Z3nK2q9vvs2wrc
2020-11-27 14:23:18  📦 Highest known block at #0
2020-11-27 14:23:18  〽️ Prometheus server started at 127.0.0.1:9615
2020-11-27 14:23:18  Listening for new connections on 127.0.0.1:9944.
2020-11-27 14:23:18  Connecting to http://localhost:33033/
```

You will also see the server side report that it hands out connections.