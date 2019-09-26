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

# Advanced Types 
## "oneof"

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

## Maps

* Maps can be used to map scalars (except float / double) to values of any type.

```proto3
message MyMessage {
  int32 id = 1;
  map<string, Result> results = 2;
}
```
* Map fields cannot be repeated.
* There is no ordering.

## Timestamp
Fields are seconds and nanoseconds
``` proto3
import "google/protobuf/timestamp.proto";

message MyMessage {
  google.protobuf.Timestamp my_field = 1;
}
```
## Duration

``` proto3
import "google/protobuf/duration.proyo";

message MyMessage {
  google.protobuf.Timestamp my_field = 1;
  google.protobuf.Duration validaty = 2;
}
```

# Protocol Buffers Options
* Options allow to alter the behavior of the protoc compiler when generating code for specific languages.
* There are few bundled options (read the docs).

# Naming Convention
* Use CamelCase for message names.
* Use underscore_separated_names for fields

```proto3
message MyLongMessage {
  string my_longfield = 1;
}
```

* Use CamelCase for Enum and CAPITAL_WITH_UNDERSCORE for values names

```
enum Foo {
  FIRST_VALUE = 0;
  SECOND_VALUE = 1;

}
```

# Intro to Protocol Budder Services
* Protocol buffers can define Services on top of Message.
* A service is a set of endpoints your application can be accessible from.
* Services need to be interpreted by a framework to generate associated code.
```proto3
//Example
serivce SearchService {
  rpc Search (SearchRequest) returns (SearchResponse)
}
```
* The main framework is gRPC but you may find others in the web.
* The advantage of Services & RPC (remote procedure calling) is that you can call your Server API from any client seamlessly.
* gRPC for example is used by Google, Netflix, CoreOS (etcd), Google Cloud API, and is gaining popularity fast.

# Today's trend is to build microservices
Microservices must exchange information and need to agree on:
* The API to exchange data
* The data format
* The error patterns
* Load Balancing
* Many other
One of the popular chioce for building API is REST (http-json)

Building API:
* Need to think about data model: JSON/XML/BINARY?
* Need to think about endpoint: GET /api/v1/user/123/post/456
* Need to think about how to invoke it and handle errors: API/Error
* Need to think about efficiency of the API: How much data do I get out of one call?
* Latency
* Scalability to 1000s of clients?
* Load balancing?
* Intter operability with many language?

# What's gRPC?
* gRPC is a free and open-source framework developed by Google
* gRPC is part of the Cloud Native Foundation (CNCF) - like Docker & Kubernetes for example
* At a high level, it allows you to define REQUEST and RESPONSE for RPC (Remote Procedure Calls) and handles all the rest for you.
* On top of it, it's modern, fast and efficient, build on top of HTTP/2, low latency, supports streaming, language independent, and makes it super easy to plug in authentication, load balancing, logging and monitoring.

```proto3
// Example
syntax = "proto3";
message Greeting {
  string first_name = 1;
}

message GreetRequest {
  Greeting greeting = 1;
}

message GreetResponse {
  string result = 1;
}

service GreetService {
  rpc Greet(GreetRequest) returns (GreetResponse) {};
}

```

# Why Protocol Buffers?
* Protocol Budders are language agnostic.
* Code can be generated for pretty much any language.
* Data is binary and efficiently serialized (small payloads).
* Very convenient for transporting a lot of data.
* Protocol Buffers allows for easy API evolution using rules.
* gRPC is the future micro-services API and mobile-server API (and maybe Web APIs).

* Easy to write message definition,
* The definition of the API is independent from the implementation.
* A huge amount of code can be generated, in any language, from a simple .proto file.
* The payload is binary, therefore very efficient to send / receive on a network and serialize / de-serializer on a CPU.
* Protocol Buffers defines rules to make an API evolve without breaking existing clients, which is helpful for microservices.

# Protocol Buffers Encoding

message -> Bytes stream -> message


# Learn gRPC

Structure: 1. Theory 30mins 2. Hands on Programming 3. Advanced

# Efficiency of Protocol Buffers over JSON

* Save Network Bandwidth
* Parsing JSON is actually CPU intensive (because the format is human readable)
* Parsing Protocol Buffers (binary format) is less CPU intensive because it's closer to how a machine represents data.
* By using gRPC, the use of Protocol Buffers means faster and more efficient communication, friendly with with mobile devices that have a slower CPU.

# gRPC can be used by any language
* Because the code can be generated for any language, it makes it super simple to create micro-services in any language that interact with each other.

# HTTP/2 vs HTTP/1

## HTTP/1:
* HTTP 1.1 was released in 1997.
* opens a new TCP connection to a server at each request
* It does not compress headers (which are plaintext)
* It only works with Request / Response mechanism (no server push)
* HTTP was originally composed of two commands: GET, POST

Nowadays, a web page loads 80 assets on average. Headers are sent at every request and are PLAINTEXT (heavy size). Each request opens a TCP connection. These inefficiencies add latency and increase network packet size.

## HTTP/2
* HTTP2 was released in 2015. 
* HTTP2 support multiplexing: The client & server can push messages in parallel over same TCP connection. This greatly reduces latency.
* HTTP2 supports server push: Servers can push streams (multiple messages) for one request from the client. This saves round trip.
* HTTP2 supports header compression.
* HTTP2/2 is binary while HTTP/1 text makes it easy for debugging. It's not efficient over the network.(Protocol buffers is a binary protocol and makes it a great match for HTTP2)
* HTTP/2 is secure (SSL is not required but recommended by default).

## HTTP/2 Bottom Line
* Less chatter
* More efficient protocol (less bandwidth)
* Reduced Latency
* Increased Security
* And you get all these improvements out of the box by using the gRPC framework!

# 4 Types of API in gRPC
* Unary
* Server Streaming
* Client Streaming
* Bi Directional Streaming

# Scalability in gRPC
* gRPC Servers are asynchronous by default
* This means they do not block threads on request
* Therefore each gRPC server can serve millins of requests in parallel
* gRPC Clients can be asynchronous or synchrounous (blocking)
* The client decides wich model works best for the performance needs
* gRPC Clinets can perform client side load balancing
* google has 10B gRPC requets per second

# Security in gRPC

* By default gRPC strongly advocates for you to use SSL (encryption over the wire) in your API
* This means that gRPC has security as a first class citizen
* Each language will provide an API to load gRPC with the required certificates and provide encryption capability out of the box
* Additionally using Interceptors, we can also provide authentication (we'll learn about Interceptors in the advanced section)

# REST vs gRPC

| gRPC | REST | 
|-------|---------|
| Protocol Buffers - smaller, faster | JSON -text based slower, bigger |
| HTTP/2 (lower latency) - from 2015 | HTTP1.1 (high latency) - from 1997 |
| Bidirectional & Async | Client => Server requests only |
| Stream Support | Request / Response support only |
| API Oriented -"What" (No constraints - free design) | CRUD Oriented (Create-Retrieve-Update-Delete / POST GET PUT DELETE) | 
|Code Generation through Protoco Buffers in any language - 1st class citizen | Code generation through OpenAPI/Swagger (add-on) - 2nd class citizen |
|RPC Based -gRPC does the plumbing for us | HTTP verbs based - we have to write the plumbing or use a 3rd party library |
| RPC Based - gRPC does the plumbing for us | HTTP verbs based - we have to write the plumbing or use a 3rd party library | 

gRPC is 25 times more performant than REST.

# Sumary

# What's an Unary API

* Unary RPC calls are the basic Request/Response that everyone is familiar with.
* Client will send one message to the server and will receive one response from the server.
* In gRPC Unary Calls are defined using Portocol Buffers
* For each RPC call we have to define a "Request" message and a "Response" message.

# Error Codes

* With gRPC, there a few error codes: https://grpc.io/docs/guides/error.html
* There is also a complete reference to implementation of error codes that close a lot of gaps with the documentation: http://avi.im/grpc-errors : intro to how to hanlde error (both client and server) in many language.

# gRPC Deadlines
* Deadlines allow gRPC clients to specify how long they are willing to wait for an gRPC to complete before the RPC is terminated with the error DEADLINE_EXCEEDED
* The gRPC documentation recommends you set a deadline for all client RPC calls.
* Setting the deadline is up to you: how long do you feel your API should have complete?
* Server should check if the deadline has exceeded and cancel the work it is doing.
* Bolg: https://grpc.io/bolg/deadlines

