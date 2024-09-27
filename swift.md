# Atlas Device SDK to Couchbase Lite Comparison Guide
The Atlas Device SDK can be thought of as an on-device ORM that creates an “object store” that is then synced to MongoDb Atlas.  The following document describes the features of Atlas and how they compare to Couchbase Mobile.

>**NOTE**
>
>This document is designed for developers using Swift for building iOS and Mac applications.
>
>
## Configuring and Opening Realm vs Couchbase Lite Database

### Realm File

Realm stores a binary-encoded version of every object and type in a [single .realm]https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/realm-files/) file.  When you open a .realm file it will create one if it doesn’t already exist.  The file is located in a path the developer specifies.

### Couchbase Lite Database

Couchbase Lite stores all JSON documents in a single database that is a filename with a cblite2 extension (technically, if you were to look at the file on disk you would see it's a directory, but you should never mess with or attempt to open the files in the directory).

When you open a database a new cblite2 file will be created if it doesn’t already exist on the disk.  The developer specifies the path to the database, or a default path is used by the SDK when no path is provided.

#### Tools for viewing Database contents
Couchbase Lite has a few ways to view the contents of a database file outside your application.

- [JetBrains IDE Plugin](https://plugins.jetbrains.com/plugin/22131-couchbase)
- [Visual Studio Code Plugin](https://marketplace.visualstudio.com/items?itemName=Couchbase.vscode-cblite)
- [cblite Command Line Tool](https://github.com/couchbaselabs/couchbase-mobile-tools/blob/master/Documentation.md)

### Opening a Realm
To open a realm, you must provide a [RealmConfiguration](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/realm-files/configure-and-open-a-realm/) object that defines the details of the realm.  This configuration is passed to the *Realm.GetInstance* function.

```swift
// Open the default realm
let defaultRealm = try! Realm()
// Open the realm with a specific file URL, for example a username
let username = "GordonCole"
var config = Realm.Configuration.defaultConfiguration
config.fileURL!.deleteLastPathComponent()
config.fileURL!.appendPathComponent(username)
config.fileURL!.appendPathExtension("realm")
let realm = try! Realm(configuration: config)
```

### Opening a Couchbase Lite Database
To open a Couchbase Lite database or create a new database, you can provide a DatabaseConfiguration object that defines the directory and an encryption key used to secure the database.  This configuration is passed on to the Database API to open the database.

```swift
do {
    // Specify the path and encryption key for the database configuration
    let config = DatabaseConfiguration()
    config.directory = "mypath" // Make sure to define as the directory path you can write to
    config.encryptionKey = EncryptionKey.password("password")

    // Create a new, or open an existing database with the specified configuration
    let database = try Database(name: "mydb", config: config)
    
    //do app logic
} catch {
    print("Error managing Couchbase Lite database: \(error)")
}
```

#### More Information
- [Couchbase Lite Database Encryption](https://docs.couchbase.com/couchbase-lite/current/swift/database.html#database-encryption)

### Custom Realm Configuration
In Realm, a custom configuration can be based to trigger additional functionality:
- Compacting a realm to reduce its file size
- Specifying a schema version or migration block for making schema changes
- Flagging a realm to be deleted if a migration is required

As stated before, Couchbase Lite doesn’t need schema management or migration plans, so there is no equivalent to this.  Couchbase Lite databases have a Database Maintenance API which can be used to run tasks like compacting the database or to rebuild indexes.

#### More Information
- [Couchbase Lite Database Maintenance API](https://docs.couchbase.com/couchbase-lite/current/swift/database.html#lbl-db-util)

### Bundle a Realm /Couchbase Lite Pre-built Database/Database Copy
The [Bundle Realm](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/realm-files/bundle-a-realm/_) method allows developers to include an existing realm during the first time setup of initial data.  There are some rules to this which are well documented.

In Couchbase Lite there the option to use a [Pre-built](https://docs.couchbase.com/couchbase-lite/current/swift/prebuilt-database.html) database, which supports the same feature set.

> **NOTE**
> You should NEVER copy a Couchbase Lite database outside using the Database.Copy API as it will invalidate other apps checkpoints.
>

#### More Information
- [Couchbase Lite - Database Copy](https://docs.couchbase.com/couchbase-lite/current/csharp/prebuilt-database.html#deploy-db)

### Summary

Both the Realm API and the Couchbase Lite Database API exhibit comparable functionalities when managing databases, particularly in terms of opening, closing, and copying database instances.

#### More Information
- [Couchbase Lite - Database API](https://docs.couchbase.com/couchbase-lite/current/swift/database.html)

## Model Data/Object Store

Realm relies on the developer to define [Realm objects](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/object-models/#define-a-new-object-type) or objects that are uniquely named that they can interact with Atlas Device and Device synchronization.

### Realm Object Types

All Realm objects inherit from the interfaces listed below and must be declared as **partial** classes.

| Name             | Documentation| 
|------------------|---|
| Object           | [Docs](https://www.mongodb.com/docs/realm-sdks/swift/latest/Extensions/Object.html)                     |
| EmbeddedObject   | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/relationships/#define-an-embedded-object-property)    |
| AsymmetricObject | [Docs](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/object-models/#define-an-asymmetric-object) |

Couchbase Lite is flexible in that all data is stored in the database as a JSON document using the Document API.  You can use SQL++ Queries to get information from the database OR you can query the database using a documentId which can be similarly thought of as Realms “ObjectId”.

When you embed objects in Realm you use the [EmbeddedObject](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/relationships/#define-an-embedded-object-property) class, whereas in Couchbase Lite you can  think of embedded data as an embedded JSON document or dictionary.

### Collection Types
Realm has several types that represent groups of objects, which they call collections.  A collection type conforms to the [RealmCollection](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/supported-types/#collection-types) protocol.  

Couchbase Lite, in contrast, does not have direct analogs to these collection types but instead supports flexible data structuring through JSON. This allows for storing arrays and dictionaries directly within JSON documents.

Couchbase Lite supports Scopes and Collections for storing JSON documents.  A scope is a namespace that may contain up to 1,000 collections.  Collections are a group of JSON documents, whose contents can be different (schema doesn’t have to be the same).  Storing documents with different schemas in the same Collection comes at no performance loss in Couchbase Lite and SQL++ has keywords to help you query for documents that are missing a property(field).

When querying data in Couchbase Lite with SQL++, it retrieves results in a ResultSet object, which can be iterated to extract individual Result objects.  Results are, essentially, JSON documents and can be treated as Dictionaries, wrapped in application objects, or serialized to JSON.

These results can then be directly cast to custom object types defined within the application, facilitating seamless integration with the app’s data architecture. Additionally, results can be converted into JSON strings using the Result.toJSON() function. This JSON data can subsequently be processed using any serialization library compatible with the platform in use, providing extensive flexibility for data manipulation and integration.

#### More Information
- [Couchbase Lite ResultSet API](https://docs.couchbase.com/couchbase-lite/current/swift/query-resultsets.html)
- [Couchbase Lite Scopes - Collections API](https://docs.couchbase.com/couchbase-lite/current/csharp/scopes-collections-manage.html)
### Summary

In using Realm and the Atlas Device SDK, developers must carefully plan their data model, particularly regarding the types of objects they inherit from, to effectively leverage the database’s capabilities. This upfront planning is crucial due to the structured nature of the data and the object-oriented approach of Realm.

Conversely, Couchbase Lite offers a flexible data handling approach. All information is stored as JSON documents in the database. This format allows developers to freely choose how they structure, write, and read their data. The flexibility of JSON documents enables a variety of patterns and structures to be implemented, adapting easily to the evolving needs of the application and reducing the need for extensive initial planning.

### Supported Data Types/Data Modeling
In Realm each platform the SDK supports as a given set of supported Field Types:
- [Field Types](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/supported-types/#property-cheat-sheet)

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
- [Couchbase Lite Data Types](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#data-types)

### ObjectId vs DocumentId

Realm objects can use the [ObjectId or UUID](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/supported-types/#unique-identifiers) for uniquely identifying an object.  The ObjectId is a 12-byte unique value, indexable, and used as a primary key.

In Couchbase Lite a [documentId](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#document-structure) can be any string value as long as it’s not null.  A document created without an ID will always have a randomly generated a GUID to be used for the documentId.

>**NOTE**
>
>Developers convert data from MongoDb Atlas to Couchbase Capella will need to take this into account when converting the _id field in a given Realm object to a DocumentId.
>

### Property Annotations
In Realm, property annotations play a critical role in defining the schema of objects. These annotations specify how objects should be stored in the database, including details on which fields are indexed, thereby enhancing query performance and ensuring data integrity.

On the other hand, Couchbase Lite operates differently due to its use of JSON documents for storage.  Couchbase Lite JSON documents are schemaless, and manages indexing through a separate Index API. This approach allows developers to create and manage indexes independently of the document storage API, providing flexibility in how data is indexed and retrieved without altering the document structure itself.

#### More Information
- [Couchbase Lite Populating a Document](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#populate-a-document)
- [Couchbase Lite Indexing API](https://docs.couchbase.com/couchbase-lite/current/swift/indexing.html)


### Backlinks and Relationships
In Realm, a [backlink](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/relationships/#define-an-inverse-relationship-property) defines an inverse, to-many relationship between objects, where backlinks cannot be null. Unlike MongoDB, which explicitly avoids using bridge tables or explicit joins for defining relationships, Realm utilizes its proprietary mechanisms.

Couchbase Lite, on the other hand, offers two approaches for managing document relationships. Firstly, it allows for embedding documents directly within other documents, akin to the feature provided by the Atlas Device SDK via embedded dictionaries in a document. This method, while straightforward, may lead to the creation of large document sizes, which can be inefficient for certain operations. Alternatively, Couchbase Lite supports the use of document IDs to establish relationships between documents. This method involves storing a document ID within another document, effectively using it as a foreign key to facilitate joins.

To manage relationships between smaller documents or to ensure efficient querying, developers might prefer using document IDs. These IDs serve as references that link documents together, akin to foreign keys in relational databases. To resolve these relationships, developers can employ SQL++ JOIN operations, supported by Couchbase Lite’s query language. By indexing the relevant document ID fields, performance impacts during such queries are minimized, ensuring efficient data retrieval across linked documents.

#### More Information
- [Couchbase Lite SQL++ JOIN](https://docs.couchbase.com/couchbase-lite/current/swift/query-n1ql-mobile.html#lbl-join)

### Changes to Object Model (schema synchronization)
Realms [documentation](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/change-an-object-model/) has a detailed explanation of when and when you don’t have to worry about schema changes that is out of the scope of this document.

Couchbase Lite is a Documents based database, in which documents are stored in [JSON](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#lbl-json-data), so you never have to be concerned about schema changes.

### Summary
In both Realm and the Atlas Device SDK, considerable effort is dedicated to comprehensively understanding the data schema and meticulously defining the synchronization model for devices. This process is crucial for ensuring that the data structures are correctly aligned with the operational requirements and synchronization protocols.

Conversely, Couchbase Lite is a document-oriented database that leverages the inherent flexibility of JSON documents. This flexibility significantly reduces the time required for schema definition, allowing developers to dynamically accommodate various data types. Consequently, any constraints or data type regulations can be efficiently managed within the application’s business logic layer, rather than at the database level. This approach not only simplifies initial setup but also enhances adaptability to evolving data requirements.

## Create & Write Data in Realm vs Couchbase Lite

### Writing Data
In Realm, you write data in a transaction which is a block of code that can do multiple sub-transactions.  Either all the transactions succeed or they fail.   An example of this is below:
```swift
// Instantiate the class and set its values.
let dog = Dog()
dog.name = "Rex"
dog.age = 10
// Get the default realm. You only need to do this once per thread.
let realm = try! Realm()
// Open a thread-safe transaction.
try! realm.write {
    // Add the instance to the realm.
    realm.add(dog)
}
```
In this code an object is created, then copied to the Realm.

For individual objects, Couchbase Lite doesn’t take this approach.  Instead, Couchbase Lite uses its MutableDocument API to allow developers to create a document and then save it into a Collection.  Within Multiple Documents, you have a few different ways you can create one.  The standard approach is to write some code that takes your data and puts it into the Mutable Document using Key Value pairs:
```swift
let mutableDocument = MutableDocument()
mutableDocument.setString("name", "Do this thing")
mutableDocument.setInt("age", 10),
try! collection.save(document: mutableDocument)
```

Another way is to use serialization libraries to serialize down the object to JSON and then create the MutableDocument based on the [JSON](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-net/api/Couchbase.Lite.MutableDocument.html#Couchbase_Lite_MutableDocument__ctor_System_String_System_String_) values.

```swift
do {
  let dog = Dog()
  dog.name = "Rex"
  dog.age = 10
  
  let encoder = JSONEncoder()
  let jsonData = try encoder.encode(dog)
  
  if let json = String(data: jsonData, encoding: .utf8) {
    let mutableDocument = MutableDocument(id: "doc-1", json: json)
    try collection.save(document: mutableDocument);
  }
} catch {
    print("Error serializing object to JSON: \(error)")
}
```
> **NOTE**
> You will always take a performance and memory hit depending on device constraints  when serializing and deserializing objects to and from JSON strings.  If performance is a concern, it's best to use the Key Value pair approach when creating documents in Couchbase Lite.
>

Couchbase Lite does provide a Batch API to write multiple documents at once.  At the local level this operation is still transactional: no other Database instances, including ones managed by the replicator can make changes during the execution of the block, and other instances will not see partial changes.

An example of a inBatch transaction in Couchbase Lite:

```swift
do {
    try database.inBatch {
        for i in 0...10 {
            let doc = MutableDocument()
            doc.setValue("user", forKey: "type")
            doc.setValue("user \(i)", forKey: "name")
            doc.setBoolean(false, forKey: "admin")
            try collection.save(document: doc)
            print("saved user document \(doc.string(forKey: "name")!)")
        }
    }
} catch let error {
    print(error.localizedDescription)
};
```

#### More Information
Couchbase Lite Document Batch API Documentation:
- [Couchbase Lite - Batch Operations](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#batch-operations)

### Create an Embedded Object

In Couchbase Lite, embedded objects are just embedded dictionaries which you can use Key Value pairs or the JSON serialization method.  An example of Key Value:
```swift
let mdContact = MutableDictionaryObject()
mdContact.setString("Mr. Frog", forKey: "name")

let mdCountry = MutableDictionaryObject()
mdCountry.setString("United States", forKey: "name")

let mdAddress = MutableDictionaryObject()
mdAddress.setDictionary(mdContact, forKey: "contact")  
mdAddress.setString("123 Pond St", forKey: "street")
mdAddress.setDictionary(mdCountry, forKey: "country")

let mutableDocument = MutableDocument()
mutableDocument.setString("Kermit", forKey: "name")
mutableDocument.setDictionary(mdAddress, forKey: "address")

do {
    try collection.save(document: mutableDocument)
} catch {
    print("Error saving document: \(error)")
}
```

An example of using the .NET Serialization to serialize an object to JSON string and then create a MutableDocument based on the JSON string value:
```csharp
let propertyOwnerContact = Contact("Mr. Frog")
let country = Country(name: "United States")
let address = Address(owner: propertyOwnerContact, street: "123 Pond St", country: country)
let contact = Contact("Kermit", address: address)
do {
    let encoder = JSONEncoder()
    let jsonData = try encoder.encode(contact)
    if let jsonString = String(data: jsonData, encoding: .utf8) {
        let mutableDocument = MutableDocument(id: "doc-1", data: jsonString)
        
        try collection.save(document: mutableDocument)
        print("Document saved successfully.")
    }
} catch {
    print("Error: \(error)")
}
```

Finally for short-lived data, a document could be created with an expiration time and then deleted from the database after the expiration time has passed.  This purge is NOT replicated to App Services.

```csharp
// Purge the document one day from now
let ttl = Calendar.current.date(byAdding: .day, value: 1, to: Date())
try collection.setDocumentExpiration(id: "doc-1", expiration: ttl)
```

#### More Information
- [Couchbase Lite - Document Expiration](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#document-expiration)

### Summary

Couchbase Lite utilizes the Documents and Blobs API to facilitate the storage of information within a Couchbase Lite database. Data storage can be efficiently managed using the MutableDocument class, which offers versatile data entry methods. Developers can set values either through key-value pairs, providing a clear and structured approach to data handling, or directly via a JSON string, which allows for rapid integration of complex data structures. This flexibility ensures that Couchbase Lite can accommodate a wide range of application needs, from simple data storage to complex data manipulation tasks.

## Read Realm Objects in Realm vs Couchbase Lite

### Read Operations

A read operation in Atlas Device SDK consists of querying database objects using either Realm Swift Query API or Swift NSPredicate support.

An example of a query:
```swift
let realm = try! Realm()

// Access all dogs in the realm
let dogs = realm.objects(Dog.self)

// Query by age
let puppies = dogs.where {
    $0.age < 2
}
```

Couchbase Lite can use the SQL++ query language to achieve the same results.  An example of using the Couchbase Lite Query API:

```swift
do {
  let query = database.createQuery("SELECT * AS dog FROM animals.dogs WHERE dog.age < 2");
  let results:ResultSet = try query.execute();
  
  // Loop through all the results
  for result in results {
        // Access the document and do something with it
        if let dog = result.dictionary(forKey: "dog") {
            // Example: Print the dog's name
            if let name = dog.string(forKey: "name") {
                print("Dog's name: \(name)")
            }
        }
    }
} catch {
    print("Query failed with error: \(error)")
}
```

### Find a Specific Object by Primary Key
In the Atlas Device SDK, you use the Find method to return an Object based on the primary key.

```swift
let realm = try! Realm()
let specificPerson = realm.object(ofType: Person.self, forPrimaryKey: 12345)
```

In the Couchbase Lite SDK for .NET, you can use the Collection API to retrieve a document by its documentId.

```swift
do {
    // Assuming 'collection' is a correctly instantiated and accessible CBLCollection
    guard let document = try collection.document(id: "12345") else {
        print("Document not found")
        return
    }

    let jsonString = try document.toJSON()
    let decoder = JSONDecoder()
    if let jsonData = jsonString.data(using: .utf8) {
        do {
            let specificPerson = try decoder.decode(Person.self, from: jsonData)
            print("Person name: \(specificPerson.name), Age: \(specificPerson.age)")
        } catch {
            print("Failed to decode JSON: \(error)")
        }
    }
} catch {
    print("Error: \(error)")
}
```

### Query All Objects of a Type
In the Atlas SDK, the **realm.objects** method can be used to return all objects of a [specific type](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/crud/read/#query-all-objects-of-a-given-type).

In Couchbase Lite the recommended approach would be to store documents by type of information into unique collections, thus you can just query a collection to get all the documents of that type.  For example, you could store all Frogs in a scope called animals and a collection called frogs, thus getting all frogs back from the database is a SQL query.

```sql
SELECT * FROM animals.dogs as Dog 
```

In Couchbase Lite, you can use [Type Check Functions](https://docs.couchbase.com/couchbase-lite/current/swift/query-n1ql-mobile.html#lbl-func-typecheck) which can check a type and returns true if it matches and false if it doesn't match.

```sql
SELECT * FROM animals.dogs as Dog WHERE ISNUMBER(dog.age)
```

### Filter by Full-Text Search (FTS)
In Swift, the Atlas Device SDK currently doesn't support Full Text Search according to this [GitHub Issue](https://github.com/realm/realm-swift/issues/3650).

In Couchbase Lite, a Full-Text Search is supported when an index is created on a collection for the properties you would like to index.

```swift
do {
    guard let collection = database.collection(withName: "collectionName", scopeName: "scopeName") else {
        print("Collection not found")
        return
    }
    let indexConfig = FullTextIndexConfiguration(["biography"])
    try collection.createIndex(withName: "idxPersonBiography", config: indexConfig)
    print("Index created successfully")
} catch {
    print("Error creating index: \(error)")
}
    
```
Once the index is created you can use the SQL++ keyword MATCH OR RANK functions to query the collection using FTS

```sql
SELECT * FROM scopeName.collectionName as person 
WHERE MATCH(person.biography, 'scientist Nobel')
```

#### More Information
- [Couchbase Lite - Full-Text Search API](https://docs.couchbase.com/couchbase-lite/current/swift/fts.html)

### Sort and Limit Results
In the Atlas SDK, you can sort and limit results using [Results.sorted(by:)](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/crud/read/#sort-query-results).

In Couchbase Lite, you can use the SQL++ ORDER BY and LIMIT keywords to sort and limit results.

```sql
SELECT DISTINCT(*) FROM animals.frogs as frog WHERE frog.owner = 'Jim Henson' ORDER BY age DESC LIMIT 2
```

### Aggregate Results

In Couchbase Lite, SQL++ provides a plethora of aggregate operators and functions.
##### More Information
- [Couchbase Lite - SQL++ for Mobile Developers](https://docs.couchbase.com/couchbase-lite/current/swift/query-n1ql-mobile.html)

### Summary

Couchbase Lite utilizes SQL++, an extension of the industry-standard SQL, to query information from its database, ensuring compatibility and ease of use with familiar syntax and robust capabilities. In contrast, the Atlas Device SDK relies on Realm Swift Query API or Swift NSPredicate to query a Realm object store.

## Updating Realm Objects in Realm vs Couchbase Lite

### Updating Realm Objects

Realm supports [update](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/crud/update/#update-an-object) and [upsert](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/crud/update/#upsert-an-object) operations on Realm objects and embedded objects. An upsert operation either inserts a new instance of an object or updates an existing object that meets certain criteria.  The syntax to update an object in a realm is the same for a local or a synced realm. However, there are additional considerations that determine whether the write operation in a synced realm is successful.  Realm requires operations for updating are done in the realm.write or realm.writeBlocking transaction block.

In Couchbase Lite updating a document is almost the same as workflow as creating a document.  You can use the MutableDocument API to update a document in a collection.  To update a document, you first retrieve the document from the collection, make the changes to the document, and then save the document back to the collection.

```swift
if let collection = database.collection(name: "frogs", scope: "animals") {
    // Get the document and convert it to a mutable form
    if let document = collection.document(id: "frogId::kermit")?.toMutable() {
        // Modify the document
        document.setString("Kermit the Frog", forKey: "name")

        // Save the document back to the collection
        do {
            try collection.save(document: document)
            print("Document saved successfully.")
        } catch {
            print("Error saving document: \(error)")
        }
    } else {
        // Handle the case where the document does not exist
        print("Document not found.")
    }
} else {
    print("Collection not found.")
}
```

### Updating many documents

As previously discussed, Couchbase Lite offers a robust [batch API](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#batch-operations) that enables the insertion or updating of multiple documents within a single transactional operation. This capability ensures atomicity at the local database level, meaning that no other Database instances, including those managed by the replicator, can modify the database during the execution of the transactional block. Additionally, other instances are prevented from observing any intermediate states of the transaction.

However, it is important to recognize that Couchbase Mobile operates as part of a distributed system. Consequently, despite the transactional integrity maintained at the local database level, the asynchronous nature of the replication process does not guarantee simultaneous visibility of changes across Sync Gateway or other devices.

## Deleting Realm Objects in Realm vs Couchbase Lite

### Deleting Realm Objects

In Realm and the Atlas Device SDK you can choose to [delete a single object](https://www.mongodb.com/docs/atlas/device-sdks/sdk/dotnet/crud/delete/#delete-an-object), multiple objects, or all objects from the realm. After you delete an object, you can no longer access or modify it.

```swift
// Previously, we've added a dog object to the realm.
let dog = Dog(value: ["name": "Max", "age": 5])
let realm = try! Realm()
try! realm.write {
    realm.add(dog)
}
// Delete the instance from the realm.
try! realm.write {
    realm.delete(dog)
}
```

In Couchbase Lite, the Collection's API provides a few different functions based on the developers need.  The first API is the Collection API delete document function.

```swift
if let collection = database.collection(name: "dogs", scope: "animals") {
    // Get the document 
    if let document = collection.document(id: "dogId::Max") {
        do {
            try collection.delete(document: document)
            print("Document deleted successfully.")
        } catch {
            print("Error deleting document: \(error)")
        }
    } else {
        // Handle the case where the document does not exist
        print("Document not found.")
    }
} else {
    print("Collection not found.")
}
```

Optionally, when saving or deleting a document to the database you can pass in a concurrency control token. This is done by passing in the concurrency control token to the save or delete function.

```csharp
if let collection = database.collection(name: "dogs", scope: "animals") {
    // Get the document 
    if let document = collection.document(id: "dogId::Max") {
        do {
            let didDelete = try collection.delete(document: document, concurrencyControl: .failOnConflict)
            if didDelete { 
                print("Document deleted successfully.")
            }
        } catch {
            print("Error deleting document: \(error)")
        }
    } else {
        // Handle the case where the document does not exist
        print("Document not found.")
    }
} else {
    print("Collection not found.")
}
```
Note that when specifying the [.failOnConflict](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-swift/Classes/Collection.html#/s:18CouchbaseLiteSwift10CollectionC6delete8document18concurrencyControlSbAA8DocumentC_AA011ConcurrencyH0OtKF:~:text=delete(document%3AconcurrencyControl%3A)) concurrency control, it will return true if the delete call succeeded and false if there was a conflict.

When calling the delete function, that deletion will propagate to the server and to all the other devices that sync with that database.  Calling delete is actually identical to saving a document with a body that is empty except for a single property "_deleted" set to true.

### Purging a Document

Couchbase Lite also provides another API to remove a document from the database.  This API is called [purge](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-swift/Classes/Collection.html#/s:18CouchbaseLiteSwift10CollectionC5purge8documentyAA8DocumentC_tKF).  Purging a document is more extreme than calling the delete API: it actually deletes the document and all its metadata. There’s nothing left to indicate the document ever existed.  For this reason, though, calling purge doesn’t replicate.

```csharp
// Assuming 'database' is already an instance of 'Database'
if let collection = database.collection(name: "frogs", scope: "animals") {
    // Get the document
    if let document = collection.document(id: "frogId::kermit") {
        do {
            // Purge the document
            try collection.purge(document: document)
            print("Document successfully purged.")
        } catch {
            // Handle errors during purging, such as document not found
            print("Error purging document: \(error.localizedDescription)")
        }
    } else {
        print("Document not found.")
    }
} else {
    print("Collection not found.")
}
```
## Manging Realm Files vs Couchbase Lite Database

### Reduce File Size

Starting in version 1.6.0 of the Atlas Device SDK, realm automatically compacts Realm files in the background.  Automatic compaction begins when the size of unused space in the file is more than twice the size of user data in the file. Automatic compaction only takes place when the file is not being accessed.

Couchbase Lite also provides the Database Maintenance API that can provide several functions, including a compact function that shrinks the database file by removing any empty pages, and deletes blobs that are no longer referenced by any documents.

```swift
Database.performMaintenance(type: .compact); 
```

#### More Information
- [Couchbase Lite - Database Maintenance](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-swift/Classes/Collection.html#/s:18CouchbaseLiteSwift10CollectionC5purge8documentyAA8DocumentC_tKF)

### Delete a Realm File

To safely delete a realm file while the app is running, you can use the [Realm.deleteFiles](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/realm-files/delete-a-realm/) method. The following code demonstrates this:
```swift
autoreleasepool {
    // all Realm usage here -- explicitly guarantee
    // that all realm objects are deallocated
    // before deleting the files
}
do {
    let app = App(id: APPID)
    let user = try await app.login(credentials: Credentials.anonymous)
    var configuration = user.flexibleSyncConfiguration()
    _ = try Realm.deleteFiles(for: configuration)
} catch {
    // handle error
}
```

In Couchbase Lite, there is a function provided to delete a database.  Deleting a database will stop all replicators, live queries and all listeners attached to it. Although attempting to close a closed database is not an error, attempting to delete a closed database is.
```swift
database.delete();
```

#### More Information
- [Couchbase Lite - Delete Database](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-swift/Classes/Database.html#/s:18CouchbaseLiteSwift8DatabaseC6deleteyyKF)

##  React to Changes Realm vs Couchbase Lite

### Change Events

In Realm, you can subscribe to changes on the following [events](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/react-to-changes/):

- [Realm notifications](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/react-to-changes/#register-a-realm-change-listener)
- [Realm collections](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/react-to-changes/#register-a-collection-change-listener)
- [Realm object](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/react-to-changes/#register-an-object-change-listener)
- [Key Path Change](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/react-to-changes/#register-a-key-path-change-listener)

Couchbase Lite provides change listeners for the following events:
- [Query Listener (Live Query)](https://docs.couchbase.com/couchbase-lite/current/swift/query-live.html)
- [Document Listener](https://docs.couchbase.com/couchbase-lite/current/swift/document.html#document-change-events)
- [Collection Listener](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-swift/Classes/Collection.html#/s:18CouchbaseLiteSwift10CollectionC17addChangeListener8listenerAA0G5TokenCyAA0dF0Vc_tF)
- [Replication Status Listener](https://docs.couchbase.com/couchbase-lite/current/swift/replication.html#lbl-repl-status)
- [Replication Document Listener](https://docs.couchbase.com/couchbase-lite/current/swift/replication.html#lbl-repl-evnts)

### Summary

Couchbase Lite offers a suite of change listener APIs that enable developers to implement event-driven architectures by responding programmatically to modifications in collections, documents, queries, and the replication process. These event listeners facilitate real-time data handling by triggering callback functions when changes occur, thereby allowing applications to maintain up-to-date states and perform actions dynamically based on the nature of the data change observed.

## Connecting to Atlas/Capella App Services
Each platform has a set of documentation that explains on how to configure App Services.

**MongoDb Atlas**
- [App Services Connection Documentation](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/app-services/connect-to-app-services-backend/)
- [Managing Users](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/users/create-and-delete-users/)

**Couchbase Capella**
- [App Services Connection Documentation](https://docs.couchbase.com/cloud/app-services/deployment/creating-an-app-service.html)
- [Managing Users](https://docs.couchbase.com/cloud/app-services/user-management/create-user.html)

Connecting to Capella App Services involves configuration of a replicator and then starting replication.  An example of a simple configuration can be found below:

```swift
guard let targetURL = URL(string: "your app endpoint url here") else {
    fatalError("Invalid URL")
}
let targetEndpoint = URLEndpoint(url: targetURL)
var config = ReplicatorConfiguration(target: targetEndpoint) 
config.addCollection(collection)

config.replicatorType = .pushAndPull

// Configure Sync Mode
config.continuous = true

// Configure Client Security 
//  Set Authentication Mode
config.authenticator = BasicAuthenticator(username: "username",
                                          password: "secret")
                                          
// Apply configuration settings to the replicator
self.replicator = Replicator.init(config: config)
self.replicator.start()
```
#### More Information
[Couchbase Lite - Replicator Configuration](https://docs.couchbase.com/couchbase-lite/current/swift/replication.html#lbl-cfg-repl)

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

The [Atlas documentation](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/logging/) covers how to set log levels and custom logs to allow developers to troubleshoot errors.

Couchbase Lite provides the ability to set logging levels and domains along with supporting Console, File, and Custom logging.

#### More Information
- [Couchbase Lite - Logging](https://docs.couchbase.com/couchbase-lite/current/swift/troubleshooting-logs.html)

### Handling Data Conflicts
Document conflicts can occur if multiple changes are made to the same version of a document by multiple peers in a distributed system. For Couchbase Mobile, this can be a Couchbase Lite or App Services Endpoint.

#### More information
- [Couchbase Lite - Data Conflicts](https://docs.couchbase.com/couchbase-lite/current/swift/conflict.html)

### Queries
The Couchbase Lite query API provides the Query Explain function that can be used to troubleshoot queries.

#### More Information
- [Couchbase Lite - Query Explain](https://docs.couchbase.com/couchbase-lite/current/swift/troubleshooting-queries.html)

## Known Limitations
There are some APIs and features in Realm that do not exist today in the same way in the Couchbase Lite SDK.  Some guidance is provided below.

### tvOS/watchOS/visionOS support
Couchbase Lite Swift framework doesn't support tvOS, watchOS, or visionOS.  More information of platforms supported can be found [here](https://docs.couchbase.com/couchbase-lite/current/swift/supported-os.html).

### Swift Concurrency Features 
The Realm SDK supports [Swift Concurrency](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/realm-files/configure-and-open-a-realm/#open-a-realm-with-swift-concurrency-features) allowing developer to isolate realm interactions or specify an actor when opening a realm file.

The Couchbase Lite SDK doesn't provide direct integration with Swift Concurrency, however, it doesn't stop the developer from writing a wrapper around database and collection operations and assigning that to a specific actor or using features like async/await.

### SwiftUI Framework
Realm Atlas has a very specific framework for [SwiftUI](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/swiftui/) developers, mimicking many features of ORMs like SwiftData.  

Couchbase Lite doesn't provide a framework that provides similar functionality.

### Xcode Playground
Realm Atlas supports [XCode Playground](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/xcode-playgrounds/).

Couchbase Lite doesn't provide a framework that provides similar functionality.

### AsymmetricObject
[AsymmetricObject](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/model-data/object-models/#define-an-asymmetric-object) are unique to Realm and Atlas Device sync and only sync unidirectional. The use case from the docs point of view is sensor type information for IoT data.

Couchbase Lite does not offer a direct API for directly pushing data to Couchbase Capella without first writing it to disk. Instead, alternatives such as configuring a replicator in PUSH-only mode are available which would only send documents saved to the database to Capella App Services without pulling new documents down to the database.

```swift
guard let targetURL = URL(string: "your app endpoint url here") else {
    fatalError("Invalid URL")
}
let targetEndpoint = URLEndpoint(url: targetURL)
var config = ReplicatorConfiguration(target: targetEndpoint) 
config.replicatorType = .push
```

This could be used with the Document purge API to provide similar functionality depending on the business rules and use case.

#### More Information
- [Couchbase Lite - Replicator Sync Mode](https://docs.couchbase.com/couchbase-lite/current/swift/replication.html#lbl-cfg-sync)

### In-Memory Realm
In Realm, you can open a realm that is stored entirely [in memory](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/realm-files/configure-and-open-a-realm/#open-an-in-memory-realm).  When this option is used, no realm file is created.

The Couchbase Lite SDK does not directly offer a feature to keep documents solely in memory; instead, all documents must first be persisted to the database before replication can occur. For scenarios where a developer requires data to reside only in memory—without any disk storage—the optimal approach would be to utilize the App Services/Sync Gateway REST API. This API enables the retrieval of documents directly into memory, bypassing the need for disk-based storage while still allowing access to up-to-date data.

### Add Initial Data to Realm
Realm provides an API that facilitates the insertion of initial data into a Realm database at the time of its first opening.

While there is no direct equivalent in Couchbase Lite, a similar outcome can be achieved through the use of a pre-built database. This approach enables the application to be loaded with data upfront, bypassing the need to sync data from App Services during the initial application launch. This could significantly reduce the wait time for users upon their first installation and startup of the application.

>**NOTE**
>
>It is trivial to provide initial data, over the network, in the first sync of the Database collection(s) when the app launches. The only reason to provide an initial pre-built database is to save on network traffic, once.
>

#### More Information
- [Couchbase Lite Pre-build Database](https://docs.couchbase.com/couchbase-lite/current/swift/prebuilt-database.html)

### Geospatial Types
In the Atlas SDK, Geospatial data, or "geodata", specifies points and geometric objects on the Earth's surface.

Couchbase Lite has no direct feature or support for Geospatial data types.  A developer would need to write custom code to handle any geospatial calculations or look into 3rd party packages.

### User Management
In the Realm SDK, it provides APIs in the SDK to [register new users](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/users/create-and-delete-users/#create-a-user).  Today the Couchbase Lite SDK provides no direct APIs to register users.  The Couchbase Capella Admin REST API does allow [creation of users](https://docs.couchbase.com/cloud/app-services/references/rest_api_admin.html#tag/Database-Security/operation/put_db-_user-name), however, you must [enable and configure](https://docs.couchbase.com/cloud/app-services/deployment/accessing-admin-apis.html) the Admin REST API before you can use it and setup security for it.

For large scale deployments, it is highly recommended to integrate with an [OpenID (OIDC) Authentication Provider](https://docs.couchbase.com/cloud/app-services/user-management/set-up-authentication-provider.html#openid-connect-oidc).

### User Authentication
In the Realm SDK, it provides an SDK to [authenticate users](https://www.mongodb.com/docs/atlas/device-sdks/sdk/swift/users/authenticate-users/#log-in).  The API returns a User object that gives you information about the user.

In Couchbase Mobile, there are different types of [client authentication](https://docs.couchbase.com/couchbase-lite/current/swift/replication.html#lbl-client-auth) supported for the Replicator, but it doesn't return a user object, and you would have to check the [status of replication](https://docs.couchbase.com/couchbase-lite/current/swift/replication.html#lbl-repl-status), specifically check the [Error](https://docs.couchbase.com/mobile/3.2.0/couchbase-lite-swift/Classes/Replicator/Status.html#/s:18CouchbaseLiteSwift10ReplicatorC6StatusV5errors5Error_pSgvp) property to see if you have issues with authentication.

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