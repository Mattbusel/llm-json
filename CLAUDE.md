# llm-json

Single-header C++17 JSON parser and builder with recursive-descent parsing.

- Single-header C++17, namespace `llm`, stb-style `#ifdef LLM_LLM_JSON_IMPLEMENTATION` guard
- MIT License — Mattbusel, 2026
- No `/WX` in CMakeLists (avoids C4702 on MSVC)
- Examples use `std::getenv("OPENAI_API_KEY")` for credentials
