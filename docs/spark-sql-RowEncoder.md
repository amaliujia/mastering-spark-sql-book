title: RowEncoder

# RowEncoder -- Encoder for DataFrames

`RowEncoder` is part of the spark-sql-Encoder.md[Encoder framework] and acts as the encoder for spark-sql-DataFrame.md[DataFrames], i.e. `Dataset[Row]` -- spark-sql-Dataset.md[Datasets] of spark-sql-Row.md[Rows].

NOTE: `DataFrame` type is a mere type alias for `Dataset[Row]` that expects a `Encoder[Row]` available in scope which is indeed `RowEncoder` itself.

`RowEncoder` is an `object` in Scala with <<apply, apply>> and other factory methods.

`RowEncoder` can create `ExpressionEncoder[Row]` from a spark-sql-StructType.md[schema] (using <<apply, apply method>>).

[source, scala]
----
import org.apache.spark.sql.types._
val schema = StructType(
  StructField("id", LongType, nullable = false) ::
  StructField("name", StringType, nullable = false) :: Nil)

import org.apache.spark.sql.catalyst.encoders.RowEncoder
scala> val encoder = RowEncoder(schema)
encoder: org.apache.spark.sql.catalyst.encoders.ExpressionEncoder[org.apache.spark.sql.Row] = class[id[0]: bigint, name[0]: string]

// RowEncoder is never flat
scala> encoder.flat
res0: Boolean = false
----

`RowEncoder` object belongs to `org.apache.spark.sql.catalyst.encoders` package.

=== [[apply]] Creating ExpressionEncoder For Row Type -- `apply` method

[source, scala]
----
apply(schema: StructType): ExpressionEncoder[Row]
----

`apply` builds spark-sql-ExpressionEncoder.md[ExpressionEncoder] of spark-sql-Row.md[Row], i.e. `ExpressionEncoder[Row]`, from the input spark-sql-schema.md[StructType] (as `schema`).

Internally, `apply` creates a spark-sql-Expression-BoundReference.md[BoundReference] for the spark-sql-Row.md[Row] type and returns a `ExpressionEncoder[Row]` for the input `schema`, a `CreateNamedStruct` serializer (using <<serializerFor, `serializerFor` internal method>>), a deserializer for the schema, and the `Row` type.

=== [[serializerFor]] `serializerFor` Internal Method

[source, scala]
----
serializerFor(inputObject: Expression, inputType: DataType): Expression
----

`serializerFor` creates an `Expression` that <<apply, is assumed to be>> `CreateNamedStruct`.

`serializerFor` takes the input `inputType` and:

1. Returns the input `inputObject` as is for native types, i.e. `NullType`, `BooleanType`, `ByteType`, `ShortType`, `IntegerType`, `LongType`, `FloatType`, `DoubleType`, `BinaryType`, `CalendarIntervalType`.
+
CAUTION: FIXME What does being native type mean?

2. For ``UserDefinedType``s, it takes the UDT class from the `SQLUserDefinedType` annotation or `UDTRegistration` object and returns an expression with `Invoke` to call `serialize` method on a `NewInstance` of the UDT class.

3. For spark-sql-DataType.md#TimestampType[TimestampType], it returns an expression with a spark-sql-Expression-StaticInvoke.md[StaticInvoke] to call `fromJavaTimestamp` on `DateTimeUtils` class.

4. ...FIXME

CAUTION: FIXME Describe me.
