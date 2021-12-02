# WARNING: project is not production ready until it will be in `main` branch.

# NanoPb C++ 

![ci](https://github.com/hacker-cb/nanopb_cpp/actions/workflows/ci.yaml/badge.svg)

Extends C++ support for the [nanopb] and [protobuf].

Project is designed to map structures generated by [nanopb] and C++ models for easy encoding/decode complicated types like variable-length strings, arrays, maps, oneof(union) messages, etc.

## Features

* One-line call encode/decode for complicated messages.
* Reusable converters for inner messages.
* Template-based classes with static methods for converters.

## Available converters:

* `ScalarConverter` - Converter for basic scalar type like `enum`.
* `MessageConverter` - Converter for messages.
* `UnionMessageConverter` - Converter for union (oneof) messages.
* `CallbackConverter` - Converter for callback fields.  **Usually no need to use directly.**
* `StringCallbackConverter` - Converter for variable size string, mapped to `std::string`.
* `RepeatedCallbackConverter` - Converter for repeated message. **Usually no need to use directly.**
* `ArrayUnsignedCallbackConverter` - Repeated convertor for `std::vector<unsigned>` or `std::list<unsigned>` (Also support other signed types: `uint64_t`, `uint32_t`, etc).
* `ArraySignedCallbackConverter` - Repeated convertor for `std::vector<int>` or `std::list<int>` (Also support other signed types: `int64_t`, `int32_t`, etc).
* `ArrayStringCallbackConverter` - Repeated convertor for `std::vector<std::string>` or `std::list<std::string>`.
* `ArrayMessageCallbackConverter` - Repeated convertor for `std::vector<CLASS>` or `std::list<CLASS>`, NOTE: `MessageConverter` for  `CLASS` should be defined. 
* `MapCallbackConverter` - Map with any type of the key and value.

## Install

### CMake install

* Use `add subdirectory()` or `CPMAddPackage()` from [CPM] to add **nanopb_cpp** to your CMake project.   
* Set `NANOPB_VERSION` cmake variable to use custom nanopb version/git tag.
* [nanopb] will be downloaded via [CPM]. Set `NANOPB_ROOT` cmake variable to use your own nanopb location if you want to skip download.
* Use `target_link_libraries(YOUR_TARGET nanopb_cpp)` to add dependency.

[CPM] example:
```cmake
set(NANOPB_CPP_VERSION master)
CPMAddPackage(NAME lib_nanopb_cpp GITHUB_REPOSITORY hacker-cb/nanopb_cpp GIT_TAG ${NANOPB_CPP_VERSION})
```
NOTE: You can add `DOWNLOAD_ONLY YES` to `CPMAddPackage()` to prevent calling `add_subdirectory()`. Project will be just downloaded in this case and you should add sources to your target manually.  

### Manual install

* Install [nanopb]. Generate code from it.
* Add [nanopb_cpp.cpp](nanopb_cpp.cpp) to your project sources.

### PlatformIO install

FIXME: TODO.

## Usage

* Use `#include nanopb_cpp.h` in your code.
* Define C++ `class/struct` for each protobuf message generated by [nanopb].
* Define converter for each local/protobuf structs, inherit from `MessageConverter`.
* Define callback converters for callback field, inherit from `CallbackConverter`.
* Use other converters depending on your protobuf structures.

## Limitations

* All C++ classes should have default constructor
* All C++ classes should have copy constructor or move constructor

## Examples

See [examples](examples) folder for the examples.
Also, you can find more examples in [tests](test/tests) folder.

### Basic string example:

#### Protobuf:

```protobuf
syntax = "proto3";

package PROTO;

message TestMessage {
  string str = 1;
}
```

#### C++ local model:

```c++
struct LOCAL_TestMessage {
    std::string str;

    LOCAL_TestMessage() = default;
    LOCAL_TestMessage(const LOCAL_TestMessage&) = delete;
    LOCAL_TestMessage(LOCAL_TestMessage&&) = default;
};
```

#### Converter:

```c++
using namespace NanoPb::Converter;

class TestMessageConverter : public MessageConverter<
        TestMessageConverter,
        LOCAL_TestMessage,
        PROTO_TestMessage,
        &PROTO_TestMessage_msg>
{
public:
    static ProtoType encoderInit(const LocalType& local) {
        return ProtoType{
                .str = StringCallbackConverter::encoderInit(local.str)
        };
    }

    static ProtoType decoderInit(LocalType& local){
        return ProtoType{
                .str = StringCallbackConverter::decoderInit(local.str)
        };
    }

    static bool decoderApply(const ProtoType& proto, LocalType& local){
        return true;
    }
};
```

#### Usage:

```c++
const LOCAL_TestMessage original(
        {"My super string"}
        );

// Define output stream. 
// NOTE: max size is just for nanopb limit, actual buffer will grow on demand.
NanoPb::StringOutputStream outputStream(65535);

// Encode
if (!NanoPb::encode<TestMessageConverter>(outputStream, original)){
    // encode error
}

// Define input stream
auto inputStream = NanoPb::StringInputStream(outputStream.release());

LOCAL_TestMessage decoded;

/// Decode message
if (!NanoPb::decode<TestMessageConverter>(inputStream, decoded)){
    // decode error
}
```

[protobuf]: https://developers.google.com/protocol-buffers
[nanopb]: https://github.com/nanopb/nanopb
[CPM]: https://github.com/cpm-cmake/CPM.cmake
