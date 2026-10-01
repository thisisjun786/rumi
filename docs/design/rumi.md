# RUMI

RUMI (Roots Under My Ideas) is a knowledge keeper. Its promise is **Know where everything you know comes from.** It reads what the user collects, ties every note to where it came from, connects what belongs together, and answers with the passages behind it. This contract fixes what RUMI is, how a vault is laid out, how notes keep their identity, how changes and proposals are recorded, how notebooks, sources, grounded chat, outputs, the digest and research requests behave, how RUMI runs, and how it links to LINA. It is normative: implementations follow it, and any change to it goes through a pull request against this file. Product principles live in the [manifesto](../../MANIFESTO.md); this contract doesn't restate them. Finishing this document doesn't mean any part of RUMI is implemented.

## Scope

This contract covers:

- the product, its voice, and its two surfaces: the RUMI app and the `rumi` CLI
- the vault format and layout, note identity, records and proposals
- sources, notebooks, grounded chat and citations, outputs, asking the whole vault, the digest, and research requests from LINA
- where RUMI runs, its runtime, its model endpoint and its packaging
- the roles of the user, RUMI and LINA in a vault
- RUMI's side of the LINA link, and the installation shapes

It doesn't cover:

- the envelope, the RUMI payload schemas, the supported-combination table and the conformance fixtures; they belong to LINA's [host protocol](https://github.com/thisisjun786/lina/blob/dev/docs/design/host-protocol.md)
- how LINA reads a vault, verifies RUMI records and retracts deleted evidence; that belongs to LINA's [materials and knowledge](https://github.com/thisisjun786/lina/blob/dev/docs/design/materials-and-knowledge.md) and [filesystem](https://github.com/thisisjun786/lina/blob/dev/docs/design/filesystem.md) contracts
- the contents and versioning of the LINA kit; they belong to LINA's [runtime](https://github.com/thisisjun786/lina/blob/dev/docs/design/runtime.md) contract
- contribution, CI, branch and release process; that belongs to [docs/policy](#repository-and-releases)

## Product

RUMI is a sibling of [LINA](https://github.com/thisisjun786/lina) and [SION](https://github.com/thisisjun786/sion), not part of a LINA product family. It is open source, lives in its own repository, keeps its own canon (the vault) and speaks in its own voice. RUMI works without LINA, and LINA works without RUMI. The sibling rules both sides follow are in LINA's [product families](https://github.com/thisisjun786/lina/blob/dev/docs/design/product-families.md) contract.

RUMI is LINA's sibling for knowledge work. As LINA hands coding to a coding agent, it hands research and organizing to RUMI: RUMI does the work inside a notebook and returns a cited brief (see [Research requests](#research-requests)).

RUMI's voice is an editor's, not a companion's. It puts sources first, says when it doesn't know, and shows what is missing. RUMI speaks only in the RUMI app, the `rumi` CLI and the outputs it writes; it never speaks inside LINA's conversation.

RUMI offers:

- **Notebooks:** a scope that gathers sources, grounded chat, notes and outputs for one subject.
- **Grounded chat:** answers drawn only from the sources in scope, with a citation on every supported sentence and unsupported sentences marked by default.
- **Outputs:** text built from a notebook, such as briefings, study guides and reports.
- **Proposals:** links, merges, splits and corrections waiting for the user, each with its reason.
- **Ask the whole vault:** the same grounded chat over every source and note in the vault.
- **Digest:** what is new, what is out of date and what to read next, each with a reason.
- **Research requests:** a question LINA hands over, researched inside a notebook and returned as a cited brief.

There are two layers. A notebook is a strict scope: its chat and outputs never reach outside it unless the user widens the scope. The vault-wide map (links, backlinks, merges, marks) and asking the whole vault span every notebook.

## Vault

A vault is a Git repository of plain Markdown files that the user owns. It is RUMI's only canon. Any editor, Obsidian included, can open it. Everything RUMI keeps lives in the vault's Git history. The only exceptions are the two Git-excluded areas inside `.rumi/` (the index and the local area) and settings kept outside the vault, such as the model credential.

### Layout

| Path | Holds | Written by | Rule |
| --- | --- | --- | --- |
| The user's folders and notes | User notes, in whatever structure the vault already has | The user; RUMI through accepted proposals or opt-in automatic apply; LINA within the scope the user granted | Every write is a commit. A write against a stale revision is rejected. The person's version wins. |
| `notebooks/<name>.md` | One notebook: title, instructions, links to its sources and notes, links to its outputs | The user; RUMI on the user's action or on a LINA research request | A notebook is a scope, not a copy. A source in several notebooks is still one file. |
| `sources/<rumi_id>/` | One source card: `source.md` and, when kept, a copy of the original | RUMI | See [Sources](#sources). |
| `inbox/` | What LINA hands over: meeting notes, answers saved from LINA's conversation, files from LINA's library | LINA, new files only | RUMI files each item into a note or a source card. |
| `.rumi/manifest.json` | Vault format version, protocol version and capability versions | RUMI | LINA reads it to decide whether it supports this vault. |
| `.rumi/inputs/` | Input envelopes from LINA: `focus` (what matters now), `request` (a research request) and `source_deleted` (an original LINA deleted) | LINA | RUMI reads inputs and never edits them. |
| `.rumi/records/` | Records of what RUMI added, merged, split, marked and deleted, and the briefs it returned | RUMI | Append-only. See [Records](#records). |
| `.rumi/proposals/` | Proposals and their reasons | RUMI | See [Proposals](#proposals-and-automatic-apply). |
| `.rumi/index/` | The search index | RUMI | Excluded from Git. Rebuildable at any time. |
| `.rumi/local/` | Local data that is never shared, such as notebook chat history | RUMI | Excluded from Git. Never pushed and never read by LINA. |

`notebooks/`, `sources/`, `inbox/` and `.rumi/` are reserved paths. Every other path belongs to the user's own structure.

### Opening a vault

- RUMI opens an existing folder or Obsidian vault in place. It never moves, renames or reorganizes existing files to fit its layout, including when a reserved path name is already in use.
- When the folder isn't a Git repository, opening it as a vault turns on version history there: RUMI initializes Git in the folder and says so.
- RUMI's first commit in a vault creates `.rumi/manifest.json` and the ignore rule that keeps `.rumi/index/` and `.rumi/local/` out of Git.
- The manifest names the vault format version. RUMI never writes a vault whose format version it doesn't support; it stops and reports. Moving a vault to a newer format is an explicit migration, and the migration is a commit.

### Derived data and secrets

- The index is derived data. Deleting it loses nothing; RUMI rebuilds it from the vault. It is never the only copy of anything.
- Model credentials, endpoint settings and other secrets never enter the vault. A vault is a Git repository that the user may push anywhere.
- RUMI never sends a vault to shared CI runners or external servers. Note text leaves the machine only in requests to the model endpoint the user configured, and to the embedding endpoint when the user turns that adapter on. A Git remote is the user's own choice.

## Identity

A note's identity is the `rumi_id` in its front matter. The identity lives in the file, so it survives renames and moves in any editor.

- RUMI doesn't stamp every note when it opens a vault. A note gets a `rumi_id` when it first enters a notebook or a record. A source card gets its `rumi_id` when it is created, and its directory is named by it.
- Adding a `rumi_id` touches only the note's front matter and is a commit under the stale-revision rule. It is the only write RUMI makes to a user's note without a proposal.
- A `rumi_id` is opaque and never reused, including after its note is deleted.
- When a file copy produces two files with the same `rumi_id`, the file that gained that `rumi_id` later in Git history is the copy, and RUMI gives it a new `rumi_id` in a commit.
- A note without a `rumi_id` is addressed by its path at a revision. RUMI never treats an equal name, an equal path, equal content or Git's rename detection as proof that two files are the same note.
- A merge keeps the surviving note's `rumi_id` and lists the merged note's `rumi_id` in the survivor, so links and outside references resolve to it. A split keeps the original `rumi_id` on the part that stays at the original path and gives new ids to the other parts.

LINA records `rumi_id` as a declared external key of a vault file; how LINA uses it is defined in LINA's [filesystem](https://github.com/thisisjun786/lina/blob/dev/docs/design/filesystem.md) contract.

## Roles and writes

No writer owns the vault exclusively. The user, RUMI and LINA may all edit notes. Three rules hold for every writer:

1. **Every change is a commit.** Commits use the vault's Git author settings, which are the user's. RUMI and LINA each tell their own commits apart by the commit hashes in their own records.
2. **Stale-revision writes are rejected.** A writer states the revision it read. If a file it writes differs from that revision at the current head, or has uncommitted edits in the working tree, the write is rejected. RUMI recomputes the change from the current revision, or turns it back into a proposal; it never retries the same write blindly.
3. **The person wins.** RUMI and LINA never overwrite, revert or reformat a person's edit, committed or not. A file with uncommitted edits belongs to the person until they are committed. RUMI never commits edits another writer left uncommitted, and each RUMI commit contains only the files RUMI changed.

| Writer | Writes | Never writes |
| --- | --- | --- |
| The user | Anything in the vault | — |
| RUMI | `sources/`, `.rumi/` except `.rumi/inputs/`, filed `inbox/` items, notebooks, outputs and saved answers on the user's action or on a LINA research request, and notes through accepted proposals or opt-in automatic apply | The user's own text outside an accepted proposal or automatic apply; `.rumi/inputs/` |
| LINA | New files in `inbox/`, envelopes in `.rumi/inputs/`, and edits to existing notes within the scope the user granted LINA | `notebooks/`, `sources/`, `.rumi/manifest.json`, `.rumi/records/`, `.rumi/proposals/`, `.rumi/index/` |

Understanding is a proposal until the user accepts it. RUMI writes its own text (summaries, key points, outputs, saved answers) as its own notes or source cards, and never rewrites the user's words to sound like its own.

## Records

A record states one change RUMI made or observed, and why. RUMI writes records to `.rumi/records/`, one envelope per record, in the shared envelope format defined by LINA's [host protocol](https://github.com/thisisjun786/lina/blob/dev/docs/design/host-protocol.md). Records are append-only: RUMI never edits or removes a committed record.

| Kind | Written when |
| --- | --- |
| add | RUMI files an item, such as an `inbox/` item or a new source, into a note, a source card or a notebook. |
| merge | A merge is applied. Both histories stay in Git, and the record names both `rumi_id` values. |
| split | A split is applied. The record names the original `rumi_id` and every new one. |
| mark | RUMI marks a note: stale, contradicted by a newer source, original updated, original unreachable, or original deleted. |
| delete | A commit by any writer removes a note that has a `rumi_id`, or a source card. |
| brief | RUMI finishes a LINA research request. See [Research requests](#research-requests). |

Every record names the commit that carries the change, the affected `rumi_id` values and paths, and the reason. A record is written only after its commit exists, so a reader can verify the commit before trusting the record.

A mark lives only in `.rumi/`. It never changes a note's body or front matter; assigning a `rumi_id` is the only change RUMI makes to front matter on its own (see [Identity](#identity)). RUMI shows marks next to the note and, for stale or contradicting notes, next to the newer source. The user may dismiss a mark; the dismissal is committed in `.rumi/`.

## Proposals and automatic apply

RUMI proposes before it rewrites. Each proposal in `.rumi/proposals/` holds its kind (link, merge, split or correction), the notes it touches, the reason, the passages the reason rests on, the change it would make, and the revision it was computed against.

- Accepting a proposal applies its change as one commit, writes its record, and marks the proposal accepted. If a target differs from the proposal's revision, RUMI recomputes the proposal instead of applying it.
- Rejecting a proposal keeps it with its rejection, so RUMI doesn't raise the same proposal again unless its inputs differ.
- Automatic apply is off by default and is turned on per vault by the user. An automatically applied change follows the same commit, stale-revision and record rules, and is reversible like any commit.

## Sources

A source is anything RUMI reads as evidence: a file, a web page, a note from another tool, or an item from LINA's library. Each source has a card at `sources/<rumi_id>/source.md`.

- Front matter holds the origin (URL, path, or LINA library asset with its revision), the capture time and the content hash.
- The body holds the extracted text with page and section markers, so every passage has a stable anchor.
- A web source is a snapshot taken at capture time. A file source keeps a copy of the original next to the card; a large binary is referenced by content hash and original location instead of being copied.
- RUMI shows each source's status: current, original updated, original unreachable, or original deleted. When RUMI sees that an original changed, it refreshes the source card from it automatically: one commit holds the new snapshot, and a mark record (`original updated`) names the notes that cite the earlier one. RUMI never overwrites a snapshot without a commit and a record, and older passage anchors stay resolvable through Git history.
- A note used as a source is a secondary source. Its citations are followed back to the original passages, and answers show it as secondary.

Parsing, passage anchors and citation checks come from the LINA kit, so the same passage of the same document has the same address in RUMI and in LINA. Text, Markdown, HTML and PDF are read first. Images, Office documents and audio follow through the same kit boundary. Parsers that need another runtime or license run as pinned separate processes and are recorded in the release manifest.

Sources enter a notebook in four ways: chosen from the vault, dropped in as files or folders, added by URL, or chosen from LINA's library when LINA is connected. A library item arrives as a new file in `inbox/`, and RUMI turns it into a source card.

## Notebooks

A notebook is the file `notebooks/<name>.md`. It holds the title, the instructions for chat and outputs in this notebook, links to its sources and notes, and links to its outputs.

- A notebook is a scope, never a copy. Adding a source to a notebook adds a link. Removing a notebook removes the scope, not its sources or notes.
- Notes are plain Markdown with citation links. They stay editable in any editor at any time.
- The notebook file is part of the vault format, so LINA can attach the same notebook to a conversation by reading the same file.

## Grounded chat and citations

Grounded chat runs in a notebook, or over the whole vault when the user asks the whole vault. Chat history is kept in `.rumi/local/`, outside Git; only what the user saves enters the vault. Citations act as a trust signal even when nobody opens them, so RUMI marks what is unsupported up front and keeps those marks wherever an answer goes, instead of relying on the user to check citations.

1. **Visible scope.** The composer shows the scope, such as "3 of 12 sources". Answers use only sources in scope. Widening the scope is an explicit user action.
2. **Sentence-level citations.** Every supported sentence ends with a chip naming its source. Hovering shows the cited passage; opening it shows the original at that passage, highlighted.
3. **Unsupported sentences marked by default.** RUMI checks support sentence by sentence with the kit's citation check. A sentence with no supporting passage in scope is marked as unsupported.
4. **Support, not confidence.** The mark says whether a passage supports the sentence. RUMI doesn't highlight words by model probability.
5. **Not found is an answer.** When the sources in scope don't answer the question, RUMI says it didn't find the answer in the selected sources and offers three next actions: widen the scope, add a source, or ask the whole vault. It never presents model knowledge as sourced.
6. **Marks travel with the text.** Saved, exported and handed-over text keeps its unsupported marks, and the save or export shows how many there are.
7. **Saved answers stay grounded.** An answer enters the vault only when the user saves it as a note or an output. Its citations stay links, and the note stays editable.
8. **Deleted evidence stays visible as deleted.** When a cited passage's source is deleted, the citation shows a deleted source; the text that cited it is kept and counts as unsupported.

Asking the whole vault follows the same rules with the whole vault as scope, and each answer names the notebooks and notes it drew from.

## Outputs

Outputs are text built from a notebook's scope, such as briefings, study guides and reports. An output is an ordinary note: plain Markdown with citation links, linked from the notebook file, editable by the user. Outputs follow the grounded-chat rules for citations and unsupported marks. Regenerating an output is a new commit under the stale-revision rule, so it never overwrites the user's edits to it.

## Digest

The digest is a short list of what is new, what is out of date, and what to read next. Every item has a reason and links to its notes and sources. The digest is an output of the vault and is written to the vault as a note.

When LINA is connected, the latest `focus` input in `.rumi/inputs/` raises the priority of related items. Without LINA, RUMI orders the digest from the vault alone. LINA shows the latest digest in its today feed as a card labeled RUMI; that card is defined in LINA's [surfaces](https://github.com/thisisjun786/lina/blob/dev/docs/design/surfaces.md) contract.

## Research requests

RUMI does the knowledge work that LINA hands over. A research request is a `request` input in `.rumi/inputs/` with a question and a target notebook.

1. RUMI opens the target notebook, or creates it when the request names a new one.
2. RUMI gathers sources for the question into the notebook as source cards, from the vault and from the web, reads them, and organizes what it finds into notes under the rules for sources, notes and proposals.
3. RUMI writes the brief as an output of the notebook and returns it to LINA as a `brief` record.

A brief is a short answer to the question, grounded in the notebook:

- It follows the grounded-chat rules: every supported sentence cites a passage, unsupported sentences are marked, and when the sources don't answer the question, the brief says so.
- The `brief` record names the request it answers, the notebook and the commit that holds the brief output. It carries the brief text and, for each citation, the source card's `rumi_id`, the revision, the passage anchor and the cited passage.
- The sources stay in the notebook. LINA receives only the brief and its cited passages.

The notebook belongs to the user like any other. The user can open it in the RUMI app and continue the research there, with or without LINA.

## The RUMI app and the `rumi` CLI

The RUMI engine does all of RUMI's work. The `rumi` CLI and the RUMI app are its two surfaces.

### Engine and CLI

- `rumi` is the terminal interface to the engine. It opens or initializes a vault, adds sources, asks a notebook or the whole vault, reviews proposals, shows the digest, rebuilds the index, and runs the engine for the app with `rumi serve`.
- One engine writes a vault at a time. While `rumi serve` runs, CLI commands that write go through it.

### RUMI app

- The RUMI app is a desktop app for Linux, macOS and Windows. It is a thin client: it shows and asks, and every write goes through the engine.
- It talks to `rumi serve` over a local socket. The messages are defined in JSON Schema, and the app's Dart types are generated from it.
- First run opens a vault: the user picks an existing folder or Obsidian vault, or creates a new one, and enters the model endpoint, which is set automatically on LINA OS. The app then shows what this vault can and can't do, and its header shows whether LINA is connected.
- The main screens are the notebook home, the add-source sheet with each source's status, the notebook view in three panes (sources, answer, notes and outputs), proposals, and the digest.
- The RUMI app is one Flutter codebase built natively for each platform, as the LINA app is. It never ships as a Flutter web build, a web-technology shell or a WebView shell, and it meets the same rules defined in LINA's [surfaces](https://github.com/thisisjun786/lina/blob/dev/docs/design/surfaces.md) contract, including platform adaptation, Korean input and accessibility.
- The RUMI app shares the LINA app's design language and design tokens: terms, flows, the visual language, icons, and widgets such as the citation chip. On mobile, the user reaches notebooks through the LINA app.

## Runtime

### Runs on the user's machine

RUMI runs as a local process on the user's own machine, with the vault on that machine. On LINA OS, LINA OS may offer to install RUMI as the user's own tool; LINA does not pin or ship a RUMI release. Once installed, RUMI runs as a separate process and is turned on when the user creates or connects a vault. There, its background work counts as background load in LINA's [non-competition](https://github.com/thisisjun786/lina/blob/dev/docs/design/non-competition.md) measurements.

### Go with the LINA kit

The RUMI engine and the `rumi` CLI are written in Go and build to a static binary per platform. They build on the LINA kit, the stateless shared Go module defined in LINA's [runtime](https://github.com/thisisjun786/lina/blob/dev/docs/design/runtime.md) contract. RUMI uses the kit's Responses adapter, loop, tool executor and sandbox wrapper, document parsing, passage anchors, citation check, and sibling protocol types.

- RUMI vendors the kit as source from one LINA release tag; LINA publishes no separate release or module of the kit. RUMI records the kit version and the LINA release tag in its release manifest.
- RUMI follows LINA's [dependency policy](https://github.com/thisisjun786/lina/blob/dev/docs/policy/dependencies.md) unchanged: Go modules with `go.mod` and `go.sum` committed, exact versions on a pinned Go toolchain, checksum database verification, `govulncheck`, and vendored code kept unchanged with RUMI's changes in wrapping layers. Its CI enforces the policy in `foundation` from the first dependency.
- The kit has no default state path, persona, skills or database writer. RUMI passes its own vault, instructions and storage, and builds the vault, proposals, notebooks and records on top of the kit.
- RUMI never carries LINA's persona, skills or memory.
- Tools in RUMI's loop run through the kit's tool executor inside its sandbox. No tool writes the vault directly; every write goes through RUMI's commit path and its stale-revision check.
- Search runs on the local index and needs no service other than the model endpoint. Embeddings are optional: when the user turns on the embedding adapter, a separate non-chat adapter to an embedding endpoint the user configures, the index adds vectors. Without it, search uses full-text search alone.

### Model endpoint

- RUMI calls exactly one OpenAI Responses-compatible endpoint for chat, configured by the user. On LINA OS, that endpoint is opencodex. The optional embedding adapter is the only other model connection.
- Calls go through the kit's Responses adapter with `store=false`, sending encrypted reasoning back on following turns.
- When the endpoint is unreachable, features that need the model say so and why. Reading the vault, browsing sources, searching, reviewing proposals and following records keep working.

### Packaging

The `rumi` CLI, which also runs the engine as `rumi serve`, ships as a static binary per platform, built with the pinned Go toolchain, and the RUMI app ships with that binary. The release manifest records each artifact by version and digest.

## LINA link

RUMI and LINA never call each other's APIs at runtime. The vault is the only point of contact, and each side keeps working when the other is absent or the link is off. RUMI without LINA is not a reduced product, and LINA without RUMI keeps every function it has.

### The vault as LINA's connected source

LINA reads the vault as a connected source. It doesn't copy the vault; the user's act of connecting it is the approval; LINA uses vault commits as revisions; and LINA keeps its derived data in its own state, outside the vault. These rules are LINA's and are defined in LINA's [filesystem](https://github.com/thisisjun786/lina/blob/dev/docs/design/filesystem.md) and [materials and knowledge](https://github.com/thisisjun786/lina/blob/dev/docs/design/materials-and-knowledge.md) contracts.

### LINA to RUMI: the mailbox

- **`inbox/`:** LINA writes new files only, each as a commit: meeting notes, answers saved from LINA's conversation, and files from LINA's library. Front matter names the origin conversation or meeting, the target notebook, the citation anchors the item carries and, for an item from LINA's library, its LINA asset id and revision. RUMI files each item into a note or a source card in the target notebook, moves it out of `inbox/` in the same commit, and writes an add record.
- **`.rumi/inputs/`:** LINA writes input envelopes of three kinds: `focus`, what matters now (goals, projects and work in progress), which RUMI uses to order the digest; `request`, a research request that RUMI answers with a brief (see [Research requests](#research-requests)); and `source_deleted`, a notice that LINA permanently deleted an original, such as a meeting recording. On a `source_deleted` input, RUMI marks the notes built from that original as original deleted.
- **Note edits:** LINA may edit existing notes within the scope the user granted, under the rules in [Roles and writes](#roles-and-writes).

A meeting note filed into the vault belongs to the vault. Deleting it in LINA doesn't remove the vault's copy.

### RUMI to LINA: records

LINA reads `.rumi/records/` as sibling records: it accepts a record only after verifying that its commit exists in the vault at the stated revision, and then only as sourced external evidence. A record is never a receipt, an approval or an instruction. A delete record lets LINA retract that evidence from its answers. A brief record gives LINA the brief and its cited passages, which LINA shows as a RUMI card with "Open in RUMI". These rules are defined in LINA's [main authority](https://github.com/thisisjun786/lina/blob/dev/docs/design/main-authority.md) and [materials and knowledge](https://github.com/thisisjun786/lina/blob/dev/docs/design/materials-and-knowledge.md) contracts.

### Envelope, versions and conformance

- Every input and every record uses the shared envelope. The envelope and the RUMI payloads are defined in JSON Schema in LINA's [host protocol](https://github.com/thisisjun786/lina/blob/dev/docs/design/host-protocol.md); RUMI uses the Go types the LINA kit generates from it.
- `.rumi/manifest.json` declares the vault format version, the protocol version and the capability versions. The vault format (notebook files, source cards and RUMI's front matter keys) is part of the protocol. LINA accepts only combinations listed in its supported-combination table and refuses and reports any other.
- RUMI leaves an input with an unsupported envelope version unprocessed and reports it in the app and the CLI.
- RUMI CI runs LINA's conformance fixtures for every protocol version RUMI declares. A RUMI release declares only versions whose fixtures pass.

### Speaker boundary and handoffs

In LINA's conversation only LINA speaks. RUMI's results, such as briefs, the digest and add records, appear there only as cards labeled RUMI. From LINA's conversation, the user can attach a notebook, save an answer to a notebook, and open a passage in RUMI. LINA answers over an attached notebook with its own engine, so it works when RUMI isn't running; both use the kit, so passage selection and citation format match. These handoffs are defined in LINA's [surfaces](https://github.com/thisisjun786/lina/blob/dev/docs/design/surfaces.md) and [materials and knowledge](https://github.com/thisisjun786/lina/blob/dev/docs/design/materials-and-knowledge.md) contracts. Opening in RUMI is a link the operating system opens in the RUMI app, not an API call.

## Installation shapes

The same vault format works in every shape.

| Shape | Where RUMI runs | Model endpoint | How LINA reaches the vault | What the LINA link adds |
| --- | --- | --- | --- | --- |
| RUMI alone | The `rumi` CLI and the RUMI app on the user's machine | The Responses-compatible endpoint the user configures | Not connected | Nothing; every RUMI function works |
| Desktop with LINA | Locally on that machine | The Responses-compatible endpoint the user configures | Through that device's Node, as a connected folder within its grant | The vault in LINA's library, notebooks attached to conversations, meeting notes and saved answers filed through `inbox/`, `focus` inputs, research requests answered with briefs, records read by LINA |
| LINA OS | A separate process, installed as the user's own tool when LINA OS offers it | opencodex | As a local connected source | The same |

## Repository and releases

RUMI lives in [thisisjun786/rumi](https://github.com/thisisjun786/rumi) under the [MIT License](../../LICENSE). It follows the same contribution, CI, branch and release policies as LINA and SION: the [contribution guide](../../CONTRIBUTING.md), [issues](../policy/issues.md), [pull requests](../policy/pull-requests.md), [CI](../policy/ci.md) with one required check, `foundation`, and [releases](../policy/releases.md) from `dev` with `main` as the release mirror and immutable `vX.Y.Z` tags. Issue #1 is the [roadmap](https://github.com/thisisjun786/rumi/issues/1).

RUMI keeps three versions apart: its product version, the protocol version (which includes the vault format version), and the LINA kit version. Each release manifest records all three, the LINA release tag the kit came from, the Go toolchain that built the binaries, and every pinned external component.

## Deferred

- Exact front matter keys, file names inside a source card, the manifest schema, the citation and unsupported-mark syntax in saved Markdown, and the default location of outputs and digests: set by the vault format version 1 specification in the vault stage.
- The `rumi_id` format: set by the vault stage.
- Handling of an existing vault whose folders already use a reserved path name: set by the vault stage's acceptance on real vaults.
- The size above which a source's original is referenced by hash instead of copied: set by measurement in the understanding stage.
- Index storage and retrieval quality bounds: set by measurement in the vault and understanding stages.
- Digest cadence and presentation: set during digest implementation acceptance.
- How the RUMI app takes the design tokens and shared widgets from LINA: set before the first RUMI app GUI work.
- The local socket transport and message schema between the RUMI app and `rumi serve`, and app screen layout: set during RUMI app implementation acceptance.
- Package formats and install locations of the `rumi` CLI and the RUMI app on each desktop OS: set by the RUMI app stage's packaging acceptance.
- Background-load bounds for RUMI on LINA OS: declared per test in LINA's non-competition measurements.
