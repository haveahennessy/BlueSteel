![Gilt Tech logo](https://raw.githubusercontent.com/gilt/Cleanroom/master/Assets/gilt-tech-logo.png)

# BlueSteel

An Avro encoding/decoding library for Swift.

BlueSteel is part of [the Cleanroom Project](https://github.com/gilt/Cleanroom) from [Gilt Tech](http://tech.gilt.com).


### Never heard of Avro?

Take a gander at the [official documentation for Avro](http://avro.apache.org/docs/current/) before reading further.


### Swift 2.1 compatibility

The `master` branch of this project is Swift 2.1 compliant and therefore **requires Xcode 7.1 or higher to compile**.


### Adding BlueSteel to your project

[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

You’ll need to [integrate BlueSteel into your project](https://github.com/gilt/BlueSteel/blob/master/INTEGRATION.md) in order to use [the API](https://rawgit.com/gilt/BlueSteel/master/Documentation/index.html) it provides. You can choose:

- [Manual integration](https://github.com/gilt/BlueSteel/blob/master/INTEGRATION.md#manual-integration), wherein you embed BlueSteel’s Xcode project within your own, **_or_**
- [Using the Carthage dependency manager](https://github.com/gilt/BlueSteel/blob/master/INTEGRATION.md#carthage-integration) to build a framework that you then embed in your application.
 
Once integrated, just add the following `import` statement to any Swift file where you want to use BlueSteel:

```swift
import BlueSteel
```


### API documentation

For detailed information on using BlueSteel, [API documentation](https://rawgit.com/gilt/BlueSteel/master/Documentation/index.html) is available.


## Usage

Since Avro data is not self describing, we're going to need to supply an Avro Schema before we can (de)serialize any data. Schema enums are constructed from a JSON schema description, in either String or NSData form.

```swift
import BlueSteel

let jsonSchema = "{ \"type\" : \"string\" }"
let schema = Schema(jsonSchema)
```

### Deserializing Avro data

Using the Schema above, we can now decode some Avro binary data.

```swift
let rawBytes: [Byte] = [0x6, 0x66, 0x6f, 0x6f]
let avro = AvroValue(schema: schema, withBytes: rawBytes)
```

We can now get the Swift String from the Avro value above using an optional getter.
```swift
if let avroString = avro.string {
    println(avroString) // Prints "foo"
}
```

### Serializing Swift data

We can use the same Schema above to serialize an AvroValue to binary.

```swift
if let serialized = avro.encode(schema) {
    println(serialized) // Prints [6, 102, 111, 111]
}
```

#### But how do we convert our own Swift types to AvroValue?

By conforming to the AvroValueConvertible protocol! You just need to extend your types with one function:
```swift
func toAvro() -> AvroValue
```

Suppose we wanted to serialize a NSUUID with the following schema:

```JSON
{
    "type" : "fixed",
    "name" : "UUID",
    "size" : 16
}
```

We could extend NSUUID as follows:

```swift
extension NSUUID : AvroValueConvertible {
    public func toAvro() -> AvroValue {
        var uuidBytes: [Byte] = [Byte](count: 16, repeatedValue: 0)
        self.getUUIDBytes(&uuidBytes)
        return AvroValue.AvroFixedValue(uuidBytes)
    }
}
```

To generate and serialize a NSUUID, we could now do:

```swift
let serialized: [Byte]? = NSUUID().toAvro().encode(uuidSchema)
```
Hey presto! We now have a byte array representing an NSUUID serialized to Avro according to the fixed schema provided.
Okay, so the example above is maybe a little bit too simple. Let's take a look at a more complex example. Suppose we have a record schema as follows:

```JSON
{
    "type": "record", 
        "name": "test",
        "fields" : [
        {"name": "a", "type": "long"},
        {"name": "b", "type": "string"}
    ]
}
```

We could create a corresponding type Swift that might look something like this:
```swift
struct testStruct {
    var a: Int64 = 0
    var b: String = ""
}
```

To convert testStruct to AvroValue, we could extend it like this:

```swift
extension testStruct : AvroValueConvertible {
    func toAvro() -> AvroValue {
        return AvroValue.AvroRecordValue([
                "a" : self.a.toAvro(),
                "b" : self.b.toAvro()])
    }
}
```

You might've noticed above that we called .toAvro() on Int64 and String values. We didn't have to define these ourselves because BlueSteel provides AvroValueConvertible extensions for Swift primitives.

So that just about covers a very quick introduction to BlueSteel. Please note that BlueSteel is still very early in development and may change significantly.


## About

The Cleanroom Project began as an experiment to re-imagine Gilt’s iOS codebase in a legacy-free, Swift-based incarnation. 

Since then, we’ve expanded the Cleanroom Project to include multi-platform support. Much of our codebase now supports tvOS in addition to iOS, and our lower-level code is usable on Mac OS X and watchOS as well.

Cleanroom Project code serves as the foundation of Gilt on TV, our tvOS app [featured by Apple during the launch of the new Apple TV](http://www.apple.com/apple-events/september-2015/). And as time goes on, we'll be replacing more and more of our existing Objective-C codebase with Cleanroom implementations.

In the meantime, we’ll be tracking the latest releases of Swift & Xcode, and [open-sourcing major portions of our codebase](https://github.com/gilt/Cleanroom#open-source-by-default) along the way.


### Contributing

BlueSteel is in active development, and we welcome your contributions.

If you’d like to contribute to this or any other Cleanroom Project repo, please read [the contribution guidelines](https://github.com/gilt/Cleanroom#contributing-to-the-cleanroom-project).


### Acknowledgements

[API documentation for BlueSteel](https://rawgit.com/gilt/BlueSteel/master/Documentation/index.html) is generated using [Realm](http://realm.io)’s [jazzy](https://github.com/realm/jazzy/) project, maintained by [JP Simard](https://github.com/jpsim) and [Samuel E. Giddins](https://github.com/segiddins).

