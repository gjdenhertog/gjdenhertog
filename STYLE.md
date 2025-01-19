# Style

I constantly learn and therefore gain new insights. I have a specific coding style which is adapted
constantly with every new insight that I gain. This document captures the current rules I consider
to be my style. A lot of this is also inspired by books, blog posts and podcasts I consume. The
biggest influence on my current status stems from the
[Tiger Style](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md). Some of
these rules are specific to my language of choice: [Rust](https://www.rust-lang.org).

## Simplicity & Elegance

Simplicity is bringing design goals together, how you can identify the "super idea" that solves the
axes simultaneously, to achieve something elegant. Simplicity is not the first attempt but the
hardest revision. The hardest part is how much thought goes into everything. Spend this energy
upfront, proactively rather than reactivly, because when thinking is done, what is spent on the
design will be dwarfed by the implementation and testing, and then again by the costs of operation
and maintenance.

> "Weeks of debugging can save hours of design." - Unknown

## Technical Debt

Solving a problem in design or implementation has less impact than solving it in production. I do
not take on technical debt, I try to do it right the first time. I rather ship something that I can
rely on with less features, than rush features and deal with problems in production at an ungodly
hour.

## Safety

[NASA's Power of Ten — Rules for Developing Safety Critical
Code](https://spinroot.com/gerard/pdf/P10.pdf) will change the way you code forever. To expand:

- Use **only very simple, explicit control flow** for clarity. **Do not use recursion** to ensure
  that all executions that should be bounded are bounded. Use **only a minimum of excellent
  abstractions** but only if they make the best sense of the domain. Abstractions are never zero
  cost. Every abstraction introduces the risk of a leaky abstraction.

- **Put a limit on everything** because, in reality, this is what we expect—everything has a limit.
  For example, all loops and all queues must have a fixed upper bound to prevent infinite loops or
  tail latency spikes. This follows the [“fail-fast”](https://en.wikipedia.org/wiki/Fail-fast)
  principle so that violations are detected sooner rather than later. Where a loop cannot terminate
  (e.g. an event loop), this must be asserted.

- Use explicitly-sized types like `u32` for everything, avoid architecture-specific `usize`.

- **Assertions detect programmer errors. Unlike operating errors, which are expected and which must
  be handled, assertion failures are unexpected. The only correct way to handle corrupt code is to
  crash. Assertions downgrade catastrophic correctness bugs into liveness bugs. Assertions are a
  force multiplier for discovering bugs by fuzzing.**

  - **Assert all function arguments and return values, pre/postconditions and invariants.** A
    function must not operate blindly on data it has not checked. The purpose of a function is to
    increase the probability that a program is correct. Assertions within a function are part of how
    functions serve this purpose. The assertion density of the code must average a minimum of two
    assertions per function.

  - **[Pair assertions](https://tigerbeetle.com/blog/2023-12-27-it-takes-two-to-contract).** For
    every property you want to enforce, try to find at least two different code paths where an
    assertion can be added. For example, assert validity of data right before writing it to disk,
    and also immediately after reading from disk.

  - On occasion, you may use a blatantly true assertion instead of a comment as stronger
    documentation where the assertion condition is critical and surprising.

  - Split compound assertions: prefer `assert(a); assert(b);` over `assert(a and b);`.
    The former is simpler to read, and provides more precise information if the condition fails.

  - **Assert the relationships of compile-time constants** as a sanity check, and also to document
    and enforce [subtle
    invariants](https://github.com/coilhq/tigerbeetle/blob/db789acfb93584e5cb9f331f9d6092ef90b53ea6/src/vsr/journal.zig#L45-L47)
    or [type
    sizes](https://github.com/coilhq/tigerbeetle/blob/578ac603326e1d3d33532701cb9285d5d2532fe7/src/ewah.zig#L41-L53).
    Compile-time assertions are extremely powerful because they are able to check a program's design
    integrity _before_ the program even executes.

  - **The golden rule of assertions is to assert the _positive space_ that you do expect AND to
    assert the _negative space_ that you do not expect** because where data moves across the
    valid/invalid boundary between these spaces is where interesting bugs are often found. This is
    also why **tests must test exhaustively**, not only with valid data but also with invalid data,
    and as valid data becomes invalid.

  - Assertions are a safety net, not a substitute for human understanding. With simulation testing,
    there is the temptation to trust the fuzzer. But a fuzzer can prove only the presence of bugs,
    not their absence. Therefore:
    - Build a precise mental model of the code first,
    - encode your understanding in the form of assertions,
    - write the code and comments to explain and justify the mental model to your reviewer.

- All memory must be statically allocated at startup. **No memory may be dynamically allocated (or
  freed and reallocated) after initialization.** This avoids unpredictable behavior that can
  significantly affect performance, and avoids use-after-free. As a second-order effect, it is our
  experience that this also makes for more efficient, simpler designs that are more performant and
  easier to maintain and reason about, compared to designs that do not consider all possible memory
  usage patterns upfront as part of the design.

- Declare variables at the **smallest possible scope**, and **minimize the number of variables in
  scope**, to reduce the probability that variables are misused.

- Restrict the length of function bodies to reduce the probability of poorly structured code. 
  Enforce a **hard limit of 70 lines per function**.

  Splitting code into functions requires taste. There are many ways to cut a wall of code into
  chunks of 70 lines, but only a few splits will feel right. Some rules of thumb:

  * Good function shape is often the inverse of an hourglass: a few parameters, a simple return
    type, and a lot of meaty logic between the braces.
  * Centralize control flow. When splitting a large function, try to keep all switch/if
    statements in the "parent" function, and move non-branchy logic fragments to helper
    functions. Divide responsibility. All control flow should be handled by _one_ function, the rest
    shouldn't care about control flow at all. In other words,
    ["push `if`s up and `for`s down"](https://matklad.github.io/2023/11/15/push-ifs-up-and-fors-down.html).
  * Similarly, centralize state manipulation. Let the parent function keep all relevant state in
    local variables, and use helpers to compute what needs to change, rather than applying the
    change directly. Keep leaf functions pure.

- Appreciate, from day one, **all compiler warnings at the compiler's strictest setting**.

- Whenever your program has to interact with external entities, **don't do things directly in
  reaction to external events**. Instead, your program should run at its own pace. Not only does
  this make your program safer by keeping the control flow of your program under your control, it
  also improves performance for the same reason (you get to batch, instead of context switching on
  every event). Additionally, this makes it easier to maintain bounds on work done per time period.

Beyond these rules:

- Compound conditions that evaluate multiple booleans make it difficult for the reader to verify
  that all cases are handled. Split compound conditions into simple conditions using nested
  `if/else` branches. Split complex `else if` chains into `else { if { } }` trees. This makes the
  branches and cases clear. Again, consider whether a single `if` does not also need a matching
  `else` branch, to ensure that the positive and negative spaces are handled or asserted.

- Negations are not easy! State invariants positively. When working with lengths and indexes, this
  form is easy to get right (and understand):

  ```rust
  if (index < length) {
    // The invariant holds.
  } else {
    // The invariant doesn't hold.
  }
  ```

  This form is harder, and also goes against the grain of how `index` would typically be compared to
  `length`, for example, in a loop condition:

  ```rust
  if (index >= length) {
    // It's not true that the invariant holds.
  }
  ```

- All errors must be handled. An [analysis of production failures in distributed data-intensive
  systems](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-yuan.pdf) found that
  the majority of catastrophic failures could have been prevented by simple testing of error
  handling code.

> “Specifically, we found that almost all (92%) of the catastrophic system failures are the result
> of incorrect handling of non-fatal errors explicitly signaled in software.”

- **Always motivate, always say why**. Never forget to say why. Because if you explain the rationale
  for a decision, it not only increases the hearer's understanding, and makes them more likely to
  adhere or comply, but it also shares criteria with them with which to evaluate the decision and
  its importance.

## Performance

- Think about performance from the beginning. The best time to solve performance is in the design
  phase, when we can not measure or profile. It is typically harder to fix a system after
  implementation and profiling, and the gains are less.
- Understand how your application uses memory.
- Profile before optimizing and establish a baseline.
- Optimize for the slowest resources first (network, disk, memory, CPU) in that order, after
  compensating for the frequency of usage, because faster resources may be used many times more.
- Use the minimum amount of carbon by making software as efficient as possible and optimize to run
  on renewable power.

## Developer Experience

> "There are only two hard things in Computer Science: cache invalidation, naming things, and
> off-by-one errors." — Phil Karlton

### Naming Things

- Get the nouns and verbs just right. Great names are the essence of great code, they capture what a
  thing is or does, and provide a crisp, intuitive mental model. They show that you understand the
  domain. Take time to find the perfect name, to find nouns and verbs that work together, so that
  the whole is greater than the sum of its parts.
- Use `snake_case` for function, variable, and file names. It helps separate words and encourages
  descriptive names.
- Do not abbreviate variable names.
- Add units or qualifiers to variable names, and put them last, sorted by descending significance,
  so that the variable starts with the most significant word, and ends with the least significant
  word.
- Try to choose realted names of the same length so that related variables all line up in the
  source. This makes the code symmetrical with clean blocks that are easier for the eye to parse and
  for the reader to check.
- Put important things near the top of a file, order matters.
- Think of how names will be used outside the code, in documentation or communication.
- Write descriptive commit messages that inform the reader.
- Code alone is not documentation, use comments to explain why you wrote the code the way you did.
- Comments are sentences, with a space after the slash, with capital letter and a full stop, or a
  colon if they relate to something that follows.

### Cache Invalidation

- Do not duplicate variables or take aliases to them. This will reduce the probability that state
  gets out of sync.
- Shrink the scipe to minimize the number of variables at play and reduce the probability that the
  wrong variable is used.
- Calculate or check variables to where/when they are used. Do not introduce variables before they
  are needed. Don't leave them around where they are not. Most bugs come down to a semantic gap,
  caused by a gap in time or space, because it's harder to check code that is not contained along
  those dimensions.
- Use simple function signatures and return types to reduce dimensionality at the call site, the
  number of branches that need to be handled at the call site, because this dimensionality can also
  be viral, propagating through the call chain.
- Ensure that functions run to completion without suspending, so that precondition assertiongs are
  true throughout the lifetime of the function. These assertions are useful documentation without a
  suspend, but may be misleading otherwise.

### Off-By-One Errors

- The usual suspects for off-by-one errors are casual interactions between `index`, a `count` or a
  `size`. These are all primitive integer types, but should be seen as distinct types, with clear
  rules to cast between them. To go from `index` to a `count` you need to add one, since indexes are
  0-based but counts are 1-based. To go from a `count` to a `size` you need to multiply by the unit.

### Configurations

- Run `cargo fmt`.
- Use 4 spaces of indentation, rather than 2 spaces or tabs, as that is more obvious and leads to
  less nested code.
- Hard limit all line lengths, without exception, to at most 100 characters. Use it up but never go
  beyond.
- Add braces to `if` statements.

### Dependencies

My goal is to have as few dependencies as possible, preferrably none. Dependencies lead to supply
chain attacks, safety and performance risks and slow install times.

### APIs

Only use versioning if absolutely necessary, try to resolve changes through evolution. Once an API
has been published, any externally observable behavior of the API cannot be changed without breaking
clients. Compatible changes can be implemented with an in-place update, the client does not break
and no new version needs to be created.

### Tooling

Tools have costs. A small standardized toolbox is simpler to operate than an array of specialized
instruments each with a dedicated manual. My primary tool is Rust. It may not be the best for
everything, but it's good enough for most things. Tools created in Rust makes them cross-platform
and portable with type safetey included.

### Developer's Journal

A developer's journal is a powerful tool for organizing thoughts, reducing ambiguity, and fostering
growth. By documenting goals, challenges, and solutions before, during, and after coding sessions,
developers can focus more effectively, avoid distractions, and learn from their experiences. The
journal serves as a private space to articulate problems, track progress, and reflect on successes
and struggles, ultimately promoting mindfulness and better decision-making. Regular journaling also
aids in retrospectives, performance reviews, and sharing insights with the team, making it a
valuable habit for personal and professional development. Read all about it
[here](https://stackoverflow.blog/2024/12/24/you-should-keep-a-developer-s-journal/).
