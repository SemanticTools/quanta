# Quanta: A Research Program on Language Without Transformers

*Sigmantics AB — a retrospective on the ancestor experiments, the cuts, and the distillation into QuantaR1*

## What this is

This is an honest writeup of a six-month concentrated research program on non-transformer approaches to language understanding, and the working system — QuantaR1 — that came out of it. The technical content matters, but the more transferable story is about **research discipline**: specifically, about making the same "stop exploring, distill, deliver" move four times at progressively higher altitudes, and ending up with something real rather than something grand-but-vapourware.

Quanta isn't a product. It's the distilled endpoint of a research lineage, and this writeup is as much about the lineage as about the endpoint.

## Phase 1 — Breadth. Many angles, each asking a different question.

Before the main line, there was a breadth-first exploration across wildly different paradigms. None of these were Quanta, but each of them asked a question whose answer fed forward.

**catLM** was the first attempt at custom-made embeddings — word vectors assembled from hashed consonants, hashed vowels, word position, POS-tag weights, and a handful of other dimensions. No neural network at query time; pure vector-space scoring. It always produced an answer, which was both its charm and its main limitation, and it could write in three tones (funny, short, professional). The lesson: you can get surprisingly far with handcrafted embeddings, but the ceiling is low and you pay for it in brittleness. What survived: the fuzzy-matching substrate that eventually became Mnt-Matcher.

**hybrid2** was a consciousness-inspired generator with an "unconscious buffer" of salient ideas that accumulated over time and bubbled up to influence text generation. Domain-aware Markov chains, fragment expansion, slot filling. The interesting part wasn't the generation quality; it was the architecture of having a dynamic latent state that influenced but didn't fully determine output. hybrid2 didn't die — its unconscious-simulator component is now a live part of **i-total**, a separate multi-agent discussion project.

**markov++** was the deliberate floor experiment: how far can you get with n-gram statistics plus a handwritten POS lexicon plus variable-order backoff? This is the minimal neuro-symbolic baseline that most papers quietly assume is trivial and skip past. Running it to exhaustion is actually informative, and the lesson — *coherence emerges later than you'd think, meaning emerges never* — shapes what you expect from anything purely statistical.

**simpleton** was a swarm of tiny domain experts, each with its own vocabulary and Markov-style transition table, with a router on the front. Mixture-of-experts at the micro scale, with trivially inspectable routing (`canHandle()` as a vocabulary lookup). Its value was as a legibility testbed — every piece of state is human-readable, so you can study what structure is actually needed for text generation and what's just scale.

**summy** and **summy2** came from the hypothesis that *you only understand text if you can summarize it*. summy2 ended up occupying an unusual middle position between classical statistical summarizers and neural ones: pattern matching to hand-curated concept frames, weighted by empirical frequency — FrameNet-style semantic proxies without embeddings. The summy family also had several visible dead ends (failed BNF parser, failed probabilistic BNF parser, failed pattern extension), which is worth noting because the public artifact count understates the actual exploration cost.

**pennyf** and **pennyf2** attempted sentence generation loosely constrained by a corpus — enough corpus influence to sound grounded, not enough to overfit. A different angle on the same generation problem hybrid2 and markov++ were probing.

**perseverity** was the one that closed out the breadth phase. It matched whole sentences in a source corpus for retrieval, sharing architecture with Intentia (which came later in folder dates but arguably earlier in design lineage). The stated moonshot was ambitious: *a fractal recursive language model*. The basics worked. The fractal part was never attempted.

**This is where the first cut happened.** Perseverity's basics working told you the direction was real; the fractal moonshot's unattainability told you that chasing it directly would eat the research program. So you parked the fractal ambition and started Intentia as something grounded and achievable.

(Dates on the breadth phase are lost — these folders were all touched during the archaeology that produced this writeup, so their mtimes are unreliable. They ran sometime in early-to-mid 2025, before the depth phase began.)

## Phase 2 — Depth. July to late September 2025, one main line.

This is the compressed part. Three months, two major iteration lines, and the direct predecessor of Quanta. Dates are from folder mtimes, verified.

**Intentia 1 through 6 (Jul 2 → Aug 5, 2025).** Six iterations in five weeks. The architecture that emerged: layered parsing with perplexity backtracking, fourteen layers numbered from 0 to 30 with deliberate gaps (so new layers could slot in without renumbering), layers that could modify and consume context within a turn or across turns, built-in calculus and number conversion, access to a directory of topic JSONs, access to current date and time, an embryonic category system, and a WordNet crawler for concept extraction. The last of those taught the most important lesson of the whole phase: **WordNet is not enough**. Simple facts like *"a cat has four legs"* aren't there. The knowledge source has to come from somewhere else. This realization is what eventually motivated Quanta's atom store, and it's the single most pivotal moment in the research program.

**pennyf (Jul 31)** slipped into the late-Intentia window as a side thread on corpus-constrained generation.

**summy and the summy family (early August)** ran in parallel with the Intentia→Paralaxia transition. summy2 is described in Phase 1 because its lineage started earlier; the specific Aug 5 artifact is its most mature form.

**Paralaxia 1 through 6b, plus scholar and milestone branches (Aug 2 → Sep 23, 2025).** Eight weeks. Paralaxia is the direct Quanta predecessor and the longest sustained single experiment in the program. Its key architectural idea was **fragment-first matching**: instead of storing whole patterns like *"what is the \*?"*, store *"what is"* and *"the \*?"* as combinable atoms, and let a fragment combo matcher assemble them at parse time. It had domains (numbers, pets) so cross-domain reasoning was at least conceptually possible, though the domains themselves were thin. The name suggests parallelism, but the parallel execution was never implemented — there wasn't enough data volume to make performance a real concern, so the sequential version was sufficient.

**parallaxia_scholar (Aug 29, 2025)** is the single most important subfolder in the whole program for understanding what Quanta is and isn't. This was **Timmy** (also called Scholar): an attempt to have an LLM teach Paralaxia new patterns through dialog. The implementation worked. The results didn't. The diagnosis — which is worth stating precisely because it generalizes — is twofold:

1. **LLMs in 2024–2025 weren't reliable enough in a tight loop.** You'd get five good outputs, then one that silently dropped a field, then three good ones, then one that hallucinated a new field. For a harvesting loop that's accumulating into a pattern library, that's catastrophic — one bad pattern poisons the corpus and you don't notice until a query breaks weeks later.

2. **The target format was wrong.** Patterns were asked for in Scholar DSL syntax. LLMs in that era were already native JSON speakers because tool-use had trained them on it, but Scholar was a dialect of one. The harvesting loop was asking the model to do two hard things at once — understand the linguistic pattern *and* serialize it into an unfamiliar syntax. Splitting those, by asking for JSON and deterministically emitting Scholar from it, would have collapsed a whole class of failures. The lesson is general: **when an LLM is a producer in your pipeline, meet it where it's fluent.**

Scholar is parked, not dead. Its conditions for revival — reliable structured output, tool use that doesn't silently break — arrived in 2026.

## The second cut. Late September 2025.

At the end of Paralaxia 6b, the breadth and depth exploration had produced a working vocabulary of ideas: layered parsing, fragment-first matching, certainty as a first-class runtime concept, categories, the atom idea, the Scholar-style harvesting loop as an aspirational but not-yet-feasible mechanism. Each individual experiment had taught something; none of them was by itself a deliverable.

The second cut was the move from exploration to distillation. Quanta began.

## Phase 3 — Delivery. Quanta1, Quanta2, the third cut, and QuantaR1.

**Quanta1 (Oct 2025)** was the first real distillation. The architecture it settled into is a six-stage pipeline — and I want to describe it precisely because it's the thing that matters technically:

1. **Parse input to patterns.** A single input can match multiple patterns, and word variants and question variants are explored together. Fragment-based matching inherited from Paralaxia. A single prompt can decompose into multiple sub-queries (*"what is a cat, and how does a dog eat"*), and the parse stage does joint segmentation and pattern-matching with a global scoring function.

2. **Pattern to query matching.** Mapping surface patterns onto the query plans that know how to satisfy them. Roughly thirty query shapes in Quanta1 — *what-is*, *where-is*, *who-is*, *how-many-X-does-Y-have*, *is-X-a-Y*, *X-vs-Y*, sequence queries over ordered categories, and more.

3. **Query data requirements to data gathering.** Fuzzy matching of query subjects by name — so *"chat"* resolves to candidate subjects *cat* and *hat* with confidences attached.

4. **Query execution.** With a subtlety that matters: if the top-scoring combination of (query interpretation, data binding) isn't decisively better than the runners-up, alternative executions run **sequentially** for the other candidates inside a certainty window. This happens at both the query level and the data level — so *"chat"* might run as *cat* and *hat*, and each of those might run under multiple query interpretations.

5. **Query response generation.** Each executed branch produces a candidate response with a final score.

6. **Query response concatenation.** For a single sub-query, the winner is chosen by **argmax over final scores** — the scoring happens *after* execution, so a parse that looked promising but produced an empty atom lookup loses to one that looked slightly worse but produced a rich answer. For multi-query prompts, the independently-chosen winners of each sub-query are concatenated.

Two things about this pipeline are worth naming, because they're where Quanta is doing something architecturally non-obvious rather than just tidily implementing the obvious.

The first is **enumerate-then-argmax**. Most systems commit to the top interpretation at the earliest possible stage and throw away the alternatives. Quanta defers commitment until after execution, so "this parse looked good but led nowhere" is representable as a low final score rather than as a crash or an empty answer. This is the move that makes the six-stage pipeline behave like a search rather than a waterfall.

The second is the **half-implemented negative feedback mechanism**. If the user rejects the last answer (*"No!"*), Quanta marks the chosen (query, data-binding) combo for the previous input shape as bad, and re-runs the prompt with that combo demoted in scoring. In a pure argmax-over-final-score system, this is almost free — the runners-up are already computed and scored; you just exclude the winner and promote the next one. It's the symbolic equivalent of RLHF's thumbs-down, operating on inspectable (pattern, binding) tuples rather than on an opaque reward model. The mechanism is sketched but not fully propagated; completing it is one of the clearest high-leverage items in the roadmap.

Quanta1 also carried forward a lot of the ancestor organs: fragment-first matching from Paralaxia, the category system embryo from Intentia, the fuzzy matching substrate from catLM, the atom-store answer to the WordNet-isn't-enough problem. It more or less worked. It had an OpenAI-compatible `/v1/chat/completions` endpoint, so it dropped into any existing chat UI as just another model id.

**Quanta2 (Oct 2025, parked).** With Quanta1 working, the natural next step was an ambitious next-generation architecture. Quanta1 was copied, and rewriting began.

It didn't land. Not because the architecture was wrong — it might well have been right — but because **the rewrite had no intermediate testable versions**. Too much code had to move before anything would run, which meant there was no way to know whether the new architecture would hold without paying most of the cost up front. That violated the methodology every other cut in the program had respected: keep the thing runnable at every step so you can see whether you're on the right track. Perseverity's basics ran before its moonshot was attempted. Intentia iterated 1→6 with each version runnable. Paralaxia iterated 1→6b with milestones as visible anchors. Quanta2 was asking for a long invisible runway, and the moment that became clear, the right move was to walk.

**The third cut, and QuantaR1 (late Oct 2025 onward).** Instead of grandstanding on Quanta2, the move was to go back to Quanta1, tidy it up, descope it, and release it as **QuantaR1** — Release 1. The thing that actually exists. The thing you can show.

QuantaR1 is what Quanta1 distilled down to when you asked *"what's the smallest subset of this that is coherent enough to ship, stable enough to maintain, and complete enough to be interesting?"* It is deliberately smaller than Quanta1 was trying to be, and much smaller than Quanta2 was trying to be. It is also the first version in the entire program that is a deliverable rather than an experiment.

**Quanta2 is parked, not dead.** Its conditions for revival — better LLM tooling to carry the grunt work of the rewrite, reliable structured output, tool use that doesn't silently break — are the same conditions that revive Scholar.

## The pattern that runs through all of this

Four cuts, each at a higher altitude than the last, each the same move:

1. **Park the fractal moonshot.** End of perseverity, start of Intentia.
2. **End the breadth-first exploration.** Commit to Intentia → Paralaxia as the depth-first main line.
3. **End the depth-first exploration.** Paralaxia 6b → Quanta1.
4. **Abandon the Quanta2 architectural ambition.** Tidy and descope Quanta1 → QuantaR1.

Each cut is: *I have learned enough from this ambitious thing. Stop, distill, deliver the smaller version that exists.* The discipline isn't stopping when things get hard; it's stopping when **the shape of the work stops matching the shape of work you know how to do well**. Invisible-runway rewrites, unbounded moonshots, and ever-growing exploration all share the property that you can't tell whether they're succeeding, and the consistent move across all four cuts is to refuse that condition.

Most research projects in adjacent areas don't survive the transition from exploration to delivery. They either ship v0.55 of the exploration as the product (brittle, illegible, unsupportable) or they chase the ambitious rewrite until the funding or the motivation runs out. QuantaR1 exists because the program consistently made the less satisfying choice — the smaller, honest, shippable one.

## The two ceilings that are moving

Two of the most interesting parked items in the program — **Scholar harvesting** and **Quanta2's rewrite** — were blocked for the *same underlying reason*, which is worth naming because it affects what the next phase looks like.

Both were limited by the state of LLM tooling in 2024–2025. Scholar couldn't get reliable structured output from the harvesting loop. Quanta2 couldn't get the rewrite cost down to something that could have intermediate testable versions. Both of those limitations were about LLMs-as-reliable-tools, not about the ideas themselves. And both of those limitations have eased substantially in 2026: structured output actually works now, tool use is reliable, the grunt work that made ambitious rewrites infeasible can now be partially automated.

That means the two biggest deferred pieces of the program are newly feasible *at roughly the same time, for roughly the same reason*. The next phase — whenever it happens — isn't a vague gesture at future work. It's a specific claim: the two ideas that were ceiling-blocked by LLM tooling can both be picked back up with the tools that didn't exist the first time. Scholar harvesting inside an AssetForge-style multi-agent loop. Quanta2's rewrite with LLM-assisted migration and structured intermediate checkpoints.

## What's interesting as a showcase

Three things, in roughly increasing order of what matters.

**The technical content.** A running, inspectable, six-stage symbolic language pipeline with enumerate-then-argmax execution, multi-query decomposition, a half-implemented conversational feedback loop, and an OpenAI-compatible endpoint — with every answer traceable to specific patterns, specific atoms, and specific rules. That's not a common artifact in 2026. Most interpretable-NLP work stops at toy examples; QuantaR1 is a real system with real coverage of real query shapes.

**The lineage and the organs.** Quanta isn't one project, it's the distilled endpoint of nine-plus named ancestor experiments, and those ancestors didn't all die when the main line converged. hybrid2's unconscious simulator lives in i-total. catLM's fuzzy matching lives in Mnt-Matcher. Paralaxia's fragment-first ideas live in QuantaR1. Intentia's categories live in QuantaR1. The WordNet-isn't-enough lesson is the reason the atom store exists at all. This is a research lab's story, not a single project's story — the program produced reusable ideas, and those ideas are distributed across multiple live systems.

**The discipline.** Four cuts, each the same move, each refusing the invisible-runway trap. This is, honestly, the part of the program that would be hardest for another researcher to replicate, and therefore the part most worth showing. The technical content can be re-derived. The decision-making pattern — stopping moonshots before they eat you, stopping rewrites that can't be intermediately tested, shipping the distilled version instead of the grand one — is rare, and it's the reason QuantaR1 exists as something you can point at instead of as a GitHub graveyard.

## Timeline

![The three phases of the Quanta research program, with the four cuts marked. Breadth-phase dates are approximate (folder mtimes were overwritten during the archaeology that produced this writeup); depth and delivery dates come from folder mtimes and are accurate to the day. QuantaR1 is the current running version and extends to the present.](timeline.png)

## Footnotes from the archaeology

- **stn1 (Sep 30, 2025)** sits right at the Paralaxia → Quanta hinge. It's in a currently-broken state. May be fixed and mined for lessons later.
- The **summy failed-branches** (`summy_failed_bnfparser2`, `summy_failed-patternext`, `summy_failed_prob_bnfparser`) are visible in the directory as explicit dead-end folders. The public artifact count of the program understates the actual exploration cost; every experiment had its own sub-lineage of tried-and-abandoned directions, and summy was unusual only in keeping them visible.
- **quanta_mmc** — a tri-directionally guided Markov chain text generator, prompted with positional anchors like *"cat@12, walk@8, dog@15"* — ran in October 2025 as a parallel side experiment to Quanta1. Not part of the main line, but worth mentioning as evidence that the breadth-first instinct never entirely shuts off.

---

*QuantaR1 is part of Sigmantics AB's ongoing work on interpretable, auditable AI systems, alongside AssetForge (multi-agent orchestration), Mnt-Matcher (fuzzy matching), VecScar (file-based embeddings), i-total (multi-agent discussion), and the Murray Ledger / AssetLedger trust-scoring lineage.*

## Related open-source work

Several of the ancestor organs described above have been extracted into standalone, published components:

- **Mnt-Matcher** — fuzzy matching library descended from catLM's matching substrate.
  https://www.npmjs.com/package/@semantictools/mnt-matcher

- **LLaMiga** — multi-provider LLM orchestration library that grew out of the Scholar / Timmy experiment. The harvesting loop didn't land in 2025, but the plumbing built to support it became a general-purpose library.
  https://www.npmjs.com/package/@semantictools/llamiga

- **i-total** — multi-agent discussion platform that can be run with a version of hybrid2 wired in for creative inserts. This is where hybrid2's unconscious-simulator component lives on.
  https://github.com/SemanticTools/i-total
