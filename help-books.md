# 📘 Implementing a macOS Help Book (macOS 15 “Sequoia” and Later)

This guide provides a modern, real-world-tested walkthrough for implementing a native Help Book system in a macOS app, based on deep debugging and live analysis of macOS 15 (Sequoia). It includes undocumented system behaviors, cache handling, and proven best practices.

## ✅ Overview

Apple's Help Book system **still works** in macOS 15, but:
- It now routes help through the **Tips.app** (rebranded Help Viewer),
- It aggressively **caches Help Books**, even persisting outdated HTML,
- And it lacks up-to-date official documentation for SwiftUI or sandboxed apps.

[Full implementation details continue here...]
