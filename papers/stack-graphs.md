# Stack graphs — Name resolution at scale (Creager & van Antwerpen, EVCS 2023)

- Title: Stack graphs: Name resolution at scale
- Authors: Douglas A. Creager, Hendrik van Antwerpen
- Venue: Eelco Visser Commemorative Symposium (EVCS 2023)
- PDF: search/stack_graphs.pdf (in this repo)
- Software & repos: https://github.com/github/stack-graphs, https://github.com/tree-sitter/tree-sitter-graph

## TL;DR
Stack graphs extend scope graphs to support type-dependent name resolution while preserving file-incremental indexing. They maintain an explicit stack of pending lookups during path-based resolution and precompute per-file "partial paths" at index time to amortize query-time costs—enabling precise, scalable code navigation at GitHub scale.

## Short summary
GitHub needed low-latency, language-agnostic code navigation across millions of repos and hundreds of languages. Existing scope-graph variants either supported file-incremental indexing (Néron) or type-directed lookups (van Antwerpen) but not both. Stack graphs combine the advantages: (1) encode bindings as graph paths with push (↓) and pop (↑) nodes that manipulate an explicit symbol stack; (2) keep file subgraphs disjoint (no non-virtual cross-file edges), using virtual root-to-root edges at query time to cross files; (3) perform type-directed intermediate lookups lazily by pausing lookups on the symbol stack during traversal; and (4) precompute per-file partial paths (with pre/post symbol-stack conditions) at index time to reduce repeated work during queries. The paper formalizes paths, partial paths, and shows how to construct and compose them. Implementation uses tree-sitter + a declarative graph-construction language (tree-sitter-graph). Stack graphs were deployed in production (Python) at GitHub (since Nov 2021).

## Detailed walkthrough

### Motivation & constraints
- Scale: petabytes of code, thousands of pushes/minute, many languages; need file-incremental analysis to avoid reprocessing unchanged files.
- Latency goals: query latency ≲100ms; index time should be reasonably fast so navigation is available soon after pushes.
- Prior work tradeoffs:
  - Néron scope graphs: file-incremental but no type-dependent lookups.
  - van Antwerpen extensions: type-dependent lookups via eager resolution at index-time but create cross-file edges, breaking incrementality.

### Stack graph representation
- Nodes: root nodes, scope nodes, push symbol nodes (↓x), pop symbol nodes (↑x).
- Edges: directed, always within a single file; crossing files only via virtual root-to-root edges added at query time.
- Push nodes seed the symbol stack; pop nodes act as guards that require stack-top to match and pop it.
- A path is i ⇝ i' {stack}. A complete binding path starts at a reference node, ends at a definition node, and has an empty stack.

### Path construction and resolution algorithm
- Lift nodes into (initial) paths: push nodes seed stacks, other nodes create empty paths.
- Append edges to expand paths; apply stack rules per node type (noop, push, pop).
- Use BFS-style exploration of paths, appending compatible edges (including virtual root edges) until complete paths (definitions) are discovered.
- Maintain cycle detection and handle ambiguous or missing bindings (useful for navigation over incomplete/ill-typed code).

### Partial paths (index-time precomputation)
- Partial paths: fragments of paths within a file, annotated with pre/post partial symbol stacks (can include a stack-variable).
- At index time, compute all partial paths between important endpoints (roots, defs, refs, key scopes) and store them keyed by file blob id.
- At query time, concatenate compatible partial paths (unifying pre/post conditions) instead of exploring edges step-by-step; this preserves soundness and completeness while shifting much work to index-time and keeping index processing file-local.

### Implementation & language support
- Graph construction is purely syntactic and declarative via tree-sitter-graph: language experts map AST patterns to graph gadgets.
- The resolution algorithm is language-agnostic and runs over the constructed stack graphs.
- Storage model: per-file subgraphs / partial-paths keyed by git blob id for reuse across commits; this enables file-incremental indexing and avoids storing per-commit duplicate artifacts.

## Production status
- Stack graphs have been running in production since November 2021 for Python on GitHub.
- Core libraries (stack-graphs, tree-sitter-graph, and rulesets) are open source (see paper references and repos).

## Key contributions
- A formal graph model (stack graphs) supporting type-directed lookups via an explicit symbol stack while preserving file-incrementality.
- A query-time path semantics that delays type-dependent lookups and an index-time partial-path precomputation strategy to amortize query cost.
- A pragmatic, language-agnostic pipeline using tree-sitter for broad language support and per-file artifact caching keyed by blob ids.

## Strengths
- Achieves both file-incrementality and type-directed resolution—resolving a key scaling tradeoff.
- Language-agnostic resolution algorithm with declarative per-language graph construction rulesets; avoids per-project builds.
- Production-proven at GitHub scale with open-source tooling.

## Limitations & open challenges
- Query-time cost remains nontrivial; partial paths mitigate but tradeoff between index-time work/storage and query latency exists.
- Partial-path storage overhead depends on language and repository; deduplication via git blobs helps but needs measurement.
- Correctness and precision depend on ruleset quality; complex languages may require intricate rules.
- Cycle/path explosion and worst-case behavior require careful cycle detection and heuristics.

## Reproducibility checklist (starter experiment)
To reproduce a basic experiment and measure costs/benefits:

1. Environment & libs
   - Clone and build: `github/stack-graphs`, `tree-sitter`, `tree-sitter-graph`, and language rulesets referenced by the authors.
   - Use a persistent store keyed by git blob id (e.g., object store or DB keyed by blob sha) to save per-file artifacts.

2. Data selection
   - Pick a small set of repositories in a target language (e.g., 10 Python repos of varying size).
   - For each repo, iterate through a sequence of commits to evaluate caching via blob reuse.

3. Index pipeline (per-file, file-incremental)
   - For each unique file blob, parse with tree-sitter and generate the stack subgraph via tree-sitter-graph ruleset.
   - Compute and persist partial paths between designated endpoints (roots, defs, refs, import/export scopes).
   - Record index-time per-file duration and persisted partial-path size.

4. Query pipeline
   - Implement the path-finding / partial-path concatenation algorithm described in the paper.
   - Measure query latency distribution (median, p95, p99) for a suite of realistic code navigation queries (jump-to-definition, find-references).
   - Measure cache hit rates for partial paths via blob reuse.

5. Baselines & comparisons
   - Compare with naive edge-traversal stack-graph query (no partial paths) to measure query-speedup.
   - Compare storage/index costs with LSIF-style full-project LSIF artifacts for the same repo snapshot.

6. Metrics to report
   - Index-time per unique file and total index time per commit sequence.
   - Storage: total partial-path bytes per repository and deduplication effects across commits.
   - Query latency: mean / median / p95 / p99; throughput under concurrent load.
   - Correctness: compare resolved definitions against an LSP or ground-truth for a set of queries.

## Suggested follow-ups / experiments
- Evaluate partial-path storage growth and deduplication in large repos over time.
- Measure end-to-end latency on polyglot repositories and for statically-typed languages (Java/TypeScript) with richer type semantics.
- Experiment with prioritized or cost-bounded path search to improve p99 latency.
- UX experiments on handling multiple/ambiguous resolutions and ranking heuristics.

## Next steps I can help with
- Commit this note to `papers/stack-graphs.md`.  
- Produce a starter repository or scripts that parse files, build stack subgraphs with tree-sitter, compute partial paths for a toy repo, and run queries to measure timings.  
- Create a concise one-page cheat-sheet (pseudocode + worked example) for implementers.

---
*Tell me whether you want me to commit this file directly to `main` at `papers/stack-graphs.md`, or create a branch and open a PR. I’ll proceed after you confirm.*