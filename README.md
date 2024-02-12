Akka.Serialization.MessagePack
===
[![Build status](https://ci.appveyor.com/api/projects/status/xaltap7v4n0m042d/branch/dev?svg=true)](https://ci.appveyor.com/project/akkadotnet-contrib/akka-serialization-messagepack/branch/dev) [![NuGet Version](http://img.shields.io/nuget/v/Akka.Serialization.MessagePack.svg?style=flat)](https://www.nuget.org/packages/Akka.Serialization.MessagePack/)

Akka.NET serialization with [MessagePack](https://github.com/neuecc/MessagePack-CSharp)

The `Akka.Serialization.MessagePack` plugin is designed to:

 - Be Low Ceremony for common cases.
 - Allow flexiblity for advanced cases.
 - Provide High Performance
 - Allow for Optional LZ4 compaction.

### Payload Versioning Notes

#### Assembly Versions
This plugin has a few different options to control payload serialization/deserialization:

 - `allow-assembly-version-mismatch`
   - If `False`, Assembly version mismatches between payload and availiable deserializer will throw.
   - Default is `True`
 - `omit-assembly-version`
   - Controls whether Assembly version is included in payload.


#### Schema Evolution

One *can* do a form of 'Schema evolution' (if you squint at it from the right angle) with MessagePack payload definitions.

Consider the following:

```csharp
[MessagePackObject(false)]
public class WireType
{
   [Key(0)]
   public int Foo {get;set;}
   [Key(1)]
   public string Bar {get;set;}
   [Key(2)]
   public byte[] Bin {get;set;}
}
```

The above uses Messagepack attributes to define that every item is packed into an array.

If one wanted to add a new field, they could do, for example,

```csharp
[MessagePackObject(false)]
public class WireType
{
   [Key(0)]
   public int Foo {get;set;}
   [Key(1)]
   public string Bar {get;set;}
   [Key(2)]
   public byte[] Bin {get;set;}
   [Key(3)]
   public byte[]? OtherBin {get;set;}
}
```

The consumer will, of course, need to do it's own checks for whether `OtherBin` is null, however the serializer itself will be able to handle using/ignoring data appropriately.

Two additional notes:

1. Any empty slots will be filled with nil up to the max Key in the array. So, in the above example, if `OtherBin` had a Key index of 64, it would be serialized as a 64 slot array with 60 `nil` values between `Bin` and `OtherBin`.
2. One can apply inheritance and DU-style behavior via attributes, see the MP docs for details.

## How to setup MessagePack as default serializer
Bind MessagePack serializer using following HOCON configuration in your actor system settings:
```hocon
akka {
  actor {
    serializers {
      messagepack = "Akka.Serialization.MessagePack.MsgPackSerializer, Akka.Serialization.MessagePack"
    }
    serialization-bindings {
      "System.Object" = messagepack
    }
  }
}
```

## Benchmarks
``` ini

BenchmarkDotNet=v0.10.9, OS=Windows 10 Redstone 2 (10.0.15063)
Processor=Intel Core i5-6400 CPU 2.70GHz (Skylake), ProcessorCount=4
Frequency=2648439 Hz, Resolution=377.5809 ns, Timer=TSC
.NET Core SDK=2.0.0
  [Host]      : .NET Core 2.0.0 (Framework 4.6.00001.0), 64bit RyuJIT
  NETCORE 2.0 : .NET Core 2.0.0 (Framework 4.6.00001.0), 64bit RyuJIT

Job=NETCORE 2.0  Platform=X64  Runtime=Core  
Server=True  Toolchain=CoreCsProj  

```
 |                                                 Method |        Mean |      Error |     StdDev |  Gen 0 | Allocated |
 |------------------------------------------------------- |------------:|-----------:|-----------:|-------:|----------:|
 |                               MsgPack_serialize_string |    248.7 ns |   2.221 ns |   2.078 ns | 0.0029 |     112 B |
 |                              Hyperion_serialize_string |    414.2 ns |   5.646 ns |   5.281 ns | 0.0257 |     832 B |
 |                               JsonNet_serialize_string |  1,854.9 ns |  36.749 ns |  43.748 ns | 0.1355 |    4336 B |
 |                         MsgPack_serialize_SimpleObject |    351.6 ns |   5.425 ns |   5.074 ns | 0.0037 |     136 B |
 |                        Hyperion_serialize_SimpleObject |    965.3 ns |  12.820 ns |  11.992 ns | 0.0331 |    1112 B |
 |                         JsonNet_serialize_SimpleObject | 23,832.4 ns | 339.575 ns | 317.639 ns | 0.4008 |   14576 B |
 |       MsgPack_serialize_SimpleOptimizedObject_int_keys |    200.1 ns |   1.425 ns |   1.333 ns | 0.0019 |      72 B |
 | Hyperion_serialize_SimpleOptimizedObject_preregistered |    493.4 ns |   6.110 ns |   5.715 ns | 0.0216 |     712 B |
 |                       MsgPack_serialize_TypelessObject |  2,096.3 ns |  21.150 ns |  19.784 ns | 0.0120 |     568 B |
 |                      Hyperion_serialize_TypelessObject |  5,698.9 ns |  34.648 ns |  32.410 ns | 0.1465 |    4952 B |
 |                       JsonNet_serialize_TypelessObject | 42,583.6 ns | 291.306 ns | 258.235 ns | 0.3342 |   13960 B |


## Maintainer
- [alexvaluyskiy](https://github.com/alexvaluyskiy)
