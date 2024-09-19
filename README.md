# Atlas Device SDK to Couchbase Lite Terminology and Technology comparison
The Atlas Device SDK can be thought of as an on-device ORM that creates an “object store” that is then synced to MongoDb Atlas.  The following document describes the features of Atlas and how they compare to Couchbase Lite and Couchbase Capella App Services. 

[!NOTE]
Links to the Atlas SDK are linked to the Kotlin SDK specifically.  The Atlas SDK is available in multiple languages and the documentation can be found [here](https://www.mongodb.com/docs/atlas/device-sdks/).

## Frozen Architecture

Couchbase Lite doesn’t use the [frozen architecture](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/frozen-arch/) paradigms, which was created as a way to pass objects between threads safely.

For most applications given that Couchbase Lite’s database is a single file, it is recommended to run all database operations on a single thread (NOT THE UI Thread) and the Couchbase Lite SDK will handle doing the work it needs to on however threads it needs to do work for processes like replication.

**Example:  Couchbase Lite Query API**

In Couchbase Lite, the Query API is designed to integrate seamlessly with platform-specific mechanisms for dealing with loading and saving data. For instance, when using the Couchbase Lite Kotlin SDK, the Coroutines and Flow APIs in Kotlin can be leveraged out of the box without any modifications. It is important to follow Android best practices for threading, especially when performing I/O operations. For example, it is recommended to use a single Dispatcher scope for database tasks, and Dispatchers.Main or ViewModelScope for UI-related tasks. Proper threading ensures efficient and responsive database interactions without blocking the UI thread.

## Model Data/Object Store

Realm relies on the developer to define [“Realm objects”](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#realm-objects) or objects that are uniquely named that they can interact with Atlas Device and Device synchronization.  Couchbase Lite is flexible in that all data is stored in the database as a JSON document.  You can use SQL++ Queries to get information from the database OR you can query the database using a documentId which can be similarly thought of as Realms “ObjectId”.

Couchbase Lite you save information to the database in a JSON Document.  Within Couchbase Lite there is the Document API that allows you to either set fields on a document to values (KV operations), or you can serialize an object down to JSON and then create the document based on the JSON values.  You can reverse take the document from JSON and then serialize it to an object.

When you embed documents in Realm you use the [EmbeddedRealmObject](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-embedded-object-type), whereas in Couchbase Lite we just think of embedded data as an embedded document.

[Asymmetric Object Types](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-asymmetric-object-type) are unique to Realm and Atlas Device sync as a way to control insert-only documents.

All documents inserted into a Couchbase Lite database are a document.  When you retrieve a document from the Couchbase Lite Database, it’s a Document object (immutable) until you convert it to a MutableDocument which would then allow you to make changes to it.  Couchbase Lite and App Service/Sync Gateway have a different [security model](https://docs.couchbase.com/sync-gateway/current/access-control-model.html) that handles what documents should be synced via documents being assigned to a [“channel”](https://docs.couchbase.com/sync-gateway/current/channels.html) and the [replication configuration](https://docs.couchbase.com/couchbase-lite/current/android/replication.html) to decide which documents in what scopes and collections should be synced vs stay internal to the local database on the device.


### Realm Object Types

| Name | Documentation| 
|---|---|
| RealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#realm-objects)                     |
| EmbeddedRealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-embedded-object-type)    |
| AsymmetricRealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-asymmetric-object-type) |

### Collection Types

In Realm there are three major collection types:  RealmList, RealmSet, and RealmDictionary.  Given Realm’s ORM-like nature, this logically makes sense.  There is no direct comparison between Realm’s collection types and Couchbase Lite.  Couchbase Lite does support querying the database and when returning results they come back in a ResultSet.  A result set can be iterated through and return the Result object, which can then be type-casted to a custom Object type defined in the application, or can be converted to JSON via the Result.toJSON function call. Once it’s a JSON string, any serialization library for the given platform can be used.

### Summary

In summary, the developer has to do a lot of planning of what type of object they should be inheriting from in order to use Realm and the Atlas Device SDK.  In Couchbase Lite, all information is stored in the database as a JSON document, and you can pick the pattern on how you want to write and read documents from the database.  

## Collections
In Realm, objects are stored in collections based on the object type (all objects in a collection are of the same type) unless the developer is using the [Realm Unstructured](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-unstructured-data) data API (RealmAny).  Storing data in this way comes at a performance cost in Realm.

Couchbase Lite supports Scopes and Collections for storing JSON documents.  A scope is a container of 1 or many Collections, up to 1,000 collections per scope.  Collections are nothing more than a group of JSON documents, whose contents can be different (schema doesn’t have to be the same).  Storing documents with different schemas in the same Collection comes at no performance loss in Couchbase Lite and SQL++ has keywords to help you query for documents that are missing a property(field).

### Supported Data Types/Data Modeling
In Realm each platform as a given set of supported Data Types:
- [Flutter](https://www.mongodb.com/docs/atlas/device-sdks/sdk/flutter/realm-database/model-data/data-types/)
- [Java](https://www.mongodb.com/docs/atlas/device-sdks/sdk/java/model-data/data-types/field-types/)
- [Kotlin](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/)
- [.NET](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/model-data/data-types/field-types/)
- [React Native](https://www.mongodb.com/docs/atlas/device-sdks/sdk/react-native/model-data/data-types/property-types/)
- [Swift](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/supported-types/#property-cheat-sheet)

In Couchbase Lite, Document Data Types supported are scalar types regardless of the language or SDK:

- Boolean
- Date
- Double
- Float
- Int
- Long
- String

On top of scalar types, Couchbase Lite also supports

- Dictionary - represents a key-value pair collection
- Array - represents an ordered collection of objects
- Blob - represents an arbitrary piece of binary data

Couchbase Lite documentation per SDK can be found here:


- [Android - Java](https://docs.couchbase.com/couchbase-lite/current/android/document.html#data-types)
- [Android - Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/document.html#data-types)
- [C](https://docs.couchbase.com/couchbase-lite/current/c/document.html#data-types/)
- [Java](https://docs.couchbase.com/couchbase-lite/current/java/document.html#data-types)
- [.NET](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#data-typess)
- [Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/document.html#data-types)
- [React Native](https://cbl-reactnative.dev/documents#document-structure)
- [Swift](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#data-types)

### ObjectId vs DocumentId

Realm objects can use the [ObjectId](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#objectid) or [UUID](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#realmuuid) for unique identifiers for Realm objects.  The ObjectId is a 12-byte unique value that is [nullable](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#objectid), indexable, and used as a primary key.  In Couchbase Lite a documentId can be any string value as long as it’s not null.  A document created without an ID will always have a randomly generated UUID used for the documentId.