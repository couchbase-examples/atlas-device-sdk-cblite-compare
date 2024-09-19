# Atlas Device SDK to Couchbase Lite Comparison Guide
The Atlas Device SDK can be thought of as an on-device ORM that creates an “object store” that is then synced to MongoDb Atlas.  The following document describes the features of Atlas and how they compare to Couchbase Lite and Couchbase Capella App Services. 

>**NOTE** 
>
>Links to the Atlas SDK are linked to the Kotlin SDK specifically.  The Atlas SDK is available in multiple languages and the documentation can be found [here](https://www.mongodb.>com/docs/atlas/device-sdks/).  All code examples are provided in Kotlin for Android or SQL++.
>
 
## Frozen Architecture

Couchbase Lite doesn’t use the [frozen architecture](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/frozen-arch/) paradigms, which the Atlas Device SDK uses as a way to pass objects between threads safely.

For most applications given that Couchbase Lite’s database is a single file, it is recommended to run all database operations on a single thread (NOT THE Main Thread) and the Couchbase Lite SDK will handle doing the work it needs to on however threads it needs to do work for processes like replication.

**Example:  Couchbase Lite Query API with the Kotlin SDK**

In Couchbase Lite, the Query API is designed to integrate seamlessly with platform-specific mechanisms for dealing with loading and saving data. For instance, when using the Couchbase Lite Kotlin SDK, the Coroutines and Flow APIs in Kotlin can be leveraged out of the box without any modifications. 

It is important to follow Android best practices for threading, especially when performing I/O operations. For example, it is recommended to use a single Dispatcher scope for database tasks, and Dispatchers.Main or ViewModelScope for UI-related tasks. Proper threading ensures efficient and responsive database interactions without blocking the UI thread.

## Model Data/Object Store

Realm relies on the developer to define "[Realm objects](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#realm-objects)" or objects that are uniquely named that they can interact with Atlas Device and Device synchronization.  

### Realm Object Types

| Name | Documentation| 
|---|---|
| RealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#realm-objects)                     |
| EmbeddedRealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-embedded-object-type)    |
| AsymmetricRealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-asymmetric-object-type) |

Couchbase Lite is flexible in that all data is stored in the database as a JSON document using the Document API.  You can use SQL++ Queries to get information from the database OR you can query the database using a documentId which can be similarly thought of as Realms “ObjectId”.

When you embed documents in Realm you use the [EmbeddedRealmObject](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-embedded-object-type), whereas in Couchbase Lite you can  think of embedded data as an embedded JSON document or dictionary.

[Asymmetric Object Types](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-asymmetric-object-type) are unique to Realm and Atlas Device sync as a way to control insert-only documents.

In contrast to Asymmetric Object Types, all documents inserted into a Couchbase Lite database are just a JSON document.  When you retrieve a document from the Couchbase Lite Database using the Collection API and provide a documentId, it’s a Document object (immutable) until you convert it to a MutableDocument which would then allow you to make changes to it.  

Couchbase Lite and App Service/Sync Gateway have a different [security model](https://docs.couchbase.com/sync-gateway/current/access-control-model.html) .  Document that should be synced are assigned to a [“channel”](https://docs.couchbase.com/sync-gateway/current/channels.html) and the [replication configuration](https://docs.couchbase.com/couchbase-lite/current/android/replication.html) to decide which documents in what scopes and collections should be synced vs stay internal to the local database on the device.

### Collection Types

Realm supports three primary collection types: RealmList, RealmSet, and RealmDictionary. Each serves specific use cases for managing collections of objects in a database. Couchbase Lite, in contrast, does not have direct analogs to these collection types but instead supports flexible data structuring through JSON. This allows for storing arrays and dictionaries (akin to RealmList and RealmDictionary, respectively) directly within JSON documents.

When querying data, Couchbase Lite retrieves results in a ResultSet, which can be iterated to extract individual Result objects.  Results are, essentially, JSON documents and can be treated as Dictionaries, wrapped in application objects, or serialized to JSON. 

These results can then be directly cast to custom object types defined within the application, facilitating seamless integration with the app’s data architecture. Additionally, results can be converted into JSON strings using the Result.toJSON() function. This JSON data can subsequently be processed using any serialization library compatible with the platform in use, providing extensive flexibility for data manipulation and integration.

Couchbase Lite ResultSet Documentation:
- [ResultSet API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/query-resultsets.html)
- [ResultSet API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/query-resultsets.html)
- [ResultSet API - C](https://docs.couchbase.com/couchbase-lite/current/c/query-resultsets.html)
- [ResultSet API - Java](https://docs.couchbase.com/couchbase-lite/current/java/query-resultsets.html)
- [ResultSet API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/query-resultsets.html)
- [ResultSet API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/query-resultsets.html)
- [ResultSet API - React Native](https://cbl-reactnative.dev/Queries/query-result-set)
- [ResultSet API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/query-resultsets.html)


### Summary

In using Realm and the Atlas Device SDK, developers must carefully plan their data model, particularly regarding the types of objects they inherit from, to effectively leverage the database’s capabilities. This upfront planning is crucial due to the structured nature of the data and the object-oriented approach of Realm.

Conversely, Couchbase Lite offers a more flexible data handling approach. All information is stored as JSON documents in the database. This format allows developers to freely choose how they structure, write, and read their data. The flexibility of JSON documents enables a variety of patterns and structures to be implemented, adapting easily to the evolving needs of the application and reducing the need for extensive initial planning.

## Collections
In Realm, objects are stored in collections based on the object type (all objects in a collection are of the same type) unless the developer is using the [Realm Unstructured](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-unstructured-data) data API (RealmAny).  Storing data in this way comes at a performance cost in Realm.

Couchbase Lite supports Scopes and Collections for storing JSON documents.  A scope is a namespace that may contain up to 1,000 collections.  Collections are a group of JSON documents, whose contents can be different (schema doesn’t have to be the same).  Storing documents with different schemas in the same Collection comes at no performance loss in Couchbase Lite and SQL++ has keywords to help you query for documents that are missing a property(field).

Couchbase Lite Scopes and Collections Documentation:
- [Scopes - Collections API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/scopes-collections-manage.html)
- [Scopes - Collections API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/scopes-collections-manage.html)
- [Scopes - Collections API - C](https://docs.couchbase.com/couchbase-lite/current/c/scopes-collections-manage.html)
- [Scopes - Collections API - Java](https://docs.couchbase.com/couchbase-lite/current/java/scopes-collections-manage.html)
- [Scopes - Collections API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/scopes-collections-manage.html)
- [Scopes - Collections API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/scopes-collections-manage.html)
- [Scopes - Collections API - React Native](https://cbl-reactnative.dev/scopes-collections)
- [Scopes - Collections API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/scopes-collections-manage.html)

### Supported Data Types/Data Modeling
In Realm each platform the SDK supports as a given set of supported Data Types:
- [Flutter](https://www.mongodb.com/docs/atlas/device-sdks/sdk/flutter/realm-database/model-data/data-types/)
- [Java](https://www.mongodb.com/docs/atlas/device-sdks/sdk/java/model-data/data-types/field-types/)
- [Kotlin](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/)
- [.NET](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/model-data/data-types/field-types/)
- [React Native](https://www.mongodb.com/docs/atlas/device-sdks/sdk/react-native/model-data/data-types/property-types/)
- [Swift](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/supported-types/#property-cheat-sheet)

In Couchbase Lite, Document API Data Types supported are scalar types regardless of the platform, language, or SDK:

- Boolean
- Date
- Double
- Float
- Int
- Long
- String

On top of scalar types, Couchbase Lite Documents also supports

- Dictionary - represents a key-value pair collection
- Array - represents an ordered collection of objects
- Blob - represents an arbitrary piece of binary data

Couchbase Lite Document - Data Types Documentation:
- [Data Types - Android - Java](https://docs.couchbase.com/couchbase-lite/current/android/document.html#data-types)
- [Data Types - Android - Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/document.html#data-types)
- [Data Types - C](https://docs.couchbase.com/couchbase-lite/current/c/document.html#data-types/)
- [Data Types - Java](https://docs.couchbase.com/couchbase-lite/current/java/document.html#data-types)
- [Data Types - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#data-typess)
- [Data Types - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/document.html#data-types)
- [Data Types - React Native](https://cbl-reactnative.dev/documents#document-structure)
- [Data Types - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#data-types)

### ObjectId vs DocumentId

Realm objects can use the [ObjectId](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#objectid) or [UUID](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#realmuuid) for unique identifiers for Realm objects.  The ObjectId is a 12-byte unique value that is [nullable](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#objectid), indexable, and used as a primary key.

In Couchbase Lite a documentId can be any string value as long as it’s not null.  A document created without an ID will always have a randomly generated UUID used for the documentId.

### Geospatial Types
In the Atlas SDK, Geospatial data, or "geodata", specifies points and geometric objects on the Earth's surface.

Couchbase Lite has no direct feature or support for Geospatial data types.  A developer would need to write custom code to handle any geospatial calculations or look into 3rd party packages. 

### Property Annotations
In Realm, property annotations play a critical role in defining the schema of objects. These annotations specify how objects should be stored in the database, including details on which fields are indexed, thereby enhancing query performance and ensuring data integrity.

On the other hand, Couchbase Lite operates differently due to its use of JSON documents for storage. Instead of using property annotations to define schemas and index fields directly within the objects, Couchbase Lite manages indexing through a separate Index API. This approach allows developers to create and manage indexes independently of the document storage API, providing flexibility in how data is indexed and retrieved without altering the document structure itself.

Couchbase Lite Index API Documentation:
- [Indexing API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/indexing.html)
- [Indexing API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/indexing.html)
- [Indexing API - C](https://docs.couchbase.com/couchbase-lite/current/c/indexing.html)
- [Indexing API - Java](https://docs.couchbase.com/couchbase-lite/current/java/indexing.html)
- [Indexing API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/indexing.html)
- [Indexing API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/indexing.html)
- [Indexing API - React Native](https://cbl-reactnative.dev/indexes)
- [Indexing API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/indexing.html)


### Backlinks and Relationships
In Realm, a [backlink](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#backlinks) defines an inverse, to-many relationship between objects, where backlinks cannot be null. Unlike MongoDB, which explicitly avoids using bridge tables or explicit joins for defining relationships, Realm utilizes its proprietary mechanisms.

Couchbase Lite, on the other hand, offers two approaches for managing document relationships. Firstly, it allows for embedding documents directly within other documents, akin to the feature provided by the Atlas Device SDK via embedded dictionaries in a document. This method, while straightforward, may lead to the creation of large document sizes, which can be inefficient for certain operations. Alternatively, Couchbase Lite supports the use of document IDs to establish relationships between documents. This method involves storing a document ID within another document, effectively using it as a foreign key to facilitate joins.

To manage relationships between smaller documents or to ensure efficient querying, developers might prefer using document IDs. These IDs serve as references that link documents together, akin to foreign keys in relational databases. To resolve these relationships, developers can employ SQL++ JOIN operations, supported by Couchbase Lite’s query language. By indexing the relevant document ID fields, performance impacts during such queries are minimized, ensuring efficient data retrieval across linked documents.

Couchbase Lite SQL++ JOIN Documentation: 
- [SQL++ JOIN - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/query-n1ql-mobile.html#lbl-join)
- [SQL++ JOIN - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/query-n1ql-mobile.html#lbl-join)
- [SQL++ JOIN - C](https://docs.couchbase.com/couchbase-lite/current/c/query-n1ql-mobile.html#lbl-join)
- [SQL++ JOIN - Java](https://docs.couchbase.com/couchbase-lite/current/java/query-n1ql-mobile.html#lbl-join)
- [SQL++ JOIN - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/query-n1ql-mobile.html#lbl-join)
- [SQL++ JOIN - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/query-n1ql-mobile.html#lbl-join)
- [SQL++ JOIN - React Native](https://cbl-reactnative.dev/Queries/sqlplusplus#join-clause)
- [SQL++ JOIN - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/query-n1ql-mobile.html#lbl-join)


### Changes to Object Model (schema synchronization)
Realms [documentation](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/change-an-object-model/) has a detailed explanation of when and when you don’t have to worry about schema changes that is out of the scope of this document.  

Couchbase Lite is a Documents based database, in which documents are stored in JSON, so you never have to be concerned about schema changes.

### Summary
In both Realm and the Atlas Device SDK, considerable effort is dedicated to comprehensively understanding the data schema and meticulously defining the synchronization model for devices. This process is crucial for ensuring that the data structures are correctly aligned with the operational requirements and synchronization protocols.

Conversely, Couchbase Lite is a document-oriented database that leverages the inherent flexibility of JSON documents. This flexibility significantly reduces the time required for schema definition, allowing developers to dynamically accommodate various data types. Consequently, any constraints or data type regulations can be efficiently managed within the application’s business logic layer, rather than at the database level. This approach not only simplifies initial setup but also enhances adaptability to evolving data requirements.

## Configuring and Opening Realm vs Couchbase Lite Database

### Realm File
Realm stores a binary-encoded version of every object and type in a single .realm file.  When you open a .realm file it will create one if it doesn’t already exist.  The file is located in a path the developer specifies.

### Couchbase Lite Database

Couchbase Lite stores all JSON documents in a single database that is a filename with a cblite2 extension.   When you open a database a new one will be created if it doesn’t already exist on the disk.  The developer specifies the path to the database, or a default path is used by the SDK when no path is provided.

### Opening a Realm
To open a realm, you must provide a [RealmConfiguration](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/open-and-close-a-realm/#open-a-realm) object that defines the details of the realm.  This configuration is passed to the *Realm.open* function.  

```kotlin
// Create a realm configuration with configuration builder
// and pass all optional arguments
val config = RealmConfiguration.Builder(
    schema = setOf(Frog::class, Person::class)
)
    .name("myRealmName.realm")
    .directory("my-directory-path")
    .encryptionKey(myEncryptionKey)
    .build()
// Open the realm with the configuration object
val realm = Realm.open(config)
Log.v("Successfully opened realm: ${realm.configuration.name}")
// ... use the open realm ...
```

### Opening a Couchbase Lite Database
To open a Couchbase Lite database or create a new database, you can provide a DatabaseConfiguration object that defines the directory and an encryption key used to secure the database.  This configuration is passed on to the Database API to open the database.
```kotlin
val db = Database(
    "my-db",
    DatabaseConfigurationFactory.newConfig(
        encryptionKey = EncryptionKey("PASSWORD"),
        directory = "my-directory-path"
    )
)
Log.v("Successfully opened database: ${db.name}")
```

### In-Memory Realm
In Realm, you can open a realm that is stored entirely [in memory](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/open-and-close-a-realm/#open-an-in-memory-realm).  When this option is used, no realm file is created.   

Couchbase Lite SDK has no equivalent feature to this.  If a developer wanted data in memory only (no information on disk), the best route would be to call the App Services/Sync Gateway REST API to retrieve documents into memory.

### Custom Realm Configuration
In Realm, a custom configuration can be based to trigger additional functionality:
- Compacting a realm to reduce its file size
- Specifying a schema version or migration block for making schema changes
- Flagging a realm to be deleted if a migration is required

As stated before, Couchbase Lite doesn’t need schema management or migration plans, so there is no equivalent to this.  Couchbase Lite databases have a Database Maintenance API which can be used to run tasks like compacting the database or to rebuild indexes.

Couchbase Lite Database Maintenance API Documentation:
- [Database Maintenance API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/database.html#lbl-db-util)
- [Database Maintenance API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/database.html#lbl-db-util)
- [Database Maintenance API - C](https://docs.couchbase.com/couchbase-lite/current/c/database.html#lbl-db-util)
- [Database Maintenance API - Java](https://docs.couchbase.com/couchbase-lite/current/java/database.html#lbl-db-util)
- [Database Maintenance API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/database.html#lbl-db-util)
- [Database Maintenance API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/database.html#lbl-db-util)
- [Database Maintenance API - React Native](https://cbl-reactnative.dev/databases#database-maintenance)
- [Database Maintenance API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/database.html#lbl-db-util)

### Add Initial Data to Realm
Realm provides an [API](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/open-and-close-a-realm/#add-initial-data-to-realm) that facilitates the insertion of initial data into a Realm database at the time of its first opening.

While there is no direct equivalent in Couchbase Lite, a similar outcome can be achieved through the use of a pre-built database. This approach enables the application to be loaded with data upfront, bypassing the need to sync data from App Services during the initial application launch. This could significantly reduce the wait time for users upon their first installation and startup of the application. 

>**NOTE**
>
>It is trivial to provide initial data, over the network, in the first sync of the Database collection(s) when the app launches. The only reason to provide an initial pre-built database is to save on network traffic, once.
>

Detailed guidance on implementing pre-built databases is available in Couchbase Lite’s documentation for each supported platform:
- [Pre-build Database - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/prebuilt-database.html)
- [Pre-build Database - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/prebuilt-database.html)
- [Pre-build Database - C](https://docs.couchbase.com/couchbase-lite/current/c/prebuilt-database.html)
- [Pre-build Database - Java](https://docs.couchbase.com/couchbase-lite/current/java/prebuilt-database.html)
- [Pre-build Database - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/prebuilt-database.html)
- [Pre-build Database - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/prebuilt-database.html)
- [Pre-build Database - React Native](https://cbl-reactnative.dev/database-prebuilt)
- [Pre-build Database - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/prebuilt-database.html)

### Close a Realm
The Realm SDK provides a method to close a realm with the code [realm.close](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/open-and-close-a-realm/#close-a-realm).  Couchbase Lite’s SDK also provides a close function which will also validate that the replicator and change listeners are closed before closing the connection via the Couchbase Lite Database *close* function.

### Copying Data into a New Realm / Couchbase Lite Database Copy
The [Realm.writeCopyTo](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/open-and-close-a-realm/#copy-data-into-a-new-realm) method allows developers to copy data from an existing realm to a new realm.  There are some rules to this which are well documented.  

In Couchbase Lite there is a *Database.copy* function that can copy a database file to a new database file.  

> **NOTE** 
> You should NEVER copy a Couchbase Lite database outside of using the Database.copy API as it will invalidate other apps checkpoints.  
> 
 
Couchbase Lite Database - Copy Documentation:
- [Database Copy - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/prebuilt-database.html#deploy-db)
- [Database Copy - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/prebuilt-database.html#deploy-db)
- [Database Copy - C](https://docs.couchbase.com/couchbase-lite/current/c/prebuilt-database.html#deploy-db)
- [Database Copy - Java](https://docs.couchbase.com/couchbase-lite/current/java/prebuilt-database.html#deploy-db)
- [Database Copy - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/prebuilt-database.html#deploy-db)
- [Database Copy - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/prebuilt-database.html#deploy-db)
- [Database Copy - React Native](https://cbl-reactnative.dev/database-prebuilt#using-pre-built-database-on-app-launch)
- [Database Copy - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/prebuilt-database.html#deploy-db)

### Summary

Both the Realm API and the Couchbase Lite Database API exhibit comparable functionalities when managing databases, particularly in terms of opening, closing, and copying database instances.

Couchbase Lite Database Documentation: 
- [Database API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/database.html)
- [Database API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/database.html)
- [Database API - C](https://docs.couchbase.com/couchbase-lite/current/c/database.html)
- [Database API - Java](https://docs.couchbase.com/couchbase-lite/current/java/database.html)
- [Database API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/database.html)
- [Database API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/database.html)
- [Database API - React Native](https://cbl-reactnative.dev/databases)
- [Database API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/database.html)

## Create & Write Data in Realm vs Couchbase Lite

### Writing Data
In Realm, you write data in a transaction which is a block of code that can do multiple transactions.  Either all the transactions succeed or they fail.   An example of this is below:
```kotlin
// Open a write transaction
realm.write {
    // Instantiate a new unmanaged Frog object
    val unmanagedFrog = Frog().apply {
        name = "Kermit"
        age = 42
        owner = "Jim Henson"
    }
    assertFalse(unmanagedFrog.isManaged())
    // Copy the object to realm to return a managed instance
    val managedFrog = copyToRealm(unmanagedFrog)
    assertTrue(managedFrog.isManaged())
    // Work with the managed object ...
}
```
In this code an object is created, then copied to the Realm. 

For individual objects, Couchbase Lite doesn’t take this approach.  Instead, Couchbase Lite uses its MutableDocument API to allow developers to create a document and then save it into a Collection.  Within Multiple Documents, you have a few different ways you can create one.  The standard approach is to write some code that takes your data and puts it into the Mutable Document using Key Value pairs:
```kotlin
val mutableDocument = MutableDocument()
mutableDocument.setString("name", "Kermit")
mutableDocument.setInt("age", 42)
mutableDocument.setString("owner", "Jim Henson")
collection.save(mutableDocument);
```

Another way is to use serialization libraries to serialize down the object to JSON and then create the MutableDocument based on the JSON values.

```kotlin
val frog = Frog("Kermit", 42, "Jim Henson")
val json = Json.encodeToString(frog)
val mutableDocument = MutableDocument("doc-1", json)
collection.save(mutableDocument);
```
> **NOTE**
> You will always take a performance and memory hit depending on device constraints when serializing and deserializing objects to and from JSON strings.  If performance is a concern, it's best to use the Key Value pair approach when creating documents in Couchbase Lite.
> 

Couchbase Lite does provide a Batch API to write multiple documents at once.  At the local level this operation is still transactional: no other Database instances, including ones managed by the replicator can make changes during the execution of the block, and other instances will not see partial changes. 

An example of a inBatch transaction in Couchbase Lite:

```kotlin
database.inBatch(UnitOfWork {
    for (i in 0..9) {
        val doc = MutableDocument()
        doc.let {
            it.setValue("type", "user")
            it.setValue("name", "user $i")
            it.setBoolean("admin", false)
        }
        log("saved user document: ${doc.getString("name")}")
    }
})
```

Couchbase Lite Document Batch API Documentation:
- [Batch Operations - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/document.html#batch-operations)
- [Batch Operations - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/document.html#batch-operations)
- [Batch Operations - C](https://docs.couchbase.com/couchbase-lite/current/c/document.html#batch-operations)
- [Batch Operations - Java](https://docs.couchbase.com/couchbase-lite/current/java/document.html#batch-operations)
- [Batch Operations - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#batch-operations)
- [Batch Operations - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/document.html#batch-operations)
- [Batch Operations - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#batch-operations)
- 
### Create an Embedded Object
In Realm the user must take care when creating embedded objects and provides an API to achieve this:

```kotlin
realm.write {
    // Instantiate a parent object with one embedded address
    val contact = Contact().apply {
        name = "Kermit"
        address = EmbeddedAddress().apply {
            propertyOwner = Contact().apply { name = "Mr. Frog" }
            street = "123 Pond St"
            country = EmbeddedCountry().apply { name = "United States" }
        }
    }
    // Copy all objects to the realm to return managed instances
    copyToRealm(contact)
}
```

In Couchbase Lite, embedded objects are just embedded dictionaries which you can use KV or the JSON serialization method.  An example of KV:
```kotlin
val mdContact = MutableDictionary()
mdContact.setString("name", "Mr. Frog")

val mdCountry = MutableDictionary()
mdCountry.setString("name", "United States")

val mdAddress = MutableDictionary()
mdAddress.setDictionary("address", mdAddress)
mdAddress.setString("street", "123 Pond St")
mdAddress.setDictionary("country", mdCountry)

val mutableDocument = MutableDocument()
mutableDocument.setString("name", "Kermit")
mutableDocument.setDictionary("address",  mdAddress)
collection.save(mutableDocument)
```

An example of using the Kotlin Serialization library to serialize an object to JSON string and then create a MutableDocument based on the JSON string value:
```kotlin
val propertyOwnerContact = Contact("Mr. Frog")
val country = Country("United States")
val address = Address(propertyOwnerContact, "123 Pond St", country)
val contact = Contact("Kermit", address)
val json = Json.encodeToString(contact)
val mutableDocument = MutableDocument("doc-1", json)
collection.save(mutableDocument);
```

### Create an Asymmetric Object
In Realm, asymmetric objects are objects that are inserted using a special API so that they are not included in the realm. Instead, they are directly published to Atlas App Services.  The use case from the docs point of view is sensor type information for IoT data.

```kotlin
// Open a write transaction
realm.write {
    // Create a new asymmetric object
    val weatherSensor = WeatherSensor().apply {
        deviceId = "WX1278UIT"
        temperatureInFarenheit = 6.7F
        barometricPressureInHg = 29.65F
        windSpeedInMph = 2
    }
    // Insert the object into the realm with the insert() extension method
    insert(weatherSensor)
// WeatherSensor object is inserted into the realm, then synced to the
// App Services backend. You CANNOT access the object locally because it's
// deleted from the local realm after sync is complete.
}
```
Couchbase Lite does not offer a direct API for certain specific operations. Instead, alternatives such as configuring a replicator in PUSH-only mode are available which would only send documents saved to the database to Capella App Services without pulling new documents down to the database. 

```kotlin
val replicatorConfig = ReplicatorConfigurationFactory.newConfig(
      target = "your url to app services",
      continuous = true,
      type = ReplicatorType.PUSH 
)
```

Couchbase Lite Replicator Type Documentation:
- [Replicator Sync Mode - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/replication.html#lbl-cfg-sync)
- [Replicator Sync Mode - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/replication.html#lbl-cfg-sync)
- [Replicator Sync Mode - C](https://docs.couchbase.com/couchbase-lite/current/c/replication.html#lbl-cfg-sync)
- [Replicator Sync Mode - Java](https://docs.couchbase.com/couchbase-lite/current/java/replication.html#lbl-cfg-sync)
- [Replicator Sync Mode - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/replication.html#lbl-cfg-sync)
- [Replicator Sync Mode - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/replication.html#lbl-cfg-sync)
- [Replicator Sync Mode - React Native](https://cbl-reactnative.dev/DataSync/remote-sync-gateway#sync-mode)
- [Replicator Sync Mode - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/replication.html#lbl-cfg-sync)


Finally for short-lived data, a document could be created with an expiration time and then deleted from the database after the expiration time has passed.  This purge is NOT replicated to App Services.

```kotlin
// Purge the document one day from now
let ttl = Calendar.current.date(
        byAdding: .day, value: 1, to: Date())

try collection.setDocumentExpiration
    (id: "doc123", expiration: ttl)
```

Couchbase Lite Document Expiration Documentation:
- [Document Expiration - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/document.html#document-expiration)
- [Document Expiration - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/document.html#document-expiration)
- [Document Expiration - C](https://docs.couchbase.com/couchbase-lite/current/c/document.html#document-expiration)
- [Document Expiration - Java](https://docs.couchbase.com/couchbase-lite/current/java/document.html#document-expiration)
- [Document Expiration - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#document-expiration)
- [Document Expiration - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/document.html#document-expiration)
- [Document Expiration - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#document-expiration)

### Create Realm Properties
Realm properties allow you to define [Realm-specific types](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/crud/create/#:~:text=Realm%2Dspecific%20types) to an object.  

Couchbase Lite features the MutableDocument API, which facilitates the mapping of fields to supported scalar types.  

Couchbase Lite Document API Documentation:
- [Document API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/document.html)
- [Document API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/document.html)
- [Document API - C](https://docs.couchbase.com/couchbase-lite/current/c/document.html)
- [Document API - Java](https://docs.couchbase.com/couchbase-lite/current/java/document.html)
- [Document API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html)
- [Document API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/document.html)
- [Document API - React Native](https://cbl-reactnative.dev/documents)
- [Document API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/document.html)

### Create a Relationship Property 

In the Atlas Device SDK, you can create a To-One or To-Many relationship between objects. An example of this:

```kotlin
// Pond class
open class Pond : RealmObject {
    @PrimaryKey
    var _id: ObjectId = ObjectId()
    var name: String = ""
    var location: String = ""
}

// Frog class
open class Frog : RealmObject {
    @PrimaryKey
    var _id: ObjectId = ObjectId()
    var name: String = ""
    var age: Int = 0
    // To-One relationships
    var favoritePond: Pond? = null
    var bestFriend: Frog? = null
}

// ... application code here
   
// Create a Pond object
val sunnyPond = Pond().apply {
        _id = ObjectId()  // MongoDB generate ObjectId automatically
        name = "Sunny Pond"
        location = "Near the forest"
}

    // Create Frog objects
val ribbit = Frog().apply {
        _id = ObjectId()
        name = "Ribbit"
        age = 6
}

val freddy = Frog().apply {
        _id = ObjectId()
        name = "Freddy"
        age = 5
        favoritePond = sunnyPond // To-One relationship with a Pond object
        bestFriend = ribbit      // To-One relationship with another Frog
}
```
When this data is synced to MongoDb Atlas, it would look something similar to this:

```json
{
  "_id": ObjectId("64f9..."),
  "name": "Freddy",
  "age": 5,
  "favoritePond": {
    "_id": ObjectId("64fa..."),
    "name": "Sunny Pond",
    "location": "Near the forest"
  },
  "bestFriend": {
    "_id": ObjectId("64fb..."),
    "name": "Ribbit",
    "age": 6
  }
}
```

To achieve the same results in Couchbase Lite, a developer would just make embedded JSON object or use the Key Value pair approach with dictionaries.  An example using serialization with an embedded JSON object is below: 

```kotlin
// Define serializable data classes
@Serializable
data class Pond(
    val id: String = UUID.randomUUID().toString(),
    val name: String = "",
    val location: String = ""
)

@Serializable
data class Frog(
    val id: String = UUID.randomUUID().toString(),
    val name: String = "",
    val age: Int = 0,
    val favoritePond: Pond? = null,
    val bestFriend: Frog? = null
)

// ...
// Save Frog as JSON to Couchbase Lite
fun saveFrogToCollection(collection: Collection, frog: Frog) {
    // Convert Frog object to JSON using Kotlinx.serialization
    val frogJson = Json.encodeToString(frog)
    // Convert JSON to a MutableDocument for Couchbase Lite
    val document = MutableDocument(frog.id, frogJson)
    // Save the document to Couchbase Lite
    collection.save(document)
}

// Retrieve Frog from Couchbase Lite
fun getFrogFromCollection(collection: Collection, frogId: String): Frog? {
    // Fetch the document by its ID
    val document = collection.getDocument(frogId) ?: return null
    // Convert JSON back to Frog object using Kotlinx.serialization
    val frogJson = document.toJSON()
    return frogJson?.let { Json.decodeFromString<Frog>(it) }
}

//  ...
// Create some Pond and Frog objects
val sunnyPond = Pond(name = "Sunny Pond", location = "Near the forest")
val ribbit = Frog(name = "Ribbit", age = 6)
val freddy = Frog(name = "Freddy", age = 5, favoritePond = sunnyPond, bestFriend = ribbit)

// Save Freddy to Couchbase Lite
saveFrogToCollection(database, freddy)

// Retrieve Freddy back from the database
val retrievedFrog = getFrogFromCollection(database, freddy.id)
println(retrievedFrog)
```

### Summary

Couchbase Lite utilizes the Documents and Blobs API to facilitate the storage of information within a Couchbase Lite database. Data storage can be efficiently managed using the MutableDocument class, which offers versatile data entry methods. Developers can set values either through key-value pairs, providing a clear and structured approach to data handling, or directly via a JSON string, which allows for rapid integration of complex data structures. This flexibility ensures that Couchbase Lite can accommodate a wide range of application needs, from simple data storage to complex data manipulation tasks.

## Read Realm Objects in Realm vs Couchbase Lite

### Read Operations

A read operation in Atlas Device SDK consists of querying database objects, then running the query when you are ready to access the results.  This is based on the [RealmQuery API](https://www.mongodb.com/docs/realm-sdks/kotlin/latest/library-base/io.realm.kotlin.query/-realm-query/index.html) and Realm Query Language (RQL).

An example of a query:
```kotlin
// Pass the object type as <T> parameter and filter by property
val findFrogs = realm.query<Frog>("age > 1")
    // Chain another query filter
    .query("owner == $0 AND name CONTAINS $1", "Jim Henson", "K")
    // Sort results by property
    .sort("age", Sort.ASCENDING)
    // Run the query
    .find()
// ... work with the results
```

Couchbase Lite can use the SQL++ query language to achieve the same results.  An example of using the Couchbase Lite Query API:

```kotlin
 val sqlQuery = """
        SELECT *
        FROM scopeName.collectionName as Frog
        WHERE age > 1
        AND owner = "Jim Henson"
        AND name LIKE "%K%"
        ORDER BY age ASC
    """.trimIndent()

// Create a query from the N1QL string
val query = database.createQuery(sqlQuery)

// Execute the query and collect results
val resultList = mutableListOf<Frog>()
query.execute().forEach { result ->
    val frogJson = result.getDictionary(0)?.toMap()?.let { Json.encodeToString(it) }
      frogJson?.let {
          val frog = Json.decodeFromString<Frog>(it)
          resultList.add(frog)
     }
}
```

### Query All Objects of a Type
In the Atlas SDK the query() function can be used to return all objects of a [specific type](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/crud/read/#query-all-objects-of-a-type).  

In Couchbase Lite the recommended approach would be to store documents by type of information into unique collections, thus you can just query a collection to get all the documents of that type.  For example, you could store all Frogs in a scope called animals and a collection called frogs, thus getting all frogs back from the database is a SQL query.

```sql
SELECT * FROM animals.frogs as Frog
```

### Query a Single Object/Filter by Primary Key
The first function on the query in the Atlas SDK can be used to return a single object of a specific type.  

```kotlin
val querySingleFrog = realm.query<Frog>().first()
val singleFrog = querySingleFrog.find()
if (singleFrog != null) {
    println("${singleFrog.name} is a frog.")
} else {
    println("No frogs found.")
}
```

Atlas also supports filtering objects by the PRIMARY_KEY_VALUE.  

```kotlin
val filterByPrimaryKey = realm.query<Frog>("_id == $0", PRIMARY_KEY_VALUE)
val findPrimaryKey = filterByPrimaryKey.find().first()
```

In Couchbase Lite you have a few options for getting a single document of a given type back.  To start out  with, if you know the document ID of a given document, you can just call the collection’s document function to get the document by its ID.  

```kotlin
val document = collection.getDocument(mutableDocument);
```

If a developer didn’t know the documentId, they could use the LIMIT keyword in SQL++ to get a single document back from a given collection.

```sql
SELECT * FROM animals.frogs as Frog LIMIT 1
```

### Filter By Embedded Object Property

To filter embedded documents in Atlas, you use the [dot notation](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/crud/read/#filter-by-embedded-object-property).

```kotlin
// Use dot notation to access the embedded object properties as if it
// were in a regular nested object
val filterEmbeddedObjectProperty =
    realm.query<Contact>("address.street == '123 Pond St'")
// You can also access properties nested within the embedded object
val queryNestedProperty = realm.query<Contact>()
    .query("address.propertyOwner.name == $0", "Mr. Frog")
```

In SQL++, you can also use dot notation to query fields of a sub-document.

```sql
SELECT * FROM exampleScope.Contact as contact WHERE address.street = '123 Pond St'

SELECT * FROM exampleScope.Contact as contact WHERE address.propertyOwner.name = 'Mr. Frog'
```

### Filter By RealmAny (Mixed) Property
A [RealmAny (Mixed) property](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/crud/read/#filter-by-embedded-object-property) represents a polymorphic value that can hold any one of its supported data types at a particular moment. 

```kotlin 
val filterByRealmAnyInt = realm.query<Frog>("favoriteThing.@type == 'int'")
val findFrog = filterByRealmAnyInt.find().first()
```

In Couchbase Lite, you can use [Type Check Functions](https://docs.couchbase.com/couchbase-lite/current/android/query-n1ql-mobile.html#lbl-func-typecheck) which can check a type and returns true if it matches and false if it doesn't match.

```sql
SELECT * FROM exampleScope.Frog as frog WHERE ISNUMBER(frog.favoriteThing)
```

### Filter by Full-Text Search (FTS)
In the [Atlas Device SDK](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/crud/read/#filter-by-full-text-search--fts--property), you can filter any property that is annotated with the [@FullText](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/property-annotations/#std-label-kotlin-fts-index) annotation.  

```kotlin
// Filter by FTS property value using 'TEXT'
// Find all frogs with "green" in the physical description
val onlyGreenFrogs =
    realm.query<Frog>("physicalDescription TEXT $0", "green").find()
// Find all frogs with "green" but not "small" in the physical description
val onlyBigGreenFrogs =
    realm.query<Frog>("physicalDescription TEXT $0", "green -small").find()
// Find all frogs with "muppet-" and "rain-" in the physical description
val muppetsInTheRain =
    realm.query<Frog>("physicalDescription TEXT $0", "muppet* rain*").find()
```
In Couchbase Lite, a Ful-Text Search index is created on a collection for the properties you would like to index.

```kotlin
val collection = database.getCollection("frogs", "animals")

collection.createIndex("idxFrogPhysicalDescription",
  FullTextIndexConfigurationFactory.newConfig
  ("physicalDescription")) 
    
```
Once the index is created you can use the SQL++ keyword MATCH OR RANK functions to query the collection using FTS
 
```sql
SELECT * FROM animals.frogs as frog 
WHERE MATCH(frog.physicalDescription, 'green')
```

Couchbase Lite documentation on Full-Text Search indexes can be found here:

- [Full-Text Search API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/fts.html)
- [Full-Text Search API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/fts.html)
- [Full-Text Search API - C](https://docs.couchbase.com/couchbase-lite/current/c/fts.html)
- [Full-Text Search API - Java](https://docs.couchbase.com/couchbase-lite/current/java/fts.html)
- [Full-Text Search API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/fts.html)
- [Full-Text Search API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/fts.html)
- [Full-Text Search API - React Native](https://cbl-reactnative.dev/full-text-search)
- [Full-Text Search API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/fts.html)

### Sort and Limit Results
In the Atlas SDK, you can sort and limit results using the sort(), limit(), and distinct() functions.

```kotlin
val organizedWithMethods = realm.query<Frog>("owner == $0", "Jim Henson")
    .sort("age", Sort.DESCENDING)
    .distinct("name")
    .limit(2)
    .find()
organizedWithMethods.forEach { frog ->
    Log.v("Method sort: ${frog.name} is ${frog.age}")
}
```

In Couchbase Lite, you can use the SQL++ ORDER BY and LIMIT keywords to sort and limit results.

```sql
SELECT DISTINCT(*) FROM animals.frogs as frog WHERE frog.owner = 'Jim Henson' ORDER BY age DESC LIMIT 2
```

### Aggregate Results
The Atlas Device SDK uses  RQL aggregate operators, one of the following convenience methods, or a combination of both:
- max()
- min()
- sum()
- count()

In Couchbase Lite, SQL++ provides a plethora of aggregate operators and functions.  
Couchbase Lite SQL++ for Mobile Developers Documentation: 
- [SQL++ for Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/query-n1ql-mobile.html)
- [SQL++ for Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/query-n1ql-mobile.html)
- [SQL++ for C](https://docs.couchbase.com/couchbase-lite/current/c/query-n1ql-mobile.html)
- [SQL++ for Java](https://docs.couchbase.com/couchbase-lite/current/java/query-n1ql-mobile.html)
- [SQL++ for .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/query-n1ql-mobile.html)
- [SQL++ for Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/query-n1ql-mobile.html)
- [SQL++ for React Native](https://cbl-reactnative.dev/sqlplusplus)
- [SQL++ for Swift](https://docs.couchbase.com/couchbase-lite/current/swift/query-n1ql-mobile.html)

### Summary

Couchbase Lite utilizes SQL++, an extension of the industry-standard SQL, to query information from its database, ensuring compatibility and ease of use with familiar syntax and robust capabilities. In contrast, the Atlas Device SDK relies on the Realm Query Language (RQL), a proprietary query language specific to Realm databases.