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