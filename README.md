# Atlas Device SDK to Couchbase Lite Comparison Guide
The Atlas Device SDK can be thought of as an on-device ORM that creates an “object store” that is then synced to MongoDb Atlas.  The following document describes the features of Atlas and how they compare to Couchbase Mobile.

## Guidance
These documents aim to elucidate the distinct differences in SDK functionalities and setup requirements between MongoDB Atlas/Atlas Device SDK and Couchbase Mobile. The Atlas Device SDK offers tailored approaches across different languages and platforms. Each section of our guidance is dedicated to a specific platform, complete with quick links to that platform’s documentation, enabling developers to access the information they need swiftly and efficiently.

- [Android](android.md)
- [.NET](dotnet.md)
- [Objective-C](objectivec.md)
- [Swift](swift.md)

## Ream Java Support
Realm only supports the [Java](https://www.mongodb.com/docs/atlas/device-sdks/sdk/java/#sdk-in-maintenance-mode) programming language for Android based applications.  If you are an Android developer, use the Android link under guidance.  The Couchbase Lite documentation provides a single location for both Java and Kotlin Android developers.

## Couchbase Lite Java support
Couchbase Lite supports the Java programming language for non-Android based applications via a separate SDK. You can find more information from the [documentation](https://docs.couchbase.com/couchbase-lite/current/java/gs-prereqs.html).  

## Realm C++ Support
The Atlas Device SDK supports the [C++ programming language](https://www.mongodb.com/docs/atlas/device-sdks/sdk/cpp/).   Couchbase Lite supports the [C programming language](https://docs.couchbase.com/couchbase-lite/current/c/gs-install.html).  

- [Realm C++ Platform Support](https://github.com/realm/realm-cpp?tab=readme-ov-file#getting-started)
- [Couchbase Lite C Platform Support](https://docs.couchbase.com/couchbase-lite/current/c/gs-install.html#lbl-platforms)

## Node.js SDK support
Couchbase Lite currently doesn't support Node.js based applications.

## Web SDK
Couchbase Lite currently doesn't support Web based applications.

## Open Source SDK projects
Couchbase Lite has open source projects that support the following platforms:

- [React Native](https://cbl-reactnative.dev/)
- [Ionic Capacitor](https://cbl-ionic.dev/)
- [Flutter/Dart](https://cbl-dart.dev/)
- [Kotlin Multiplatform](https://kotbase.dev/current/)
 
## Support
The documentation provided herein is offered “as-is” without any form of support. We do not guarantee the accuracy, reliability, or timeliness of the information contained within these documents. Users should be aware that the use of this documentation is at their own risk, and it is their responsibility to ensure that the information meets their specific requirements.