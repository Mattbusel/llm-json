# llm-json

[![CI](https://github.com/Mattbusel/llm-json/actions/workflows/ci.yml/badge.svg)](https://github.com/Mattbusel/llm-json/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![C++17](https://img.shields.io/badge/C%2B%2B-17-blue.svg)
![Single header](https://img.shields.io/badge/single-header-green.svg)

A small JSON parser and builder for C++ LLM code: build request bodies, parse responses, no dependencies.

> Part of **[llm-cpp](https://github.com/Mattbusel/llm-cpp)**, a family of 26 single-header C++ libraries for building on LLM APIs. Each one stands alone: copy one header, include it, done.

Almost every LLM call means building a JSON request and digging a string out of a JSON response. llm-json is a compact recursive-descent parser and builder with a friendly `Value` type, sized for that job, so you do not need to pull a large JSON library into a small tool.

## Features

- `Value` type covering null, bool, int, float, string, array and object; ints and floats are kept distinct
- Build with `operator[]`, `push_back`, `set`, initializer lists and `make_object({{"k", v}})`
- Objects keep insertion order when serialized
- `parse()` throws on bad input; `try_parse()` returns `ok`, an error message and the error offset instead
- Safe lookups with defaults via `get(key, default)`
- Compact or pretty-printed output with `dump()`

## Quick start

Requirements: a C++17 compiler. No other dependencies, no network access.

1. Copy [`include/llm_json.hpp`](include/llm_json.hpp) into your project.
2. In exactly one `.cpp` file, `#define LLM_JSON_IMPLEMENTATION` before including it. Other files just `#include "llm_json.hpp"`.

```cpp
#define LLM_JSON_IMPLEMENTATION
#include "llm_json.hpp"
#include <iostream>

namespace json = llm::json;

int main() {
    // Build a chat completion request body
    json::Value req = json::object();
    req["model"] = "gpt-4o-mini";
    req["temperature"] = 0.5;
    req["messages"] = json::array();
    req["messages"].push_back(json::make_object({{"role", "user"}, {"content", "Hello"}}));
    std::cout << req.dump_pretty() << "\n";

    // Parse a response without exceptions
    json::ParseResult pr = json::try_parse(R"({"choices":[{"message":{"content":"Hi!"}}]})");
    if (pr.ok)
        std::cout << pr.value["choices"][0]["message"]["content"].as_string() << "\n";
    else
        std::cout << "parse error at " << pr.error_offset << ": " << pr.error << "\n";
}
```

Build and run:

```bash
g++ -std=c++17 -I include example.cpp -o example
./example
```

Output:

```text
{
  "model": "gpt-4o-mini",
  "temperature": 0.5,
  "messages": [
    {
      "role": "user",
      "content": "Hello"
    }
  ]
}
Hi!
```

## API

Everything lives in namespace `llm` (`llm::json`).

| Function / type | What it does |
|---|---|
| `json::parse(str)` / `json::try_parse(str)` | Parse text into a `Value` (throwing, or returning a `ParseResult`) |
| `json::object()`, `json::array()`, `json::make_object({...})` | Construct containers |
| `Value::operator[]`, `get`, `contains`, `keys`, `erase`, `push_back`, `size` | Read and modify values |
| `is_*()` / `as_*()` | Type checks and typed accessors (throw on type mismatch) |
| `dump(indent)`, `dump_compact()`, `dump_pretty()` | Serialize |

## How it works

Parsing is a recursive-descent parser over the input string that handles standard escapes including `\u` sequences. Objects are stored as a vector of key/value pairs, which preserves insertion order and keeps small objects cheap; key lookup is a linear scan.

## Examples

The [`examples/`](examples) folder has runnable programs:

- [`basic_json.cpp`](examples/basic_json.cpp)
- [`basic_parse.cpp`](examples/basic_parse.cpp)
- [`build_request.cpp`](examples/build_request.cpp)
- [`json_types.cpp`](examples/json_types.cpp)
- [`parse_response.cpp`](examples/parse_response.cpp)

Build the examples with CMake:

```bash
cmake -B build
cmake --build build
```

## Limitations

- Linear key lookup makes very large objects slower than a hash-map based library.
- It is a strict JSON parser; it does not extract JSON from surrounding prose or markdown fences (see [llm-format](https://github.com/Mattbusel/llm-format) for that).

## License

MIT. See [LICENSE](LICENSE).
