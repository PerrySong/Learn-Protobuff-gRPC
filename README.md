# Protocol Buffers

Pros:
* Data is fully typed
* Data is compressed automatically (less CPU usage)
* Schema (defined using .proto file) is needed to generate code and read the data
* Documentation can be embedded in the schema
* Data can be read across any language
* Schema can evolve over time, in a safe manner (schema evolution)
* 3-10x smaller, 29-100x faster than XML
* Code is generated for you automatically!

Cons:
* Support languages might be lacking
* Can not "open" the serialized data with a text editor

Google -> use it for almost all internal project

# How to use:
.proto file Human-readable -> Java/Go/Python -> Encode / decode -> Serialized Data

gRPC use Protocol Buffers to exchange data


# Updating Protocol Rules
* 1. Do not change the numeric tags for any existing fields.
* 2. You can add new fields, and old code will just ignore them
* 3. Likewise, if the old / new code reads unknown data, the default will take place
* 4. Fields can be removed, as long as tag number is not used again in your updated message type. You may want to rename the field instead perhaps adding the prefix "OBSOLETE_", or make the tag reserved so that future users of your .proto can not accidentally reuse the number.
* 5. For data type changes (int32 to int64 for example, please refer to documentation) 

# Adding Fields
* Let's add a field to our schema (new tag number)

```proto
// Old schema
messsage myMessage {
  int32 id = 1;
}

// New schema
message myMessage {
  int32 id = 1;
  string first_name = 2;
}
```

* If that field is sent to old code, the old code will not know what that tag number corresponds to and the field will be ignored.
* Oppositely, if we read old data with the new code, the new field will not be found and the default value will be assumed (empty string)

* Default values should always be interpreted with care.


# Renaming Field
* Keep tag number and everything is fine

# Removing fields

Lets remove a field in our schema

```proto
// Old schema
messsage myMessage {
  int32 id = 1;
  string first_name = 2;
}

// New schema
message myMessage {
  int32 id = 1;
}
```

* If old code doesn't find the field anymore, the default value will be used.
* Oppositedly, if we read old data with the new code, the deleted field will just be dropped.
* Default value should always be interpreted with care!

## Removing Fields Reserved Tags (Recommended)

* When removing a field you should ALWAYS reserve the tag and the name

```proto
// Example:
// old
message myMessage {
  int32 id = 1;
  string first_name = 2;
}

//new
message MyMessage {
  reserved 2;
  reserved "first_name";
  int32 id = 1;
}
```

* This prevents the tag to be re-used and this prevents the name to be re-used
* This is necessary to prevent conflicts in the codebase

## Removing Fields Make some fields obsolete
* The alternative is that instead of removing a feild, you rename it to OBSOLETE_field_name.
* The downside is that you may have to populate that field while your client get upgraded to use the newer field that replaces it (Which has a new tag)

# Reserved KeyWORDS
* You can reserve TAGS and FIELD NAMES.
* Your can't mix TAGS and FIELD NAMES in the same reserved statement.
* We reserve TAGS to prevent new fields from re-using tags (which would break old code at runtime)
* We reserce FIELD NAMES to prevent code bugs.
* Do not ever remove any reserved tags.

# Beware of Defaults!

* Defaults are great but they are tricky to deal with

* Defaults allow us to easily evolve Protobuf files without breaking any existing or new codes.
* They also ensure we know that a field will always have a non-null values
* But they are dangerous, because...
* You cannot differentiate from a mssing field or id a value equal to the default was set.

* So make sure the default value doesn't have meaning for your business.

* Deal with default values in your code if needed (with if statements)

# Evolving Enumerations
* Enumerations can evolve 
  * add, remove or reserve
* If the code doesn't know what the received Enum value will be used .
* Therefore, make first value "UNKNOWN = 0". (Recommended)

# Protocal Buffer Advanced
# Integer Types

* int32, int64, uint32, uint64, sint32, sint64, fixed32, fixed64, sfixed32, sfixed64

* Each type is basically constructed to hanlde:
  1. Range of allowed values: 64 bits has more values than 32 bits.
  2. Whether negative values are allowed.
  3. Size efficiency on serialization.

* This advanced and meant for perfomance and space optimization

* sint32, sint64 encode negative values will (through the use of yechnique called ZigZag). Choose accordingly based on if your field can have negative values!

* uint*, int*, sint* (* = 32 or * = 64), are variable encoding meaning that if they can use less space, they will (for small values)

* fixed32 use 4bytes constantly. More efficient than uint32 if values are often greater than 2^28, fixed64 use 8 bytes constantly. More efficient than uint64 if values are often greater than 2^56.

# Advanced Types "oneof"

You can use oneof to tell protocol buffers that only one field can have a value:
``` proto
message MyMessage {
  int32 id = 1;
  oneof example_oneof {
    string my_string = 2;
    bool my_bool = 3;
  }
}
```
* oneof fields cannot be repeated

* Evolving schemas using oneof is complicated (see documentation)
* On read, all fields wil be null except the last one that was set at write

