# EffinitiveFramework Workspace Instructions

## Project Overview
High-performance C# web framework designed to outperform FastEndpoints and compete with GenHTTP. Now with HTTP/2 support.

## Checklist

- [x] Create copilot-instructions.md file in .github directory
- [x] Scaffold the C# solution structure
- [x] Implement core framework components
- [x] Create benchmark project
- [x] Create sample API project
- [x] Build and verify projects
- [x] Create documentation
- [x] Implement HTTP/2 support with binary framing
- [x] Implement HPACK header compression
- [x] Add ALPN negotiation for protocol selection
- [x] Create HTTP/2 documentation

## Development Guidelines
- Focus on zero-allocation patterns where possible
- Use Span<T>, Memory<T>, and ArrayPool for memory efficiency
- Minimize boxing and use struct types strategically
- Optimize hot paths with aggressive inlining
- Use BenchmarkDotNet for all performance measurements
- Target .NET 8+ for latest performance improvements
- Support both HTTP/1.1 and HTTP/2 protocols

## HTTP/2 Implementation
- Binary framing layer with all 9 frame types
- HPACK compression (static table + dynamic table + Huffman)
- ALPN negotiation during TLS handshake
- Stream multiplexing and flow control
- Follow RFC 7540 and RFC 7541 specifications

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HBartosch)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/HBartosch)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
