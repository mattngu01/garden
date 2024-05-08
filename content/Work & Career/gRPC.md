https://www.altexsoft.com/blog/what-is-grpc/
https://grpc.io/docs/what-is-grpc/introduction/

> In gRPC, a client application can directly call a method on a server application on a different machine as if it were a local object, making it easier for you to create distributed applications and services.
> - https://grpc.io/docs/what-is-grpc/introduction/


Essentially [RPC], but Google's implementation on HTTP/2.

Advantages: very lightweight & performant

Uses [Protocol buffers](https://protobuf.dev/overview/) to serialize / deserialize structured data, then a compiler automatically generate data classes in preferred language
- explicit agreement on data structure, unlike REST

often compared to [GraphQL] due to its similar maturities, age
