# Atlas Device SDK to Couchbase Lite Terminology and Technology comparison
The Atlas Device SDK can be thought of as an on-device ORM that creates an “object store” that is then synced to MongoDb Atlas.  The following document describes the features of Atlas and how they compare to Couchbase Lite and Couchbase Capella App Services. 

>**NOTE** 
>
>Links to the Atlas SDK are linked to the Kotlin SDK specifically.  The Atlas SDK is available in multiple languages and the documentation can be found [here](https://www.mongodb.>com/docs/atlas/device-sdks/).  All code examples are provided in Kotlin for Android.
>
 
## Frozen Architecture

Couchbase Lite doesn’t use the [frozen architecture](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/frozen-arch/) paradigms, which was created as a way to pass objects between threads safely.

For most applications given that Couchbase Lite’s database is a single file, it is recommended to run all database operations on a single thread (NOT THE UI Thread) and the Couchbase Lite SDK will handle doing the work it needs to on however threads it needs to do work for processes like replication.

**Example:  Couchbase Lite Query API with the Kotlin SDK**

In Couchbase Lite, the Query API is designed to integrate seamlessly with platform-specific mechanisms for dealing with loading and saving data. For instance, when using the Couchbase Lite Kotlin SDK, the Coroutines and Flow APIs in Kotlin can be leveraged out of the box without any modifications. 

It is important to follow Android best practices for threading, especially when performing I/O operations. For example, it is recommended to use a single Dispatcher scope for database tasks, and Dispatchers.Main or ViewModelScope for UI-related tasks. Proper threading ensures efficient and responsive database interactions without blocking the UI thread.



## Model Data/Object Store

Realm relies on the developer to define "[Realm objects](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#realm-objects)" or objects that are uniquely named that they can interact with Atlas Device and Device synchronization.  Couchbase Lite is flexible in that all data is stored in the database as a JSON document.  You can use SQL++ Queries to get information from the database OR you can query the database using a documentId which can be similarly thought of as Realms “ObjectId”.

Couchbase Lite you save information to the database in a JSON Document.  Within Couchbase Lite there is the Document API that allows you to either set fields on a document to values (KV operations), or you can serialize an object down to JSON and then create the document based on the JSON values.  You can reverse take the document from JSON and then serialize it to an object.

When you embed documents in Realm you use the [EmbeddedRealmObject](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-embedded-object-type), whereas in Couchbase Lite we just think of embedded data as an embedded document.

[Asymmetric Object Types](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-asymmetric-object-type) are unique to Realm and Atlas Device sync as a way to control insert-only documents.

All documents inserted into a Couchbase Lite database are a document.  When you retrieve a document from the Couchbase Lite Database using the Collection API and provide a documentId, it’s a Document object (immutable) until you convert it to a MutableDocument which would then allow you to make changes to it.  

Couchbase Lite and App Service/Sync Gateway have a different [security model](https://docs.couchbase.com/sync-gateway/current/access-control-model.html) that handles what documents should be synced via documents being assigned to a [“channel”](https://docs.couchbase.com/sync-gateway/current/channels.html) and the [replication configuration](https://docs.couchbase.com/couchbase-lite/current/android/replication.html) to decide which documents in what scopes and collections should be synced vs stay internal to the local database on the device.


### Realm Object Types

| Name | Documentation| 
|---|---|
| RealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#realm-objects)                     |
| EmbeddedRealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-embedded-object-type)    |
| AsymmetricRealmObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-an-asymmetric-object-type) |

### Collection Types

In Realm there are three major collection types:  RealmList, RealmSet, and RealmDictionary.  There is no direct comparison between Realm’s collection types and Couchbase Lite.  

Couchbase Lite does support querying the database and when returning results they come back in a ResultSet.  A result set can be iterated through and return the Result object, which can then be type-casted to a custom Object type defined in the application, or can be converted to JSON via the Result.toJSON function call. Once it’s a JSON string, any serialization library for the given platform can be used.

The ResultSet documentation can be found here:
- [ResultSet API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/query-resultsets.html)
- [ResultSet API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/query-resultsets.html)
- [ResultSet API - C](https://docs.couchbase.com/couchbase-lite/current/c/query-resultsets.html)
- [ResultSet API - Java](https://docs.couchbase.com/couchbase-lite/current/java/query-resultsets.html)
- [ResultSet API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/query-resultsets.html)
- [ResultSet API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/query-resultsets.html)
- [ResultSet API - React Native](https://cbl-reactnative.dev/Queries/query-result-set)
- [ResultSet API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/query-resultsets.html)


### Summary

In summary, the developer has to do a lot of planning of what type of object they should be inheriting from in order to use Realm and the Atlas Device SDK.  In Couchbase Lite, all information is stored in the database as a JSON document, and you can pick the pattern on how you want to write and read documents from the database.  

## Collections
In Realm, objects are stored in collections based on the object type (all objects in a collection are of the same type) unless the developer is using the [Realm Unstructured](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#define-unstructured-data) data API (RealmAny).  Storing data in this way comes at a performance cost in Realm.

Couchbase Lite supports Scopes and Collections for storing JSON documents.  A scope is a container of 1 or many Collections, up to 1,000 collections per scope.  Collections are a group of JSON documents, whose contents can be different (schema doesn’t have to be the same).  Storing documents with different schemas in the same Collection comes at no performance loss in Couchbase Lite and SQL++ has keywords to help you query for documents that are missing a property(field).

The Scopes and Collections documentation for Couchbase Lite can be found here:
- [Scopes - Collections API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/scopes-collections-manage.html)
- [Scopes - Collections API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/scopes-collections-manage.html)
- [Scopes - Collections API - C](https://docs.couchbase.com/couchbase-lite/current/c/scopes-collections-manage.html)
- [Scopes - Collections API - Java](https://docs.couchbase.com/couchbase-lite/current/java/scopes-collections-manage.html)
- [Scopes - Collections API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/scopes-collections-manage.html)
- [Scopes - Collections API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/scopes-collections-manage.html)
- [Scopes - Collections API - React Native](https://cbl-reactnative.dev/scopes-collections)
- [Scopes - Collections API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/scopes-collections-manage.html)

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

Couchbase Lite Data Types documentation can be found here:

- [Android - Java](https://docs.couchbase.com/couchbase-lite/current/android/document.html#data-types)
- [Android - Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/document.html#data-types)
- [C](https://docs.couchbase.com/couchbase-lite/current/c/document.html#data-types/)
- [Java](https://docs.couchbase.com/couchbase-lite/current/java/document.html#data-types)
- [.NET](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#data-typess)
- [Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/document.html#data-types)
- [React Native](https://cbl-reactnative.dev/documents#document-structure)
- [Swift](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#data-types)

### ObjectId vs DocumentId

Realm objects can use the [ObjectId](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#objectid) or [UUID](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#realmuuid) for unique identifiers for Realm objects.  The ObjectId is a 12-byte unique value that is [nullable](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#objectid), indexable, and used as a primary key.  
In Couchbase Lite a documentId can be any string value as long as it’s not null.  A document created without an ID will always have a randomly generated UUID used for the documentId.

### Geospatial Types
Couchbase Lite has no direct feature or support for Geospatial data types.  A developer would need to write custom code to handle any geospatial calculations such as querying the database for all documents from a certain distance of a given Geospatial location.

### Property Annotations
 In realm, property annotations are used to define the schema of the object and how it should be stored in the database including which fields are indexed.
 
Couchbase Lite has no direct feature or support for Property Annotations because documents are stored in JSON format, thus no need for them.

In Couchbase Lite, Indexes are created outside the storage API via the Index API.

Couchbase Lite documentation on the Index API can be found here:

- [Indexing API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/indexing.html)
- [Indexing API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/indexing.html)
- [Indexing API - C](https://docs.couchbase.com/couchbase-lite/current/c/indexing.html)
- [Indexing API - Java](https://docs.couchbase.com/couchbase-lite/current/java/indexing.html)
- [Indexing API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/indexing.html)
- [Indexing API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/indexing.html)
- [Indexing API - React Native](https://cbl-reactnative.dev/indexes)
- [Indexing API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/indexing.html)


### Backlinks and Relationships
In Realm a [backlink](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/supported-types/#backlinks) is an inverse, to-many relationship between objects.  Backlinks can not be null.  MongoDb specifically states they don’t use bridge tables or explicit joins to define relationships.   

In Couchbase Lite, a developer can use a document with a documentId of two different documents to bridge documents for a join, a documentId reference in another document to do a join (foreign key), or you can embed documents in other documents.  

Embedded documents into other documents will always come at the cost of large documents.  When dealing with smaller documents, a relationship via documentId can be used.  Relationships between different documents are typically modeled using document IDs which can kind of be thought of as a foreign keys to represent links between documents. To infer relationships , you would need to use a reverse lookup mechanism by querying the database to find documents that reference a particular document using the SQL++ JOIN keyword.  By using Indexes on those fields, you don’t take a performance hit when running these kinds of queries.

Couchbase Lite documentation on SQL++ JOIN keyboard can be found here:

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
In Realm and Atlas Device sync a lot of work goes into understanding the schema of data and defining the device sync data model.  Couchbase Lite is a Document based database which JSON documents are flexible meaning you have to spend a lot less time defining these and thus can handle any data type restrictions in your applications business logic.

## Configuring and Opening Realm vs Couchbase Lite Database

### Realm File
Realm stores a binary-encoded version of every object and type in a single .realm file.  When you open a .realm file it will create one if it doesn’t already exist.  The file is located in a path the developer specifies.

### Couchbase Lite Database

Couchbase Lite stores all JSON documents in a single database that is a filename with a cblite2 extension.  When you open a database a new one will be created if it doesn’t already exist on the disk.  The developer specifies the path to the database, or a default path is used by the SDK when no path is provided.

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

- [Database Maintenance API - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/database.html#lbl-db-util)
- [Database Maintenance API - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/database.html#lbl-db-util)
- [Database Maintenance API - C](https://docs.couchbase.com/couchbase-lite/current/c/database.html#lbl-db-util)
- [Database Maintenance API - Java](https://docs.couchbase.com/couchbase-lite/current/java/database.html#lbl-db-util)
- [Database Maintenance API - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/database.html#lbl-db-util)
- [Database Maintenance API - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/database.html#lbl-db-util)
- [Database Maintenance API - React Native](https://cbl-reactnative.dev/databases#database-maintenance)
- [Database Maintenance API - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/database.html#lbl-db-util)

### Add Initial Data to Realm
Realm provides an [API](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/open-and-close-a-realm/#add-initial-data-to-realm) for inserting data into the realm file when the realm is opened for the first time.  

There isn’t a pattern in Couchbase Lite that maps one-to-one to this, however, you can create a database based on a Pre-built Database, which would achieve the same results.  A pre-built database allows you to load your app with data instead of syncing it from App Services during startup to minimize customer wait time on initial install and launch of the application.  Each platform in Couchbase Lite has specific documentation on Pre-built Database files:

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
 
An example on how to copy a database file is provided in the Couchbase Lite documentation.

- [Database Copy - Android-Java](https://docs.couchbase.com/couchbase-lite/current/android/prebuilt-database.html#deploy-db)
- [Database Copy - Android-Kotlin](https://docs.couchbase.com/couchbase-lite/current/android/prebuilt-database.html#deploy-db)
- [Database Copy - C](https://docs.couchbase.com/couchbase-lite/current/c/prebuilt-database.html#deploy-db)
- [Database Copy - Java](https://docs.couchbase.com/couchbase-lite/current/java/prebuilt-database.html#deploy-db)
- [Database Copy - .NET](https://docs.couchbase.com/couchbase-lite/current/csharp/prebuilt-database.html#deploy-db)
- [Database Copy - Objective-C](https://docs.couchbase.com/couchbase-lite/current/objc/prebuilt-database.html#deploy-db)
- [Database Copy - React Native](https://cbl-reactnative.dev/database-prebuilt#using-pre-built-database-on-app-launch)
- [Database Copy - Swift](https://docs.couchbase.com/couchbase-lite/current/swift/prebuilt-database.html#deploy-db)

### Summary

The Realm API is similar to the Couchbase Lite Database API when it comes to opening, closing, and copying a database.  

The Couchbase Lite Database Documentation can be found here:
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
> You will always take a small performance hit when serializing and deserializing objects to and from JSON.  If performance is a concern, it's best to use the Key Value pair approach when creating documents in Couchbase Lite.
> 

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

In Couchbase Lite, embedded objects are just embedded documents which you can use KV or the JSON serialization method.  An example of KV:
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

An example of Serialization:
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
Couchbase Lite provides no direct API to achieve the same results.  Instead, a replicator could be configured to only PUSH mode and not pull and then the database documents could be pruned from time to time, or even have the application create a new database at a regular interval depending on the applications need and performance.

```kotlin
val replicatorConfig = ReplicatorConfigurationFactory.newConfig(
      target = "your url to app services",
      continuous = true,
      type = ReplicatorType.PUSH 
)
```

### Create Realm Properties
Realm properties allow you to define [Realm-specific types](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/crud/create/#:~:text=Realm%2Dspecific%20types.) to an object.  

Couchbase Lite provides the equivalent with the MutableDocument API and types it supports.  Couchbase Lite’s documentation explains how to use MutableDocuments along with data in Dictionaries, Array’s and Blob’s.

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

To achieve the same results in Couchbase Lite, a developer would just make embedded documents. 

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

Couchbase Lite uses the Documents and Blobs API to save information into a Couchbase Lite database.   Information can be saved using the MutableDocument class which allows for setting values via Key Value Pairs or via a JSON String.

