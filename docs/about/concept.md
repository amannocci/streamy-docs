# Concepts

## Source

## Flow

## Sink

## Transformer

## Json

To manipulate Json, Streamy proposes its own Json AST because Json is the base of everything in Streamy.

### Json AST

The base type in Streamy is MaybeJson, and has several subtypes representing different Json types :
- JsUndefined : An undefined Json value. It is useful to avoid Option[Json]  pattern in some case and for performance reason.
- JsDefined : A defined Json value.
    - JsObject : A Json object, represented as a Map.
    - JsArray : A Json array, represented as an Array.
    - JsString : A Json String.
    - JsBoolean: A Json Boolean.
    - JsNull: A Json Null.
    - JsBytes : A Json Bytes.
    - JsNumber : A Json number, represented has several subtypes.
        - JsInt : A Json Int.
        - JsLong : A Json Long.
        - JsFloat : A Json Float.
        - JsDouble: A Json Double.
        - JsBigDecimal: A Json Big Decimal.

### Reading and writing json

The `io.techcode.streamy.util.json._`  package has several methods for reading and writing json.

#### Reading from ByteString

Parse a ByteString input into a Json value and will throw a ParseException in case of failure.

```scala
val result: Json = Json.parseByteStringUnsafe(ByteString("""
{
  "foo": "bar"
}
"""))
```

Parse a ByteString input into a Json value and will return an Either[Throwable, Json] value.

```scala
val result: Either[Throwable, Json] = Json.parseByteString(ByteString("""
{
  "foo": "bar"
}
"""))
```

#### Reading from String

Parse a String input into a Json value and will throw a ParseException in case of failure.

```scala
val result: Json = Json.parseStringUnsafe("""
{
  "foo": "bar"
}
""")
```

Parse a ByteString input into a Json value and will return an Either[Throwable, Json] value.

```scala
val result: Either[Throwable, Json] = Json.parseString("""
{
  "foo": "bar"
}
""")
```

#### Reading from Bytes
Parse an Array[Byte] input into a Json value and will throw a ParseException in case of failure.

```scala
val result: Json = Json.parseBytesUnsafe("""
{
  "foo": "bar"
}
""".getBytes())
```

Parse a ByteString input into a Json value and will return an Either[Throwable, Json] value.

```scala
val result: Either[Throwable, Json] = Json.parseBytes("""
{
  "foo": "bar"
}
""".getBytes())
```

#### Writing to ByteString

Print a Json input into a ByteString value and will throw a PrintException in case of failure.

```scala
val result: ByteString = Json.printByteStringUnsafe(Json.obj("foo" -> "bar"))
```

Print a Json input into a ByteString and will return aEither[Throwable, ByteString] value.

```scala
val result: Either[Throwable, ByteString] = Json.printByteString(Json.obj(
  "foo" -> "bar"
))
```

#### Writing to String

Print a Json input into a String value and will throw a PrintException in case of failure.

```scala
val result: String = Json.printStringUnsafe(Json.obj("foo" -> "bar"))
```

Print a Json input into a String and will return aEither[Throwable, String] value.

```scala
val result: Either[Throwable, String] = Json.printString(Json.obj(
  "foo" -> "bar"
))
```

### Manipulating Json AST

#### Evaluate

To read a value from a Json AST, Streamy provides a implementation of the [RFC-6901](https://tools.ietf.org/html/rfc6901).  
To simplify, we will use the following json object.

```scala
import io.techcode.streamy.util.json._

# Object example
val json = Json.obj(
  "string" -> "string"
  "int" -> 1024,
  "long" -> 1024L,
  "float" -> 1.0F,
  "double" -> 1.0D,
  "big_decimal" -> BigDecimal("1"),
  "boolean" -> true,
  "bytes" -> ByteString("bytes"),
  "object" -> Json.obj(
    "string" -> "obj_string"
  ),
  "array" -> Json.arr(
    "arr_string"
  )
)
```

You can attempt to read any value at any level.  
The `evaluate` method will returns either a Json value or a JsUndefined.

```scala
json.evaluate(Root / "string")
// JsString("string")

json.evaluate(Root / "unknown")
// JsUndefined
```

You can read json values as valid primitive. Be careful that you can't cast any json value to any primitive and a cast exception will be raised in such cases.

```scala
// Evaluate a field as String value
json.evaluate(Root / "string").get[String]
// string

// Evaluate a field as Int value
json.evaluate(Root / "int").get[Int]
// 1024

// Evaluate a field as Long value
json.evaluate(Root / "long").get[Long]
// 1024

// Evaluate a field as Float value
json.evaluate(Root / "float").get[Float]
// 1.0

// Evaluate a field as Double value
json.evaluate(Root / "double").get[Double]
// 1.0

// Evaluate a field as BigDecimal value
json.evaluate(Root / "big_decimal").get[BigDecimal]
// 1

// Evaluate a field as Boolean value
json.evaluate(Root / "boolean").get[Boolean]
// true

// Evaluate a field as Bytes value.
json.evaluate(Root / "bytes").get[ByteString]
// ByteString("bytes")

// Evaluate a field as a non valid value.
json.evaluate(Root / "bytes").get[String]
// throw ClassCastException
```

You can read json values as valid json primitives.

```scala
// Evaluate a field as JsNumber value
json.evaluate(Root / "int").get[JsNumber]

// Evaluate a field as Json and ensure that the value isn't empty.
json.evaluate(Root / "object").get[Json]

// Evaluate a field as JsObject value.
json.evaluate(Root / "object").get[JsObject]
json.evaluate(Root / "object" / "string").get[String]

// Evaluate a field as JsArray value.
json.evaluate(Root / "array").get[JsArray]
json.evaluate(Root / "array" / 0).get[String]

// Evaluate a field or default
json.evaluate(Root / "unknown").getOrElse[String]("default")

// Evaluate a field and transform
json.evaluate(Root / "string").map(_ => "Hello world !")

// Evaluate a field and maybe transform
json.evaluate(Root / "string").flatMap(_ => JsUndefined)

// Evaluate a field and do something only if exist
json.evaluate(Root / "string").ifExist(println)
```

#### Patch

## Parser

## Printer

## StreamEvent