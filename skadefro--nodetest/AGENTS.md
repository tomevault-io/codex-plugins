## Coding Conventions

- Always use **double quotes** for strings.
- When generating JavaScript, **follow these conventions** strictly.

## Data Access

All data must be stored in **OpenIAP’s MongoDB database** using the official client library.

Start by declaring and connecting the client:

```js
const { Client } = require("openiap");
const client = new Client();
await client.connect();
```

## Connection Lifecycle

If you're writing code that needs to reinitialize state when reconnecting (e.g., registering a queue or setting up a watch), wrap it in an `onConnected` function and hook into the client event stream:
This **MUST** be called after connect(), do not worry, you will still get the initial `Connected` and `SignedIn`

```js
await client.connect();
client.on_client_event((event) => {
    if (event && event.event === "SignedIn") {
        onConnected().catch((error) => {
            client.error(error);
        });
    }
});

async function onConnected() {
    const queuename = client.register_queue({ queuename: queue }, (event) => {
        // Handle incoming event
    });
}
```

## API Reference

Use the following methods when interacting with the OpenIAP platform:

### Database Operations

```js
client.query({ collectionname, query, projection = "", orderby = "", skip = 0, top = 100, queryas = "", explain = false })
client.aggregate({ collectionname, aggregates = "[]", queryas = "", hint = "", explain = false })
client.count({ collectionname, query = "", queryas = "", explain = false })
client.distinct({ collectionname, field, query = "", queryas = "", explain = false })
client.insert_one({ collectionname, item, w = 1, j = false })
client.insert_many({ collectionname, items, w = 1, j = false, skipresults = false })
client.update_one({ collectionname, item, w = 1, j = false })
client.insert_or_update_one({ collectionname, item, uniqeness = "_id", w = 1, j = false })
client.delete_one({ collectionname, id })
client.delete_many({ collectionname, query = "" })
```

### Collection & Index Management

```js
client.list_collections()
client.create_collection({ collectionname })
client.drop_collection(collectionname)
client.get_indexes(collectionname)
client.create_index({ collectionname, index, options, name })
client.drop_index(collectionname, indexname)
```

### Authentication

```js
client.signin({ username, password, jwt, agent, version, longtoken = false, validateonly = false, ping = false })
client.connect()
client.disconnect()
```

### File Transfer

```js
client.upload({ filepath, filename, mimetype, metadata, collectionname })
client.download({ collectionname, id, folder, filename })
```

### Work Items
Work item queues contains a list of "units of work", something that needs to be processed. Items start in state "new", and when we pop an item, the server updates its state to "processing", therefore it's VITAL that we always update the workitem's state to either "retry" if there was an error, or "successful" if there was no error and call update_workitem to save it.
```js
client.push_workitem({ wiq, wiqid, name, payload = "{}", nextrun = 0, success_wiqid = "", failed_wiqid = "", success_wiq = "", failed_wiq = "", priority = 2, files = [] })
client.pop_workitem({ wiq, wiqid, downloadfolder = "." })
client.update_workitem({ workitem, ignoremaxretries = false, files = [] })
client.delete_workitem(id)
```

### Events & Messaging

```js
// Register a watch (change stream), and calls callback everytime an object is inserted, updated or deleted
client.watch({ collectionname, paths }, callback) // callback({id, operation, document }, event_counter)
client.unwatch(watchid)
// Register a queue and handle incoming messages, returns queuename used for receiving messages
client.register_queue({ queuename }, callback) // callback({ queuename, correlation_id, replyto, routingkey, exchangename, data })
// Register an exchange and handle incoming messages, returns queuename used for receiving messages
client.register_exchange({ exchangename, algorithm, routingkey, addqueue }, callback)  // callback({ queuename, correlation_id, replyto, routingkey, exchangename, data })
client.unregister_queue(queuename)
// Sends a message to a message queue or exchange, queuename or exchangename is mandatory, so is data
client.queue_message({ queuename, data, replyto, exchangename, correlation_id, routingkey, striptoken, expiration })
// Sends a message to a message queue or exchange, and waits for a reply, and returns it
client.rpc({ queuename, data, striptoken })
```

### Helpers

```js
// call custom commands, not added to the official API yet, these might change over time and will not be backward compatible 
client.custom_command({ command, id = "", name = "", data = "" }) // data must be a JSON string or ""

client.on_client_event(callback)
client.off_client_event(eventid)

client.uniqeid()
client.formatBytes(bytes, decimals = 2) // used to format a number to b/MB/GB etc.
client.stringify(obj) // Better error messages than JSON.stringify when input is malformed

// Create an observable gauge, that can be used to create custom graphs in grafana, like keeping track of items processed or users online etx.
// the client will automatically keep reporting the last set name, until you call disable_observable_gauge
client.set_f64_observable_gauge(name, value, description)
client.set_u64_observable_gauge(name, value, description)
client.set_i64_observable_gauge(name, value, description)
client.disable_observable_gauge(name)

client.enable_tracing("openiap=info") // Always call this early to enable logging, other options are openiap=error, openiap=debug or openiap=trace
client.disable_tracing()

client.info(...) // Use this instead of console.log
client.error(...) // Use this instead of console.error
client.verbose(...) // Use this instead of console.debug
client.trace(...)
```

## Logging

**Never use** `console.log()` or `console.debug()`.

Instead, use the OpenIAP logging functions:

- `client.info(...)`
- `client.warn(...)`
- `client.error(...)`
- `client.verbose(...)`
- `client.trace(...)`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skadefro)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/skadefro)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
