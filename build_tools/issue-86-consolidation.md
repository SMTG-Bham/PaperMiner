# Issue #86 consolidation record

[Issue #86](https://github.com/SMTG-Bham/PaperMinerToolkit/issues/86) is addressed
on `codex/issue-86-consolidation`, branched from `dev` at `dc2f505`. The commits
are local review stages; nothing has been pushed. This record distinguishes
completed consolidation from differences that need a separate policy decision.

## Scope and review method

The audit covered all first-party package modules, tests and helpers, CLI and
configuration code, browser validation JavaScript/HTML, Gaudi scripts, recipes,
six notebooks (26 code cells), documentation examples, all five Gaudi shell/batch
scripts, build configuration and CI. At the
baseline this included 49 package modules and 54 test files plus helpers. No
first-party source was inaccessible. Generated outputs, vendored/external
libraries, binary fixture internals and Git history outside the checked-out
source were excluded. Notebook and cluster code was inspected, not executed on
live services or accelerator hardware.

Each extraction preserves its callers' input coercion, output shape, ordering,
exceptions and side effects. New tests focus on failure boundaries and scientific
contracts. Independent expected values, frozen legacy schemas, provider fixture
records and numerical reference assertions remain independent of production
helpers. Thin public/provider wrappers remain where they are compatibility or
runtime patching boundaries.

## Completed consolidation, in review order

Paths below are relative to the repository root. A shared implementation may
have more documentation than the deleted blocks; reducing separately maintained
behaviour is the goal.

| Responsibility | Canonical implementation and changed callers | Contracts preserved and evidence | Risk |
| --- | --- | --- | --- |
| Pipeline state changes | `paperminertoolkit/corpus/database.py:set_pipeline_status`; aliases in `paperminertoolkit/extraction/scrape.py` and `paperminertoolkit/workflows/download.py` | Unknown columns raise before mutation; supplied errors win; successful/stored states clear prior errors. `tests/corpus/test_status.py`. | Low |
| Registry target loading | `paperminertoolkit/providers/registry.py:_resolve_callable`; `resolve_handler`, `resolve_probe`, `resolve_reachability` | Missing capability and optional target handling stay local; target attributes are resolved on each call, preserving monkeypatches; malformed-target wording retained. `tests/workflows/test_sources.py`. | Low |
| Latest preprint revision merging | `paperminertoolkit/providers/rxiv.py:collapse_versions`; Rxiv and chemRxiv `latest_versions` | Provider grouping keys remain distinct; latest input wins ties, falsey fields do not erase prior values, earliest posting date and first-appearance order survive; input mappings are copied. Provider regression tests. | Low |
| OpenAlex cached content URLs | `paperminertoolkit/providers/openalex.py:_content_url`; `cached_pdf_url`, `grobid_xml_url` | Explicit URLs override availability; absent/malformed metadata and missing work ID return empty strings; PDF/TEI suffixes stay distinct. OpenAlex tests. | Low |
| Search-result row construction | `paperminertoolkit/workflows/search.py:_paper_rows`; CORE, OpenAlex, PubMed, arXiv and Rxiv wrappers | Raw provider mapping stays local; fixed DataFrame columns, generator inputs and empty frames retained. OpenAlex abstract reconstructed once. `tests/workflows/test_search_rows.py`. | Low |
| Filter write lifecycle | `paperminertoolkit/corpus/filtering.py:_apply_filter`, `_replace_filter_results`, `_normalize_join_operator`; regex/topic apply and topic refresh | Evaluation algorithms and validation order remain distinct. Definition, result, stack updates and refresh rollback remain transactional. SQLite failure triggers verify complete rollback, stable IDs, creation time and stack order. | Medium |
| Test doubles and corpus readers | `tests/doubles.py:NullProgress`, existing `FakeResponse`/`FakeSession`; `tests/corpus_helpers.py` | Recording progress doubles remain separate; enrichment's infinite empty-page behaviour is explicit. Existing assertions retained; test snapshots write to temporary review directories. | Low |
| Gaudi provenance checks | `examples/sol_gaudi/check_gaudi_environment.py:main`; both server/scrape sbatch scripts | Same import-free package lookup, namespace handling, missing-package and shadowing errors. Direct execution and Slurm spool paths handled. Offline tests and shell syntax checks. Cluster lifecycle stays separate. | Low; live Slurm unverified |
| Topic diagnostics and artifacts | `paperminertoolkit/workflows/topics.py:_corpus_report`, `_add_vocabulary_report`, `_dominant_topic_metrics`, `_topic_diversity`, `_fingerprint_record`, `_prediction_rows`, `_representative_row`, `_save_training_artifacts` | Shared report thresholds, version-2 hashes, `.12g` probability precision, CSV schemas, UTF-8, missing-vocabulary rows and metadata serialization. Training options built once; timestamp reuses `corpus.database.utc_now`. Original versus refactored streaming, online and batch runs had identical components, model IDs and artifacts except timing/timestamps. | Medium |
| Text extraction stage and token budgets | `paperminertoolkit/extraction/scrape.py:scrape_papers`, `_asset_path`; `paperminertoolkit/extraction/tokenizer.py:request_token_budget` used by scraping, compression and unit conversion | Abstract/text modes are mutually exclusive. Earlier chunk results survive later failures, text still feeds image context, summary/status/error semantics retained. Filename sanitation, bytes and default extensions tested. Same token reserve and minimum budget. | Medium |
| Review-file persistence | `paperminertoolkit/workflows/validation.py:_atomic_json`; `ReviewApp.save`, `_save_input_snapshot` | Compact snapshots versus indented decisions, UTF-8/newline, same-directory atomic replace, temporary cleanup and post-success in-memory updates. Serialization and replacement failures tested. | Low |
| Provider JSON/XML response decoding | `paperminertoolkit/providers/base.py:decode_payload`, `require_mapping`, `decode_xml`; generic request wrappers, PubMed and chemRxiv | Request/auth/pacing remain local; PubMed accepts non-mapping JSON and checks its error envelope; chemRxiv challenge explanations and chained causes retained; mapping copies and blank XML semantics tested. | Low |
| Preprint download and enrichment | `paperminertoolkit/workflows/download.py:_preprint_record`, `_download_rxiv_pdf`, `_download_rxiv_text`, `_preprint_abstract`; `paperminertoolkit/workflows/enrichment.py:_identifier_candidates`, `_preprint_fields`, `_preprint_category_rows`, `_preprint_provenance` | Native identifier/version/DOI distinctions, last candidate wins, sparse/empty mappings, category duplicates/rank gaps, error text, JATS document identity and filesystem errors retained. chemRxiv PDF fallback and keywords remain separate. `tests/workflows/test_preprint_adapters.py`. | Medium |
| Settings and topic CLI options | `paperminertoolkit/settings.py:SETTING_ENV_VARS`, `MODEL_ENV_PREFIXES`, `MODEL_ENV_FIELDS`, `_update_email`; named topic decorators in `paperminertoolkit/cli.py` | Nonempty environment overrides, file validation order, truthy mapping reload, prompts and errors retained. Complete CLI help and option metadata compared before/after; command-specific help remains local. | Low |
| Repeated archive/migration tests | Parameterized legacy identifier migrations and `stub_rxiv_walk` in `tests/corpus/test_corpus.py` and `tests/workflows/test_search.py`; archive contract tests in `tests/providers/test_rxiv.py` | Frozen schema field lists and provider-specific expected identifiers/records stay independent. Only common mechanics are shared; provider-specific cases remain explicit. | Low |
| XML primitives and repeated document parsing | `paperminertoolkit/_xml.py:local_name`, `element_text`; provider aliases and `paperminertoolkit/corpus/xml_layout.py:_parse_xml_root`, `_layout_from_xml_root` | Inline text/tail adjacency retained (`H<sub>2</sub>O` becomes `H2O`), Unicode spacing and namespace handling tested. Normal JATS/Elsevier/TEI layout calls reuse the parsed root; embedded TEI retains its serialize/reparse boundary and malformed-tail rejection. Format-specific figure, table, reference and locator interpretation remain separate. | Low for primitives; medium for parsed-root reuse |
| Anthropic request setup | `paperminertoolkit/extraction/_anthropic.py:request_headers`, `messages_url`; model and tokenizer callers | API version, auth headers, payloads and distinct timeouts retained. Root, versioned and proxy API bases covered. Token counting's duplicated `/v1/v1` was corrected in a separate `fix:` commit; full Messages endpoint URLs are not supported base URLs. | Low; explicit endpoint behaviour fix |
| Abstract text and small metadata overlaps | `paperminertoolkit/providers/base.py:html_plain_text` with search/download coercion left local; `paperminertoolkit/providers/openalex.py:author_names`; `paperminertoolkit/workflows/download.py:_download_text` delegates to `_download_elsevier_text`; dead `_elsevier_string_formatter` removed | Search's falsey-list policy differs from download's presence policy; metadata adapters retain their provider-specific coercion and precedence. No live caller used the deleted formatter. | Low |

## Decisions deferred after inspection

These are not interchangeable implementations. A later change should establish
the contract before removing them; none requires blocking the completed stages.

| Candidate and exact locations | Observed difference | Next decision/verification |
| --- | --- | --- |
| DOI normalization: `paperminertoolkit/corpus/database.py:_clean_doi`, `paperminertoolkit/corpus/metadata.py:clean_doi`, `paperminertoolkit/workflows/validation.py:_doi` | For float NaN, results are `''`, `'nan'`, `''`. `http://dx.doi.org/10.1000/ABC` keeps its URL in the database normalizer but becomes `10.1000/abc` in the other two. An encoded resolver URL with a query is decoded/query-stripped only by metadata cleanup. Punctuation and trailing-slash policies also differ. | Decide persisted identity and merge rules, then audit collisions on representative corpora and design any migration. A replacement today could merge previously distinct stored papers. |
| Complete bibliography mapping: `paperminertoolkit/providers/crossref.py:crossref_work_to_paper`, `_publication_date`; `paperminertoolkit/corpus/metadata.py:crossref_fields`; `paperminertoolkit/workflows/enrichment.py:_crossref_fields`, `_date_parts`, `_openalex_fields` | Crossref `[99, 1, 2]` becomes `99-01-02` in the importer and `0099-01-02` in enrichment date formatting. Dict/Mapping acceptance, author whitespace, HTML and Unicode cleanup differ. Enrichment includes precedence, preservation and retraction/child-row rules absent from imports. | Define date coercion, sparse author and malformed-provider policies; compare real payload fixtures and transaction effects before sharing whole adapters. Exact OpenAlex author formatting was safe to share separately. |
| Credential resolution: provider `configured_api_key`/`configured_email` functions and settings loader | Crossref honors an explicit empty mapping and has anonymous fallback/notice logic; several other providers reload on an empty mapping, fall back to environment and return `None` or `''` differently. NCBI pacing also depends on key presence. | Agree explicit-empty versus omitted semantics, precedence and return types before a universal resolver. Settings environment maps were consolidated without changing these contracts. |
| Browser decision traversal: `paperminertoolkit/resources/validation.js:structureState`, `extraTreeState`, `answer`; `paperminertoolkit/workflows/validation_scoring.py:_score_node`, `_score_extra`, `_decode` | Browser functions produce completion/incorrect state for rendering; Python counts TP/FP/FN/pending and accepts Python-style containers. The browser accepts nonempty extra-status strings, while Python whitelists `correct`/`incorrect`; nested nulls and empty containers also take different branches. The scorer is the authoritative export boundary. | Establish shared cross-language fixtures and a decision-schema contract first. Sharing runtime code would add a new architecture/dependency; no such change is justified by similarity alone. |
| Stored identifier indexing: `paperminertoolkit/workflows/enrichment.py:_pubmed_candidates`, `_arxiv_candidates` | These intentionally inspect only stored IDs/prefixes. Provider resolvers may have broader lookup/fallback behaviour. | Keep the short explicit loops until a reusable stored-only resolver exists; do not introduce accidental HTTP lookup or broaden identifier matching. |

## Similar code intentionally kept separate

- Streaming LDA uses bounded cached sparse batches and `partial_fit`; in-memory
  LDA uses `fit_transform`. Evaluation sampling and representative selection
  differ. At an equal-probability cutoff streaming keeps the first retained
  paper while the in-memory NumPy ranking can choose the later one. Only
  formatting and report arithmetic are shared. No scientific reference
  implementation was replaced by the implementation it verifies.
- PDF text extraction, page rendering, figure cropping and XML/JATS/TEI layout
  parsing operate on different formats, coordinate systems and outputs.
  Shared XML primitives do not collapse these algorithms.
- NER token windows preserve raw character offsets. Generative text splitting,
  unit-conversion requests and image batching have different boundaries and
  output alignment. Only their identical request token-budget arithmetic is shared.
- Lossless stored-asset compression and model-context compression have different
  correctness requirements. Neither replaces the other.
- Provider facades, DOI/PMID/arXiv identifier grammars, request protocols,
  archive start dates, optional credentials, error envelopes and retrieval
  fallback order remain explicit. PMC cloud/eutils retrieval stays separate. The
  legacy Elsevier boolean wrapper now delegates to the structured tuple-returning
  adapter; both still propagate provider errors, with the outer source dispatcher
  responsible for catching them.
- Recipes and notebook workflows remain complete standalone examples. The
  `polymer` (36 fields) and `polymer_db` (86 fields) recipes share 20 identical
  full field specifications, but their record identities, prompts, aliases and
  scientific scope differ. The complete JSON schemas remain public artifacts. Gaudi
  server-only and combined scrape jobs retain their different process lifetimes,
  cleanup and scheduling logic; only the provenance check is shared.
- `docs/requirements.txt` intentionally repeats the five documentation dependency
  constraints in `pyproject.toml`. Read the Docs installs the package with
  `--no-deps`, avoiding accelerator libraries. Keep the small explicit list
  rather than introduce a generator or new installation mechanism.

## Verification and review stages

The initial isolated offline baseline passed 1,283 tests (32 network/slow tests
deselected). The original environment lacked Matplotlib and tried to save review
snapshots outside the writable workspace; an existing temporary Matplotlib
installation and temporary review fixture resolved these environmental issues.
No environment or dependency files were changed.

Validation on Python 3.14:

- Final offline suite: **1,440 passed, 32 deselected**. The remaining 83 warnings
  come from installed Torch/joblib deprecations, as in the baseline.
- Real Headroom compression integration, run separately with model-hub offline
  mode enabled: **1 passed, 13 deselected**.
- Ruff over package, tests and the new Gaudi helper: passed.
- Python compilation of package and examples: passed.
- Shell syntax checks for both changed sbatch scripts: passed.
- Sphinx HTML build with warnings treated as errors: passed.
- Whitespace/diff checks: passed.
- Independent review found and corrected non-native boolean topic mode values:
  integer and NumPy boolean inputs again serialize as native booleans, preserving
  immutable model IDs. Regression tests cover both training modes.
- Real pre/post LDA artifact comparisons covered streaming, online in-memory and
  batch training. XML layout comparisons covered synthetic formats, wrapped TEI,
  Rxiv content, three real fixtures and malformed/tail cases.

Live provider/model network calls, actual Slurm/Gaudi execution and the Python
3.11 CI matrix were not run locally. They remain outside the verified runtime
coverage; their first-party source was inspected.

Package code excluding blank lines, standalone comments and docstrings decreased
from **15,902 to 15,556 lines** (346 fewer). Physical package source increased by
114 lines because the shared interfaces now have their own documentation; this
is not a claim of overall raw file-size reduction. Tests gained failure and
contract coverage while repeated mechanics were removed. The important reduction
is that each listed shared responsibility has one maintained implementation.

### Local commit index

The stages below are committed in this order. The final documentation commit
adds this record and a style-only future-annotations import in the Gaudi helper.

```text
1f7a012 refactor: share pipeline and provider primitives (#86)
759d075 refactor: unify filter persistence transactions (#86)
9317610 test: reuse HTTP progress and corpus helpers (#86)
ae86b7c refactor: share Gaudi environment provenance check (#86)
aaa15e6 refactor: share topic reporting and artifact serialization (#86)
6891586 refactor: share extraction flow and request token budgeting (#86)
b7b26f1 refactor: share atomic review JSON persistence (#86)
c8dfc74 refactor: centralize provider payload decoding (#86)
579a521 refactor: share preprint download and enrichment mechanics (#86)
43c492c refactor: share settings mappings prompts and topic options (#86)
9be9d05 test: parameterize shared migrations and archive paging (#86)
a87ffb6 refactor: share XML text and namespace primitives (#86)
a03f80f refactor: share Anthropic request conventions (#86)
4260413 test: preserve figure materialization filename behavior (#86)
e8a7169 fix: avoid duplicate API version in Anthropic token counts (#86)
8c0a243 refactor: reuse parsed XML roots in document layouts (#86)
8320366 test: consolidate shared preprint archive contracts (#86)
032d7b8 refactor: share proven metadata primitives and flag distinct policies (#86)
a8aa5ba fix: preserve boolean topic mode in model identities (#86)
c97f2f7 test: retain shared provider edge cases in one place (#86)
```

### Follow-up order for deferred candidates

1. Preserve the observed policy differences as cross-path fixtures before any
   broader adapter/credential consolidation. Existing metadata characterization
   tests are the starting point; decide whether each difference is intentional.
2. If DOI identity is unified, specify the canonical policy and collision
   handling, then test a versioned migration of existing corpora before changing
   database callers. Do not silently replace matching rules.
3. Define a shared validation decision schema and cross-language fixtures for
   null/empty containers, unknown statuses, malformed input, mismatched types,
   reused/out-of-range pairs and JSON Pointer escaping. Only then assess whether
   shared runtime execution is worth the additional dependency and architecture.
4. Keep the scientific algorithms, provider protocols, reference expectations,
   full recipes and standalone examples separate unless new evidence establishes
   equivalent behaviour and a simpler common responsibility.
