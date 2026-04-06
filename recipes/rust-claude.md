# CLAUDE.md for Rust Projects

## Build & Test Commands
- **Build Project:** `cargo build`
- **Run Project:** `cargo run`
- **Run Tests:** `cargo test`
- **Run Specific Test:** `cargo test test_name`
- **Linting:** `cargo clippy -- -D warnings`
- **Formatting:** `cargo fmt`
- **Documentation:** `cargo doc --open`

## Rust Architecture & Conventions
- **Error Handling:** - Use `thiserror` for library-level structured errors.
    - Use `anyhow` for application-level / main function error handling.
    - Avoid `unwrap()` or `expect()` in production code; prefer `?` or explicit error mapping.
- **Workspaces:** For multi-crate projects, manage dependencies in the root `Cargo.toml` using `[workspace.dependencies]`.
- **Unsafe Code:** - **NEVER** use `unsafe` unless performance-critical and accompanied by a `// SAFETY:` comment.
    - Prefer safe wrappers or standard library abstractions.
- **Naming:** - Variables/Functions: `snake_case`.
    - Types/Traits: `PascalCase`.
    - Macros: `snake_case!`.

## Code Style & Quality
- **Clippy:** Adhere to all Clippy suggestions. Common ignores must be documented in `Cargo.toml`.
- **Async:** Use `tokio` as the default runtime unless otherwise specified.
- **Traits:** Prefer "Composition over Inheritance" via Trait implementation.
