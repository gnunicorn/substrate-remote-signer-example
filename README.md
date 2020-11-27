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


## Integration example

This simply shows how to integrate _any_ remote Keystore into a substrate-based chain easily. Aside from `lib`, which contains the specific protocol and the example-server the only change we did compared to the original `node-template` (actually `node` of it only) was adding the `lib` as dependency (named `tssrs`) and then replace the given `remote_keystore` with one that matches on the particular uri-name:

```diff

diff --git a/src/service.rs b/src/service.rs
index dcc35ea..9dda162 100644
--- a/src/service.rs
+++ b/src/service.rs
@@ -10,7 +10,7 @@ use sc_executor::native_executor_instance;
 pub use sc_executor::NativeExecutor;
 use sp_consensus_aura::sr25519::{AuthorityPair as AuraPair};
 use sc_finality_grandpa::SharedVoterState;
-use sc_keystore::LocalKeystore;
+use tssrs::client::RemoteKeystore;

 // Our native executor instance.
 native_executor_instance!(
@@ -83,11 +83,14 @@ pub fn new_partial(config: &Configuration) -> Result<sc_service::PartialComponen
 	})
 }

-fn remote_keystore(_url: &String) -> Result<Arc<LocalKeystore>, &'static str> {
-	// FIXME: here would the concrete keystore been build,
-	//        must return a concrete type (NOT `LocalKeystore`) that
-	//        implements `CryptoStore` and `SyncCryptoStore`
-	Err("Remote Keystore not supported.")
+// WE have implemented this here to integrate the new url-scheme
+fn remote_keystore(url: &String) -> Result<Arc<RemoteKeystore>, String> {
+	if url.starts_with("tssrs+") {
+		RemoteKeystore::open(url[6..].to_string(), None)
+			.map(Arc::new)
+	} else {
+		Err("Remote Keystore not supported.".to_owned())
+	}
 }

 /// Builds a new service for a full client.

```

That's it.