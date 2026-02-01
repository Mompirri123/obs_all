# Turn 3 Evaluation — furl Ordered Query Option (Model A vs Model B)

## Evaluation Table

| Question of which is / has | Answer Given | Justoification Why? |
| --- | --- | --- |
| Overall Better Solution | A slightly better than B | Both rely on the existing `sort` / `query_sort` option, but A keeps broader edge‑case coverage (dont_quote, empty keys, special characters, no mutation). |
| Better logic and correctness | A slightly better than B | A’s tests cover more boundary cases and regression risks; B’s tests are good but narrower. |
| Better Naming and Clarity | Tie | No new names introduced in either; both keep existing `sort`/`query_sort` terms. |
| Better Organization and Clarity | Tie | No structural changes; both only touch tests/docstrings. |
| Better Interface Design | Tie | Both preserve default behavior and use the existing optional `sort`/`query_sort` flags. |
| Better error handling and robustness | A slightly better than B | A includes more edge cases (dont_quote, empty keys, special chars, no mutation), reducing risk. |
| Better comments and documentation | B slightly better than A | B adds clearer narrative comments around stable ordering and deterministic output, but A is still solid. |
| Ready for review / merge | A slightly better than B | A’s broader coverage makes it safer to merge with fewer gaps; B removes some previously covered cases. |

## Model A — Pros and Cons

**Pros**
- Model A does a good job at keeping the existing `sort` / `query_sort` option while adding a wide test net around it (quote_plus, dont_quote, empty keys/values, special characters, and “no mutation” of original order). This makes it clear the option is deterministic without breaking other encoding behaviors.
- Tests verify deterministic output for different insertion orders and ensure defaults remain unchanged.

**Cons**
- It doesn’t add new functionality beyond tests and a small docstring tweak, so the improvement is mainly validation rather than implementation.
- Some new deterministic cases (stable ordering of duplicate keys, non‑string keys) are covered more explicitly in Model B.

## Model B — Pros and Cons

**Pros**
- Model B adds clear tests for stable sorting, non‑string keys, and deterministic output across different insertion orders.
- Tests are explicit about default behavior vs sorted behavior and include query_sort combined with other options.

**Cons**
- It drops several edge‑case tests present in A (dont_quote, empty keys, special characters, and “sort does not mutate original order”). That shrinks coverage and may let regressions slip.
- Changes a docstring line but doesn’t add new implementation logic; the improvement is mostly tests and wording.

## Justification — Why one is better
Both models use the existing `sort`/`query_sort` option to provide deterministic URL output without changing defaults. Model A is slightly better overall because it retains broader test coverage for edge cases that are easy to break (quoting rules, empty keys/values, and preserving insertion order when sort is off). Model B’s tests are good but narrower; it trades some important edge cases for new deterministic checks. That makes A the safer merge choice for maintaining correctness across more scenarios.
