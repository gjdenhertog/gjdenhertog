# Style

I constantly learn and therefore gain new insights. I have a specific coding style which is adapted constantly with every new insight that I gain. This document captures the current rules I consider to be my style.
A lot of this is also inspired by books, blog posts and podcasts I consume. The biggest influence on my current status stems from the [Tiger Style](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md). Some of these rules are specific to my language of choice: [Rust](https://www.rust-lang.org).

## Simplicity & Elegance

TODO

## Technical Debt

> "Weeks of debugging can save hours of design." - Unknown

Solving a problem in design or implementation has less impact than solving it in production. I do not take on technical debt, I try to do it right the first time. I rather ship something that I can rely on with less features, than rush features and deal with problems in production at an ungodly hour.

## Safety

TODO

## Performance

TODO

## Developer Experience

> "There are only two hard things in Computer Science: cache invalidation, naming things, and
> off-by-one errors." — Phil Karlton

### Naming Things

TODO

### Cache Invalidation

TODO

### Off-By-One Errors

TODO

### Configurations

- Run `cargo fmt`.
- Use 4 spaces of indentation, rather than 2 spaces or tabs, as that is more obvious and leads to less nested code.
- Hard limit all line lengths, without exception, to at most 100 characters. Use it up but never go beyond.
- Add braces to `if` statements.

### Dependencies

My goal is to have as few dependencies as possible, preferrably none. Dependencies lead to supply chain attacks, safety and performance risks and slow install times.

### Tooling

Tools have costs. A small standardized toolbox is simpler to operate than an array of specialized instruments each with a dedicated manual. My primary tool is Rust. It may not be the best for everything, but it's good enough for most things. Tools created in Rust makes them cross-platform and portable with type safetey included.
