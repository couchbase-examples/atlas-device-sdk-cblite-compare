# Atlas Device SDK to Couchbase Lite Comparison Guide
The Atlas Device SDK can be thought of as an on-device ORM that creates an “object store” that is then synced to MongoDb Atlas.  The following document describes the features of Atlas and how they compare to Couchbase Mobile.

>**NOTE**
>
>This document is designed for developers using the .NET Framework and the C# language for building applications.  
>
> 
## Configuring and Opening Realm vs Couchbase Lite Database

### Realm File

Realm stores a binary-encoded version of every object and type in a [single .realm](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/realm-files/realms/#realm-files) file.  When you open a .realm file it will create one if it doesn’t already exist.  The file is located in a path the developer specifies.

### Couchbase Lite Database

Couchbase Lite stores all JSON documents in a single database that is a filename with a cblite2 extension (technically, if you were to look at the file on disk you would see it's a directory, but you should never mess with or attempt to open the files in the directory).

When you open a database a new cblite2 file will be created if it doesn’t already exist on the disk.  The developer specifies the path to the database, or a default path is used by the SDK when no path is provided.

#### Tools for viewing Database contents
Couchbase Lite has a few ways to view the contents of a database file outside your application.

- [JetBrains IDE Plugin for Rider](https://plugins.jetbrains.com/plugin/22131-couchbase)
- [Visual Studio Code Plugin](https://marketplace.visualstudio.com/items?itemName=Couchbase.vscode-cblite)
- [cblite Command Line Tool](https://github.com/couchbaselabs/couchbase-mobile-tools/blob/master/Documentation.md)

### Opening a Realm
To open a realm, you must provide a [RealmConfiguration](https://www.mongodb.com/docs/realm-sdks/dotnet/latest/reference/Realms.RealmConfiguration.html#Realms_RealmConfiguration__ctor_System_String_) object that defines the details of the realm.  This configuration is passed to the *Realm.GetInstance* function.

```csharp
// Create a realm configuration with configuration builder
// and pass all optional arguments
var config = new RealmConfiguration(pathToDb + "my.realm")
{
    IsReadOnly = true,
};
Realm localRealm;
try
{
    localRealm = Realm.GetInstance(config);
}
catch (RealmFileAccessErrorException ex)
{
    Console.WriteLine($@"Error creating or opening the
    realm file. {ex.Message}");
}
// ... use the open realm ...
```

### Opening a Couchbase Lite Database
To open a Couchbase Lite database or create a new database, you can provide a DatabaseConfiguration object that defines the directory and an encryption key used to secure the database.  This configuration is passed on to the Database API to open the database.

```csharp
// Create a new, or open an existing database with encryption enabled
var config = new DatabaseConfiguration
        {
            // Or, derive a key yourself and pass a byte array of the proper size
            Directory = pathToDb 
            EncryptionKey = new EncryptionKey("password")
        };

using var database = new Database("mydb", config);
```

#### More Information
- [Couchbase Lite Database Encryption](https://docs.couchbase.com/couchbase-lite/current/csharp/database.html#database-encryption)

### Custom Realm Configuration
In Realm, a custom configuration can be based to trigger additional functionality:
- Compacting a realm to reduce its file size
- Specifying a schema version or migration block for making schema changes
- Flagging a realm to be deleted if a migration is required

As stated before, Couchbase Lite doesn’t need schema management or migration plans, so there is no equivalent to this.  Couchbase Lite databases have a Database Maintenance API which can be used to run tasks like compacting the database or to rebuild indexes.

#### More Information
- [Couchbase Lite Database Maintenance API](https://docs.couchbase.com/couchbase-lite/current/csharp/database.html#lbl-db-util)

### Scoping the Realm 
The Realm SDK realm object [implements IDisposable](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/realm-files/realms/#scoping-the-realm) interface to ensure native resources are freed up. 

The Couchbase Lite .NET SDK provides a [close](https://docs.couchbase.com/couchbase-lite/current/csharp/database.html#close-database) function which will also validate that the replicator and change listeners are closed before closing the connection.  The [Database](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-net/api/Couchbase.Lite.Database.html) class also implements the IDisposable interface to ensure native resources are freed up. 

### Copying Data into a New Realm / Couchbase Lite Database Copy
The [WriteCopy](https://www.mongodb.com/docs/realm-sdks/dotnet/latest/reference/Realms.Realm.html#Realms_Realm_WriteCopy_Realms_RealmConfigurationBase_) method allows developers to copy data from an existing realm to a new realm.  There are some rules to this which are well documented.

In Couchbase Lite there is a [Database.Copy](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-net/api/Couchbase.Lite.Database.html#Couchbase_Lite_Database_Copy_System_String_System_String_Couchbase_Lite_DatabaseConfiguration_) function that can copy a database file to a new database file.

> **NOTE**
> You should NEVER copy a Couchbase Lite database outside using the Database.Copy API as it will invalidate other apps checkpoints.
>

#### More Information
- [Couchbase Lite - Database Copy](https://docs.couchbase.com/couchbase-lite/current/csharp/prebuilt-database.html#deploy-db)

### Summary

Both the Realm API and the Couchbase Lite Database API exhibit comparable functionalities when managing databases, particularly in terms of opening, closing, and copying database instances.

#### More Information
- [Couchbase Lite - Database API](https://docs.couchbase.com/couchbase-lite/current/csharp/database.html)

## Model Data/Object Store

Realm relies on the developer to define [Realm objects](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/schemas/define-realm-object-model/#realm-objects) or objects that are uniquely named that they can interact with Atlas Device and Device synchronization.

### Realm Object Types

All Realm objects inherit from the interfaces listed below and must be declared as **partial** classes.

| Name                   | Documentation| 
|------------------------|---|
| IRealmObject           | [Docs](https://www.mongodb.com/docs/realm-sdks/dotnet/latest/reference/Realms.IRealmObject.html)                     |
| IEmbeddedRealmObject   | [Docs](https://www.mongodb.com/docs/realm-sdks/dotnet/latest/reference/Realms.IEmbeddedObject.html)    |
| IAsymmetricRealmObject | [Docs](https://www.mongodb.com/docs/realm-sdks/dotnet/latest/reference/Realms.IAsymmetricObject.html) |

Couchbase Lite is flexible in that all data is stored in the database as a JSON document using the Document API.  You can use SQL++ Queries to get information from the database OR you can query the database using a documentId which can be similarly thought of as Realms “ObjectId”.

When you embed documents in Realm you use the [IEmbeddedObject](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/model-data/data-types/embedded-objects/) interface, whereas in Couchbase Lite you can  think of embedded data as an embedded JSON document or dictionary.

### Collection Types

Realm supports three primary collection types: RealmList, RealmSet, and RealmDictionary. Each serves specific use cases for managing collections of objects in a database. Couchbase Lite, in contrast, does not have direct analogs to these collection types but instead supports flexible data structuring through JSON. This allows for storing arrays and dictionaries (akin to RealmList and RealmDictionary, respectively) directly within JSON documents.

When querying data in Couchbase Lite with SQL++, it retrieves results in a ResultSet object, which can be iterated to extract individual Result objects.  Results are, essentially, JSON documents and can be treated as Dictionaries, wrapped in application objects, or serialized to JSON.

These results can then be directly cast to custom object types defined within the application, facilitating seamless integration with the app’s data architecture. Additionally, results can be converted into JSON strings using the Result.toJSON() function. This JSON data can subsequently be processed using any serialization library compatible with the platform in use, providing extensive flexibility for data manipulation and integration.

#### More Information
- [Couchbase Lite ResultSet API](https://docs.couchbase.com/couchbase-lite/current/csharp/query-resultsets.html)

### Summary

In using Realm and the Atlas Device SDK, developers must carefully plan their data model, particularly regarding the types of objects they inherit from, to effectively leverage the database’s capabilities. This upfront planning is crucial due to the structured nature of the data and the object-oriented approach of Realm.

Conversely, Couchbase Lite offers a flexible data handling approach. All information is stored as JSON documents in the database. This format allows developers to freely choose how they structure, write, and read their data. The flexibility of JSON documents enables a variety of patterns and structures to be implemented, adapting easily to the evolving needs of the application and reducing the need for extensive initial planning.

## Collections
In Realm, objects are stored in [collections](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/model-data/data-types/collections/#working-wth-collections) based on the object type (all objects in a collection are of the same type). 

Couchbase Lite supports Scopes and Collections for storing JSON documents.  A scope is a namespace that may contain up to 1,000 collections.  Collections are a group of JSON documents, whose contents can be different (schema doesn’t have to be the same).  Storing documents with different schemas in the same Collection comes at no performance loss in Couchbase Lite and SQL++ has keywords to help you query for documents that are missing a property(field).

#### More Information
- [Couchbase Lite Scopes - Collections API](https://docs.couchbase.com/couchbase-lite/current/csharp/scopes-collections-manage.html)

### Supported Data Types/Data Modeling
In Realm each platform the SDK supports as a given set of supported Field Types:
- [Field Types](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/model-data/data-types/field-types/)

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

#### More Information
- [Couchbase Lite Data Types](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#data-types)

### ObjectId vs DocumentId

Realm objects can use the [ObjectId or Guid](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/model-data/data-types/field-types/#guid-and-objectid-properties).  The ObjectId is a 12-byte unique value, indexable, and used as a primary key.

In Couchbase Lite a [documentId](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#document-structure) can be any string value as long as it’s not null.  A document created without an ID will always have a randomly generated a GUID to be used for the documentId.

>**NOTE**
>
>Developers convert data from MongoDb Atlas to Couchbase Capella will need to take this into account when converting the _id field in a given Realm object to a DocumentId.
>

### Property Annotations
In Realm, property annotations play a critical role in defining the schema of objects. These annotations specify how objects should be stored in the database, including details on which fields are indexed, thereby enhancing query performance and ensuring data integrity.

On the other hand, Couchbase Lite operates differently due to its use of JSON documents for storage. Instead of using property annotations to define schemas and index fields directly within the objects, Couchbase Lite manages indexing through a separate Index API. This approach allows developers to create and manage indexes independently of the document storage API, providing flexibility in how data is indexed and retrieved without altering the document structure itself.

#### More Information
- [Couchbase Lite Populating a Document](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#populate-a-document)
- [Couchbase Lite Indexing API](https://docs.couchbase.com/couchbase-lite/current/csharp/indexing.html)


### Backlinks and Relationships
In Realm, a [backlink](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/model-data/relationships/#inverse-relationship) defines an inverse, to-many relationship between objects, where backlinks cannot be null. Unlike MongoDB, which explicitly avoids using bridge tables or explicit joins for defining relationships, Realm utilizes its proprietary mechanisms.

Couchbase Lite, on the other hand, offers two approaches for managing document relationships. Firstly, it allows for embedding documents directly within other documents, akin to the feature provided by the Atlas Device SDK via embedded dictionaries in a document. This method, while straightforward, may lead to the creation of large document sizes, which can be inefficient for certain operations. Alternatively, Couchbase Lite supports the use of document IDs to establish relationships between documents. This method involves storing a document ID within another document, effectively using it as a foreign key to facilitate joins.

To manage relationships between smaller documents or to ensure efficient querying, developers might prefer using document IDs. These IDs serve as references that link documents together, akin to foreign keys in relational databases. To resolve these relationships, developers can employ SQL++ JOIN operations, supported by Couchbase Lite’s query language. By indexing the relevant document ID fields, performance impacts during such queries are minimized, ensuring efficient data retrieval across linked documents.

#### More Information
- [Couchbase Lite SQL++ JOIN](https://docs.couchbase.com/couchbase-lite/current/csharp/query-n1ql-mobile.html#lbl-join)

### Changes to Object Model (schema synchronization)
Realms [documentation](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/model-data/change-an-object-model/) has a detailed explanation of when and when you don’t have to worry about schema changes that is out of the scope of this document.

Couchbase Lite is a Documents based database, in which documents are stored in [JSON](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#lbl-json-data), so you never have to be concerned about schema changes.

### Summary
In both Realm and the Atlas Device SDK, considerable effort is dedicated to comprehensively understanding the data schema and meticulously defining the synchronization model for devices. This process is crucial for ensuring that the data structures are correctly aligned with the operational requirements and synchronization protocols.

Conversely, Couchbase Lite is a document-oriented database that leverages the inherent flexibility of JSON documents. This flexibility significantly reduces the time required for schema definition, allowing developers to dynamically accommodate various data types. Consequently, any constraints or data type regulations can be efficiently managed within the application’s business logic layer, rather than at the database level. This approach not only simplifies initial setup but also enhances adaptability to evolving data requirements.

## Create & Write Data in Realm vs Couchbase Lite

### Writing Data
In Realm, you write data in a transaction which is a block of code that can do multiple sub-transactions.  Either all the transactions succeed or they fail.   An example of this is below:
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

#### More Information
Couchbase Lite Document Batch API Documentation:
- [Couchbase Lite - Batch Operations](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#batch-operations)

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

#### More Information
- [Couchbase Lite - Replicator Sync Mode](https://docs.couchbase.com/couchbase-lite/current/csharp/replication.html#lbl-cfg-sync)

Finally for short-lived data, a document could be created with an expiration time and then deleted from the database after the expiration time has passed.  This purge is NOT replicated to App Services.

```kotlin
// Purge the document one day from now
let ttl = Calendar.current.date(
        byAdding: .day, value: 1, to: Date())

try collection.setDocumentExpiration
    (id: "doc123", expiration: ttl)
```

#### More Information
- [Couchbase Lite - Document Expiration](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#document-expiration)

### Create Realm Properties
Realm properties allow you to define [Realm-specific types](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/crud/create/#:~:text=Realm%2Dspecific%20types) to an object.

Couchbase Lite features the MutableDocument API, which facilitates the mapping of fields to supported scalar types.

- [Couchbase Lite - Document API](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html)

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

In Couchbase Lite, you can use [Type Check Functions](https://docs.couchbase.com/couchbase-lite/current/csharp/query-n1ql-mobile.html#lbl-func-typecheck) which can check a type and returns true if it matches and false if it doesn't match.

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

#### More Information
- [Couchbase Lite - Full-Text Search API](https://docs.couchbase.com/couchbase-lite/current/csharp/fts.html)

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
##### More Information
- [Couchbase Lite - SQL++ for Mobile Developers](https://docs.couchbase.com/couchbase-lite/current/csharp/query-n1ql-mobile.html)

### Summary

Couchbase Lite utilizes SQL++, an extension of the industry-standard SQL, to query information from its database, ensuring compatibility and ease of use with familiar syntax and robust capabilities. In contrast, the Atlas Device SDK relies on the Realm Query Language (RQL), a proprietary query language specific to Realm databases.

## Updating Realm Objects in Realm vs Couchbase Lite

### Updating Realm Objects

Realm supports update and upsert operations on Realm objects and embedded objects. An upsert operation either inserts a new instance of an object or updates an existing object that meets certain criteria.  The syntax to update an object in a realm is the same for a local or a synced realm. However, there are additional considerations that determine whether the write operation in a synced realm is successful.  Realm requires operations for updating are done in the realm.write or realm.writeBlocking transaction block.

In Couchbase Lite updating a document is almost the same as workflow as creating a document.  You can use the MutableDocument API to update a document in a collection.  To update a document, you first retrieve the document from the collection, make the changes to the document, and then save the document back to the collection.

```kotlin
val collection = database.getCollection("frogs", "animals")
collection.getDocument("frogId::kermit")
    .toMutable()
    ?.let { mutableDocument ->
        mutableDocument.setString("name", "Kermit the Frog")
        collection.save(mutableDocument)
    }
```

### Updating many documents

As previously discussed, Couchbase Lite offers a robust [batch API](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#batch-operations) that enables the insertion or updating of multiple documents within a single transactional operation. This capability ensures atomicity at the local database level, meaning that no other Database instances, including those managed by the replicator, can modify the database during the execution of the transactional block. Additionally, other instances are prevented from observing any intermediate states of the transaction.

However, it is important to recognize that Couchbase Mobile operates as part of a distributed system. Consequently, despite the transactional integrity maintained at the local database level, the asynchronous nature of the replication process does not guarantee simultaneous visibility of changes across Sync Gateway or other devices.

## Deleting Realm Objects in Realm vs Couchbase Lite

### Deleting Realm Objects

In Realm and the Atlas Device SDK you can choose to delete a single object, multiple objects, or all objects from the realm. After you delete an object, you can no longer access or modify it.

```kotlin
// Open a write transaction
realm.write {
    // Query the Frog type and filter by primary key value
  val frogToDelete: Frog = query<Frog>("_id == $0", PRIMARY_KEY_VALUE)
                              .find()
                              .first()
    // Pass the query results to delete()
  delete(frogToDelete)
}
```

In Couchbase Lite, the Collection's API provides a few different functions based on the developers need.  The first API is the Collection API delete document function.

```kotlin
val collection = database.getCollection("frogs", "animals")
collection.getDocument("frogId::kermit")
    .let { 
        collection.delete(it)
    }
```

Optionally, when saving or deleting a document to the database you can pass in a concurrency control token. This is done by passing in the concurrency control token to the save or delete function.

```kotlin
val collection = database.getCollection("frogs", "animals")
collection.getDocument("frogId::kermit")
    .let { 
        val didFailDelete = collection
            .delete(it, 
                    ConcurrencyControl.FAIL_ON_CONFLICT)
    }
```
Note that when specifying the failOnConflict concurrency control, and conflict occurred, the delete operation will fail with 'false' value returned.

When calling the delete function, that deletion will propagate to the server and to all the other devices that sync with that database.  Calling delete is actually identical to saving a document with a body that is empty except for a single property "_deleted" set to true.

### Purging a Document

Couchbase Lite also provides another API to remove a document from the database.  This API is called purge.  Purging a document is more extreme than calling the delete API: it actually deletes the document and all its metadata. There’s nothing left to indicate the document ever existed.  For this reason, though, calling purge doesn’t replicate.

```kotlin
val collection = database.getCollection("frogs", "animals")
collection.getDocument("frogId::kermit")
    .let { 
        collection
            .delete(it, 
                    ConcurrencyControl.FAIL_ON_CONFLICT)
    }
```


## Manging Realm Files vs Couchbase Lite Database

### Reduce File Size

Starting in version 1.6.0 of the Atlas Device SDK, realm automatically compacts Realm files in the background.  Automatic compaction begins when the size of unused space in the file is more than twice the size of user data in the file. Automatic compaction only takes place when the file is not being accessed.

Couchbase Lite also provides the Database Maintenance API that can provide several functions, including a compact function that shrinks the database file by removing any empty pages, and deletes blobs that are no longer referenced by any documents.

```kotlin
database.performMaintenance(DatabaseMaintenance.COMPACT) 
```

#### More Information
- [Couchbase Lite - Database Maintenance](https://docs.couchbase.com/couchbase-lite/current/csharp/database.html#lbl-db-util)

### Delete a Realm File

To safely delete a realm file while the app is running, you can use the Realm.deleteRealm() method. The following code demonstrates this:
```kotlin
// You must close a realm before deleting it
realm.close()
// Delete the realm
Realm.deleteRealm(config)
```

In Couchbase Lite, there is a function provided to delete a database.  Deleting a database will stop all replicators, live queries and all listeners attached to it. Although attempting to close a closed database is not an error, attempting to delete a closed database is.
```kotlin
database.delete()
```

#### More Information
- [Couchbase Lite - Delete Database](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-net/api/Couchbase.Lite.Database.html#Couchbase_Lite_Database_Delete)

##  React to Changes Realm vs Couchbase Lite

### Change Events

In Realm, you can subscribe to changes on the following [events](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/react-to-changes/):

- [Query on collection](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/react-to-changes/#std-label-kotlin-query-change-listener)
- [Realm object](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/react-to-changes/#std-label-kotlin-realm-object-change-listener)
- [Realm collections](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/react-to-changes/#std-label-kotlin-realm-list-change-listener) (e.g list)

The Atlas Device SDK only provides notifications for objects nested up to four layers deep. If you need to react to changes in more deeply-nested objects.

Couchbase Lite provides change listeners for the following events:
- [Query Listener (Live Query)](https://docs.couchbase.com/couchbase-lite/current/csharp/query-live.html)
- [Document Listener](https://docs.couchbase.com/couchbase-lite/current/csharp/document.html#document-change-events)
- [Collection Listener](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-net/api/Couchbase.Lite.Collection.html#Couchbase_Lite_Collection_AddChangeListener_System_EventHandler_Couchbase_Lite_CollectionChangedEventArgs__)
- [Replication Status Listener](https://docs.couchbase.com/couchbase-lite/current/csharp/replication.html#lbl-repl-status)
- [Replication Document Listener](https://docs.couchbase.com/couchbase-lite/current/csharp/replication.html#lbl-repl-evnts)

### Summary

Couchbase Lite offers a suite of change listener APIs that enable developers to implement event-driven architectures by responding programmatically to modifications in collections, documents, queries, and the replication process. These event listeners facilitate real-time data handling by triggering callback functions when changes occur, thereby allowing applications to maintain up-to-date states and perform actions dynamically based on the nature of the data change observed.

## Connecting to Atlas/Capella App Services
Each platform has a set of documentation that explains on how to configure App Services.

**MongoDb Atlas**
[App Services Connection Documentation](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/app-services/connect-to-app-services-backend/)
[Managing Users](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/users/authenticate-users/)

**Couchbase Capella**
[App Services Connection Documentation](https://docs.couchbase.com/cloud/app-services/deployment/creating-an-app-service.html)
[Managing Users](https://docs.couchbase.com/cloud/app-services/user-management/create-user.html)

Connecting to Capella App Services involves configuration of a replicator and then starting replication.  An example of a simple configuration can be found below:

```kotlin
// initialize the replicator configuration
val repl = Replicator(
    ReplicatorConfigurationFactory.newConfig(
        target = URLEndpoint(URI("ws://localhost:4984/mydatabase")),
        collections = mapOf(collections to null),
        //  other config params as required . .
        heartbeat = 150,
        maxAttempts = 20,
        maxAttemptWaitTime = 600
    )
)
repl.start()
thisReplicator = repl
```
#### More Information
[Couchbase Lite - Replicator Configuration](https://docs.couchbase.com/couchbase-lite/current/csharp/replication.html#lbl-cfg-repl)

## Syncing Realm vs Couchbase Lite Sync with Capella App Services
From a high level both platforms can sync information from the device to the cloud via there respective App Services.  There are differences between Atlas App Services security model which uses subscriptions and Couchbase Capella's security model, which is based on channels.  Before syncing data to the cloud, developers should review the documentation below.

Couchbase Capella App Services is a service that automates and manages a cluster of Sync Gateway servers in the cloud that are  used to sync information between App Services and a Couchbase Capella Bucket.  Some documentation will need to be reviewed from the Sync Gateway documentation area of the docs while other information can be found in the Couchbase Capella App Services documentation. It's recommended that developers and architects understand Channels, Users, Roles, the Import Process, and the Sync Function that is used to set security on documents before attempting to configure Couchbase Capella App Services.

- [Sync Gateway - Access Control Concepts](https://docs.couchbase.com/sync-gateway/current/access-control-concepts.html)
- [Sync Gateway - Channels](https://docs.couchbase.com/sync-gateway/current/channels.html)
- [Sync Gateway - Import Process](https://docs.couchbase.com/sync-gateway/current/import-processing.html)
- [Sync Gateway - Sync Function](https://docs.couchbase.com/sync-gateway/current/sync-function.html)
- [Sync Gateway - Delta Sync](https://docs.couchbase.com/sync-gateway/current/delta-sync.html)
-------
- [App Services - Create App Endpoint](https://docs.couchbase.com/cloud/app-services/deployment/creating-an-app-endpoint.html)
- [App Services - Adding Security with Channels](https://docs.couchbase.com/cloud/app-services/channels/channels.html)
- [App Services - Configuring Access Control and Data Validation](https://docs.couchbase.com/cloud/app-services/deployment/access-control-data-validation.html)
- [App Services - Creating Roles](https://docs.couchbase.com/cloud/app-services/user-management/create-app-role.html)
- [App Services - Creating Users](https://docs.couchbase.com/cloud/app-services/user-management/create-user.html)
- [App Services - Connecting](https://docs.couchbase.com/cloud/app-services/connect/connect-apps-to-endpoint.html)

For large scale deployments using an Authentication Provider that supports OpenID Connect is an easier way to manage users.
- [App Services - Authentication Providers](https://docs.couchbase.com/cloud/app-services/user-management/set-up-authentication-provider.html)

For applications that don't support OpenID Connect, REST APIs can be used to manage users, roles, and channels.  The REST API documentation can be found here:
[Capella App Services - REST API Documentation](https://docs.couchbase.com/cloud/app-services/references/rest_api_admin.html#tag/Database-Security)

## Troubleshooting

### Logging

The [Atlas documentation](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/logging/) covers how to set log levels and custom logs to allow developers to troubleshoot errors.

Couchbase Lite provides the ability to set logging levels and domains along with supporting Console, File, and Custom logging.

#### More Information
- [Couchbase Lite - Logging](https://docs.couchbase.com/couchbase-lite/current/csharp/troubleshooting-logs.html)

### Handling Data Conflicts
Document conflicts can occur if multiple changes are made to the same version of a document by multiple peers in a distributed system. For Couchbase Mobile, this can be a Couchbase Lite or App Services Endpoint.

#### More information
- [Couchbase Lite - Data Conflicts](https://docs.couchbase.com/couchbase-lite/current/csharp/conflict.html)

### Queries
The Couchbase Lite query API provides the Query Explain function that can be used to troubleshoot queries.

#### More Information
- [Couchbase Lite - Query Explain](https://docs.couchbase.com/couchbase-lite/current/csharp/troubleshooting-queries.html)

## Known Limitations
There are some APIs and features in Realm that do not exist today in the same way in the Couchbase Lite SDK.  Some guidance is provided below.

### IAsymmetricObject
[IAsymmetricObject](https://www.mongodb.com/docs/realm-sdks/dotnet/latest/reference/Realms.IAsymmetricObject.html) are unique to Realm and Atlas Device sync as a way to control insert-only documents.

In contrast to Asymmetric Object Types, all documents inserted into a Couchbase Lite database are just a JSON document.  When you retrieve a document from the Couchbase Lite Database using the Collection API and provide a documentId, it’s a Document object (immutable) until you convert it to a MutableDocument which would then allow you to make changes to it.

Couchbase Lite does provide a document purge API which in some use cases could be used to provide similar functionality to IAsymmetricObject interface depending on the use case and business rules required.


### In-Memory Realm
In Realm, you can open a realm that is stored entirely [in memory](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/open-and-close-a-realm/#open-an-in-memory-realm).  When this option is used, no realm file is created.

The Couchbase Lite SDK does not directly offer a feature to keep documents solely in memory; instead, all documents must first be persisted to the database before replication can occur. For scenarios where a developer requires data to reside only in memory—without any disk storage—the optimal approach would be to utilize the App Services/Sync Gateway REST API. This API enables the retrieval of documents directly into memory, bypassing the need for disk-based storage while still allowing access to up-to-date data.

### Add Initial Data to Realm
Realm provides an [API](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/realm-database/open-and-close-a-realm/#add-initial-data-to-realm) that facilitates the insertion of initial data into a Realm database at the time of its first opening.

While there is no direct equivalent in Couchbase Lite, a similar outcome can be achieved through the use of a pre-built database. This approach enables the application to be loaded with data upfront, bypassing the need to sync data from App Services during the initial application launch. This could significantly reduce the wait time for users upon their first installation and startup of the application.

>**NOTE**
>
>It is trivial to provide initial data, over the network, in the first sync of the Database collection(s) when the app launches. The only reason to provide an initial pre-built database is to save on network traffic, once.
>

#### More Information
- [Couchbase Lite Pre-build Database](https://docs.couchbase.com/couchbase-lite/current/csharp/prebuilt-database.html)

### Geospatial Types
In the Atlas SDK, Geospatial data, or "geodata", specifies points and geometric objects on the Earth's surface.

Couchbase Lite has no direct feature or support for Geospatial data types.  A developer would need to write custom code to handle any geospatial calculations or look into 3rd party packages.

### User Management
In the Realm SDK, it provides APIs in the SDK to [register new users](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/users/authenticate-users/).  Today the Couchbase Lite SDK provides no direct APIs to register users.  The Couchbase Capella Admin REST API does allow [creation of users](https://docs.couchbase.com/cloud/app-services/references/rest_api_admin.html#tag/Database-Security/operation/put_db-_user-name), however, you must [enable and configure](https://docs.couchbase.com/cloud/app-services/deployment/accessing-admin-apis.html) the Admin REST API before you can use it and setup security for it.

For large scale deployments, it is highly recommended to integrate with an [OpenID (OIDC) Authentication Provider](https://docs.couchbase.com/cloud/app-services/user-management/set-up-authentication-provider.html#openid-connect-oidc).

### User Authentication
In the Realm SDK, it provides an SDK to [authenticate users](https://www.mongodb.com/docs/atlas/device-sdks/sdk/kotlin/users/authenticate-users/#log-in-a-user).  The API returns a User object that gives you information about the user.

In Couchbase Mobile, there are different types of [client authentication](https://docs.couchbase.com/couchbase-lite/current/csharp/replication.html#lbl-client-auth) supported for the Replicator, but it doesn't return a user object, and you would have to check the [status of replication](https://docs.couchbase.com/couchbase-lite/current/csharp/replication.html#lbl-repl-status), specifically check the [Error](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-net/api/Couchbase.Lite.Sync.ReplicatorStatus.html#Couchbase_Lite_Sync_ReplicatorStatus_Error) property to see if you have issues with authentication.

### Migration of Data
There are several tools that can be used to migrate data from MongoDb Atlas to Couchbase Capella, but a developer needs to understand there data to understand any data transformations or other processes needed when exporting the data from MongoDb Atlas to Couchbase Capella.

#### JetBrains IDE Plugin
The [JetBrains IDE Plugin](https://youtu.be/xCKhzo2jSv4?si=wHyjPra7F0XWccK2&t=270) for Couchbase has a data import feature to allow developers to migrate data from MongoDb Atlas to Couchbase Capella a one-time job.  This is designed for developers to quickly move data for setting up a development environment.

#### cbimport (Couchbase Import)
[cbimport](https://docs.couchbase.com/server/current/tools/cbimport-json.html) is a command line tool that can be used to import information into Couchbase Capella from text files.

#### Import Data from Capella UI
Couchbase Capella's [Data Tools UI](https://docs.couchbase.com/cloud/clusters/data-service/import-data-documents.html) provides the ability to directly import data from text files.

#### Import Data using Couchbase SDK
The [Couchbase SDK's](https://docs.couchbase.com/home/sdk.html) provide APIs for [importing data](https://docs.couchbase.com/cloud/guides/import.html) directly into Capella, allowing developers to customize and transform data during the import process.