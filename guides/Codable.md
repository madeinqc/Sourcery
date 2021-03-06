## I want to generate `Codable` implementation

This template generates `Codable` implementation for structs that implement  `AutoCodable`, `AutoDecodable` or  `AutoEncodable` protocols. You should define these protocols as follows:

```swift
protocol AutoDecodable: Decodable {}
protocol AutoEncodable: Encodable {}
protocol AutoCodable: AutoDecodable, AutoEncodable {}
```

### [Swift template](https://github.com/krzysztofzablocki/Sourcery/blob/master/Templates/Templates/AutoCodable.swifttemplate)

### Generating coding keys.

If you have few keys that are not matching default key strategy you have to specify only these keys, all other keys will be generated and inlined by the template:  
  

```swift
struct Person: AutoDecodable {
    let id: String
    let firstName: Bool
    let surname: String

    enum CodingKeys: String, CodingKey {
        // this is the custom key that you define manually
        case firstName = "first_name"

// sourcery:inline:auto:Person.CodingKeys.AutoCodable
        // the rest is generated by the template
        case id
        case surname
// sourcery:end
    }

}
```

Computed properties are not encoded by default, but if you define a coding key for computed property, template will generate code that will encode it.

If you don't define any keys manually the template will generate `CodingKeys` enum with the keys for all stored properties, but only if custom implementation of `init(from:)` or `encode(to:)` is needed.


### Generating `init(from:)` constructor.

Template will generate implementation of  `init(from:)` when needed. You can define additional methods and properties on your type to be used to decode it.
  
  - method to get decoding container. This is useful if your type needs to be decoded from a nested key(s):
  
```swift
struct MyStruct: AutoDecodable {
    let value: Int

    enum CodingKeys: String, CodingKey {
        case nested
        case value
    }

    static func decodingContainer(_ decoder: Decoder) throws -> KeyedDecodingContainer<CodingKeys> {
        return try decoder.container(keyedBy: CodingKeys.self)
            .nestedContainer(keyedBy: CodingKeys.self, forKey: .nested)
    }
}
```

  - method to decode a property. This is useful if you need to decode some property manually:
  
```swift
struct MyStruct: AutoDecodable {
    let myProperty: Int

    static func decodeMyProperty(from container: KeyedDecodingContainer<CodingKeys>) -> Int? {
        return (try? container.decode(String.self, forKey: .myProperty)).flatMap(Int.init)
    }
    //or
    static func decodeMyProperty(from decoder: Decoder) throws -> Int {
        return try decoder.container(keyedBy: CodingKeys.self)
            .decode(Int.self, forKey: .myProperty)
    }
}
```

These methods can throw or not and can return optional or non-optional result.

  - default property value. You can define a static variable that will be used as a default value of a property if decoding results in `nil` value:

```swift
struct MyStruct: AutoDecodable {
    let myProperty: Int

    static let defaultMyProperty: Int = 0
}
```

### Generating `encode(to:)` method.

Template will generate implementation of `encode(to:)` method when needed. You can define additional methods to be used to encode it.

  - method to get encoding container. This is useful if your type needs to be encoded into a nested key(s):
  
```swift
struct MyStruct: AutoDecodable {
    let value: Int

    enum CodingKeys: String, CodingKey {
        case nested
        case value
    }

    func encodingContainer(_ encoder: Encoder) -> KeyedEncodingContainer<CodingKeys> {
        var container = encoder.container(keyedBy: CodingKeys.self)
        return container.nestedContainer(keyedBy: CodingKeys.self, forKey: .nested)
    }
}
```

  - method to encode a property. This is useful when you need to manually encode a property:

```swift
struct MyStruct: AutoDecodable {
    let myProperty: Int

    func encodeMyProperty(to container: inout KeyedEncodingContainer<CodingKeys>) {
        try? container.decode(String(myProperty), forKey: .myProperty)
    }
    //or
    func encodeMyProperty(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(String(myProperty), forKey: .myProperty)
    }
}
```

These methods may throw or not. 

If you need to manually encode computed property and you have defined custom encoding method for it, template will generate a coding key for it too, so you don't have to define it manually (though you may still need to define it if it needs custom raw value).

  - method to encode any additional values. This is useful when you need to encode computed properties or constant values:

```swift
struct MyStruct: AutoDecodable {

    func encodeAdditionalValues(to container: inout KeyedEncodingContainer<CodingKeys>) throws {
        ...
    }
    // or
    func encodeAdditionalValues(to encoder: Encoder) throws {
        ...
    }
}
```
  
This method will be called in the end of generated encoding method.

  - enum `SkipEncodingKeys` for keys to be skipped during encoding. This is useful when you have stored properties that you don't want to encode, i.e. constants:
  
  ```swift
  struct MyStruct: AutoCodable {
      let value: Int
      let skipValue: Int
  
      enum SkipEncodingKeys {
          case skipValue
      }
  }
```

  
