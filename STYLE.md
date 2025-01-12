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

- Get the nouns and verbs just right. Great names are the essence of great code, they capture what a thing is or does, and provide a crisp, intuitive mental model. They show that you understand the domain. Take time to find the perfect name, to find nouns and verbs that work together, so that the whole is greater than the sum of its parts.
- Use `snake_case` for function, variable, and file names. It helps separate words and encourages descriptive names.
- Do not abbreviate variable names.
- Add units or qualifiers to variable names, and put them last, sorted by descending significance, so that the variable starts with the most significant word, and ends with the least significant word.
- Try to choose realted names of the same length so that related variables all line up in the source. This makes the code symmetrical with clean blocks that are easier for the eye to parse and for the reader to check.
- Put important things near the top of a file, order matters.
- Think of how names will be used outside the code, in documentation or communication.
- Write descriptive commit messages that inform the reader.
- Code alone is not documentation, use comments to explain why you wrote the code the way you did.
- Comments are sentences, with a space after the slash, with capital letter and a full stop, or a colon if they relate to something that follows.

### Cache Invalidation

- Do not duplicate variables or take aliases to them. This will reduce the probability that state gets out of sync.
- Shrink the scipe to minimize the number of variables at play and reduce the probability that the wrong variable is used.
- Calculate or check variables to where/when they are used. Do not introduce variables before they are needed. Don't leave them around where they are not. Most bugs come down to a semantic gap, caused by a gap in time or space, because it's harder to check code that is not contained along those dimensions.
- Use simple function signatures and return types to reduce dimensionality at the call site, the number of branches that need to be handled at the call site, because this dimensionality can also be viral, propagating through the call chain.
- Ensure that functions run to completion without suspending, so that precondition assertiongs are true throughout the lifetime of the function. These assertions are useful documentation without a suspend, but may be misleading otherwise.

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

### Developer's Journal

A developer's journal is a powerful tool for organizing thoughts, reducing ambiguity, and fostering growth. By documenting goals, challenges, and solutions before, during, and after coding sessions, developers can focus more effectively, avoid distractions, and learn from their experiences. The journal serves as a private space to articulate problems, track progress, and reflect on successes and struggles, ultimately promoting mindfulness and better decision-making. Regular journaling also aids in retrospectives, performance reviews, and sharing insights with the team, making it a valuable habit for personal and professional development. Read all about it [here](https://stackoverflow.blog/2024/12/24/you-should-keep-a-developer-s-journal/).
