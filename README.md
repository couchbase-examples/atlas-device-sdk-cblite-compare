# Atlas Device SDK to Couchbase Lite Terminology and Technology comparison
The Atlas Device SDK can be thought of as an on-device ORM that creates an “object store” that is then synced to MongoDb Atlas.  The following document describes the features of Atlas and how they compare to Couchbase Lite and Couchbase Capella App Services.

## Frozen Architecture

Couchbase Lite doesn’t use the [frozen architecture](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/frozen-arch/) {:target="_blank" rel="noopener"} paradigms, which was created as a way to pass objects between threads safely.

For most applications given that Couchbase Lite’s database is a single file, it is recommended to run all database operations on a single thread (NOT THE UI Thread) and the Couchbase Lite SDK will handle doing the work it needs to on however threads it needs to do work for processes like replication.

**Example:  Couchbase Lite Query API**

In Couchbase Lite, the Query API is designed to integrate seamlessly with platform-specific mechanisms for dealing with loading and saving data. For instance, when using the Couchbase Lite Kotlin SDK, the Coroutines and Flow APIs in Kotlin can be leveraged out of the box without any modifications. It is important to follow Android best practices for threading, especially when performing I/O operations. For example, it is recommended to use a single Dispatcher scope for database tasks, and Dispatchers.Main or ViewModelScope for UI-related tasks. Proper threading ensures efficient and responsive database interactions without blocking the UI thread.
