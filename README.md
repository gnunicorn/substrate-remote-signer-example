# substrate-remote-signer-example
Too Simple Substrate Remote Signer (tssrs) protocol example implementation

⚠️ This is an example to show how to integrate a remote-signer into [Substrate](). This has severe security issues and can't and should _never_ be used in any production environment. This is for educational purposes only.

## trying it out

The main crate contains the actual code of the protocol, the two examples show a fully working integration. One acts as the sever, the other implements a default `node-template` with the specific remote-keystore-matching function. You can try it out as follows:

To set up let's create a local keystore we can reuse. Start the node and let it create the keystore, then wait until a few blocks have been created and kill it:

```bash
# build the node, run it with a local keystore
cargo run -- --dev -d .local
```

Now we copy that over and feed it to the example server:
```bash
# copy the keystore out of the local project
mv .local/dev/keystore .local/remote-keystore
# now run the server with the remote keystore
cargo run --manifest-path example-server/Cargo.toml -- --keystore-path .local/remote-keystore
```

While this runs, start the node again in a second terminal, this time give it an empty local keystore and point it to the remote server

```bash
cargo run -- --dev -d .local --remote-keystore tssrs+http://localhost:33033/ --keystore-path /tmp/keystore
```


You'll see it does author blocks as before using the alice key despite the local keystore being (and remaining) empty.