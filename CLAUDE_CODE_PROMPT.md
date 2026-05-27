# Claude Code Prompt — Hackathon Starter Guide Website

Paste this entire file into Claude Code. It is fully self-contained — all guide
content is embedded below, so no extra files are needed.

---

Build a single-page static website that presents starter guides for a team
hackathon. It will be shared with the hackathon group, so it should look clean,
professional, and load instantly.

## What to build

A single `index.html` file with inline CSS and JS — no framework, no build
tooling, no external dependencies, no `fetch()`. It must work by double-clicking
the file. All content is provided in this prompt; embed it directly into the
HTML as you build, converting the Markdown to HTML yourself.

### Layout

- Header: title "Hackathon Starter Guides", subtitle "Five teams, five projects
  — hints for tackling them with Glean, Gemini & Zapier".
- A horizontal **tab bar** with one tab per team; tab label = the project name.
- Below the tabs, a content panel showing the active guide. Clicking a tab
  switches the visible guide. Default to the first tab on load.
- Use the URL hash (e.g. `#brief-agent`) so a tab is linkable and survives
  refresh. The hash slug for each guide is given as its section id below.

### Content panel, per guide

- Project title as a prominent heading.
- The one-line problem statement styled as a callout/lead paragraph.
- The team members shown as a small pill/badge row.
- Remaining sections rendered with clear headings, readable body text, and
  properly formatted bullet lists. Preserve all Markdown links as real anchors.

### Design

- Modern, calm, professional. Generous white space, ~680px max content width,
  comfortable line height.
- One accent color (a confident blue or indigo) for the active tab and section
  headings.
- System font stack. No icon libraries. Subtle touches only — a light
  background on the callout, a clear active-tab indicator, a simple tab switch.
- Fully responsive: the tab bar wraps or scrolls horizontally on narrow screens
  and stays tappable.
- Short footer line: "Internal hackathon resource".

### Quality bar

- Semantic, accessible tabs (`role="tab"` / `role="tabpanel"`, arrow-key
  navigation, visible focus states).
- No console errors. Valid, self-contained HTML.

Produce the final result as a single `index.html`.

---

# GUIDE CONTENT

Each guide below is one tab. The heading directly under `## TAB:` gives the
hash slug to use for that tab's section id.


## TAB: `brief-agent`

# Brief Agent / Generator

**The problem in one line:** Turn messy project context — rough notes, voice memos, half-formed asks — into a polished, Notion-ready creative brief through a guided conversation.

**Team:** Claire, Dana, Meg, Michelle Griffin

---

## How to think about it

A brief is a *structured extraction* problem. The information already exists in someone's head or scattered notes; the job is to pull it into a fixed shape (goal, audience, message, deliverables, timing, success metrics) and flag what's missing. The hard part isn't generation — it's asking the right follow-up questions and knowing when an answer is too vague to be useful.

## A suggested architecture

- **Gemini as the interviewer.** Give it the target brief schema as a system prompt and have it run a guided Q&A: ask one question at a time, react to answers, ask smart follow-ups. Gemini's long context lets you paste in a voice-memo transcript or a wall of Slack messages and have it extract everything answerable before it even asks a question — so the human only fills gaps.
- **Glean for the "pull in past examples and brand guidelines" requirement.** Index your past briefs, brand guide, and messaging docs in Glean. When the agent has a draft goal/audience, query Glean for similar past briefs and feed the top matches back into Gemini as few-shot examples. This is what makes the output sound like *your* briefs, not a generic template.
- **Zapier for delivery.** Final step: a Zap that takes the structured brief and creates a formatted Notion page. Notion has a native Zapier integration — map each brief section to a Notion block or database property.

## Concrete first steps

1. Write the brief schema down explicitly — every field, plus a one-line "what good looks like" for each. This single doc is the backbone of the whole agent.
2. Build the Gemini interview loop with a hardcoded sample of messy input. Get the conversation feeling right before adding any retrieval.
3. Add the "vague ask" detector: have Gemini score each filled field as specific / vague / missing and refuse to finalize until vague fields are addressed.
4. Layer in Glean retrieval for past examples once the core loop works.
5. Wire the Notion output Zap last.

## Hints & gotchas

- Resist the urge to ask all questions upfront. The magic is *adaptive* follow-ups — a good answer to "who's the audience" should change the next question.
- "Prescriptive vs descriptive" balance: have the agent explicitly ask the requester how much creative latitude the writer should have, and capture it as a field.
- Voice memo support is mostly free — Gemini handles audio natively, or transcribe first. Don't over-build this.
- Demo tip: show the same messy input going in and a clean Notion brief coming out. The contrast sells it.

---

## TAB: `community-update-agent`

# Community Update Agent

**The problem in one line:** Watch the `#typeform-release` Slack channel for product updates, then find community conversations where that news should be shared — keeping the community current and re-establishing Typeform as the source of truth.

**Team:** Grace, James, Phoebe, Suzie

---

## How to think about it

This is a *matching* problem with two halves: (1) detect a new release and understand what it actually changes, and (2) find community threads where that release answers an open question or corrects stale info. The second half is the interesting one — it's semantic search, not keyword matching. Someone asking "is there a way to do X" should match a release that adds X even if they never use the release's name.

## A suggested architecture

- **Zapier as the trigger.** A Zap on the Slack "new message in `#typeform-release`" event kicks off the whole flow. This is the cleanest possible entry point — no polling, no cron.
- **Gemini to parse the release.** Slack release notes are written for internal readers. Have Gemini summarize each update into a structured object: what changed, who it affects, what user problems it now solves, and a few paraphrased "questions this would answer." Those paraphrased questions become your search queries.
- **Glean to find the conversations.** If your community platform is indexed in Glean, query it with the paraphrased questions to surface relevant threads. Glean's semantic ranking is what catches the "asked it differently" cases. If the community *isn't* in Glean, that's your first scoping decision — see gotchas.
- **Zapier to close the loop.** Post the matches back into a review channel: "Release X looks relevant to these 3 threads — [links]." Keep a human in the loop before anything is posted publicly.

## Concrete first steps

1. Pull 5–10 real past releases and, by hand, find the community threads each *should* have been shared in. This is your ground truth — you'll test everything against it.
2. Build the Gemini release-parser and check that its paraphrased questions actually resemble how customers ask.
3. Test retrieval against your hand-labeled set. Measure: of the threads you found manually, how many does the agent surface?
4. Wire the Slack trigger and the review-channel output.

## Hints & gotchas

- **Confirm the community is searchable first.** If your community platform isn't a Glean connector, you may need its API or a scraper — settle this in hour one, it changes the whole build.
- Keep a human approval step. Auto-posting into community threads is a reputation risk; the agent should *recommend*, a person should *post*.
- Stale matches are a real failure mode — an old thread may already be resolved. Have Gemini check thread recency/status before recommending.
- Demo tip: take a genuine recent release and show the agent surfacing a real thread nobody had connected to it.

---

## TAB: `content-engineering-system`

# Content Engineering System

**The problem in one line:** A centralized source of truth that dynamically feeds AI your brand guide, messaging docs, and content-type best practices — so contractors, agencies, and internal writers all work from the same inputs instead of material scattered across Figma, Notion, and Slides.

**Team:** Jackie Pecquex, Nina

---

## How to think about it

The reference Ahrefs post frames this well: content engineering is about making your brand context *retrievable by AI on demand*, rather than copy-pasted into prompts by whoever remembers where the doc lives. The deliverable isn't a document — it's a system where "write a blog intro" automatically pulls the right brand voice, the right content-type rules, and the right examples without the writer assembling them.

This is the most infrastructure-flavored of the five projects, which suits a 2-person team: less UI, more plumbing.

## A suggested architecture

- **Glean as the source of truth.** This is the core. Glean already indexes Notion, Slides/Drive, and Figma — so instead of *moving* all the scattered material, you make it *retrievable in place*. The win is that you don't have to migrate anything; you curate what Glean indexes and how it's tagged.
- **A Glean Agent per content type.** Build agents (or saved prompt scaffolds) for exec comms, blog, social, etc. Each one knows which messaging docs and best-practice guides are authoritative for that content type, and retrieves them on demand.
- **Gemini as the generation layer.** When a writer asks for a draft, Gemini receives the retrieved brand context + content-type rules as grounding. Long context means you can stuff in the full brand guide rather than a summary.
- **Zapier for freshness.** A scheduled Zap can check whether key source docs changed and flag the system (or a Slack channel) when the "source of truth" might be drifting from what's indexed.

## Concrete first steps

1. Inventory the scattered material: list every brand guide, messaging doc, and best-practice file, and where it lives. Decide which is *authoritative* where versions conflict — this curation is the real work.
2. Confirm what Glean already indexes and fill the gaps.
3. Build one content-type agent end to end (blog is a good first pick) before generalizing.
4. Define the retrieval contract: given a content-type request, what exactly gets pulled and in what priority order.

## Hints & gotchas

- The hard part is *curation*, not *technology*. Conflicting and outdated docs are the enemy — a system that confidently retrieves the wrong brand guide is worse than no system.
- Don't try to cover all content types in the hackathon. One excellent type beats six mediocre ones.
- Make "who owns this source doc" explicit — a source of truth with no owner goes stale immediately.
- Demo tip: show the same content request answered by a generic model vs. your grounded system, side by side. The on-brand difference is the pitch.

---

## TAB: `customer-journey-360`

# Customer Journey 360

**The problem in one line:** A unified view of everything a single customer receives across Iterable, Salesloft, Zendesk, in-app messages, and other touchpoints — so field teams can prep for calls and QBRs, investigate self-serve behavior, spot duplicate comms, and audit self-serve-to-sales handoffs using real examples instead of theory.

**Team:** Josh Rosenblatt, nico

---

## How to think about it

This is a *data aggregation and timeline* problem. Each tool holds one slice of what a customer experienced; the value is stitching them into a single chronological story for one customer. Start by nailing the timeline view for *one* customer before worrying about scale or polish.

## A suggested architecture

- **Zapier to gather the touchpoints.** Zapier connects to Iterable, Salesloft, Zendesk and more. The cleanest hackathon scope: pick one customer (or a small set), and use Zapier to pull their recent events from each tool into a single store — a Google Sheet is perfectly fine for a hackathon.
- **Gemini to build the narrative.** Feed the merged, timestamped events to Gemini and have it produce: a clean chronological timeline, a plain-English summary ("here's what this customer has experienced in the last 60 days"), and flags for duplicate or conflicting comms. Gemini's long context handles a busy customer's full event history in one pass.
- **Glean for context the event logs don't carry.** Glean can surface related Zendesk tickets, internal Slack discussion, and account notes about that customer — the qualitative layer that explains *why* behind the events.
- **Output:** a prep doc or dashboard a CSM can read before a call. Could be a generated Google Doc (via Zapier) or a simple page.

## Concrete first steps

1. Pick one real customer with a rich, messy history — a good demo subject.
2. Get raw event exports from each of the 5 tools for that customer, by hand if needed. Don't automate collection until you know what the data looks like.
3. Define a common event shape: timestamp, channel, type, summary. Normalizing into this is the core work.
4. Build the Gemini timeline + duplicate-detection step.
5. Automate collection with Zapier last, once the shape is proven.

## Hints & gotchas

- Timestamp normalization across five tools (time zones, formats) is the unglamorous part that will eat time — budget for it.
- "Duplicate comms" detection is a high-value, demoable feature — two tools emailing the same customer the same week is exactly the kind of finding that lands.
- Don't try to integrate all touchpoints live. Three tools, done well, beats five half-wired.
- Demo tip: show a real customer's timeline and point to a genuine duplicate or a rough handoff. Concrete beats abstract.
- Privacy: use a real but internal-friendly account, and don't surface anything sensitive in a shared demo.

---

## TAB: `update-mega-mind`

# Update Mega Mind

**The problem in one line:** A system fed with project tooling, experiment updates, and status updates that can be queried to auto-generate recurring WBR, MBR, and other status decks — eliminating the repetitive deck-building that's currently eating into evenings.

**Team:** Jasmine Howard, Kay-Kay, Xavier Mangin

---

## How to think about it

Two distinct jobs here, and it helps to separate them: (1) *gather and synthesize* what happened this week/month, and (2) *render* it into the recurring deck format. Most of the value and most of the difficulty is in job 1 — pulling scattered updates into an accurate, well-summarized status. Job 2 is templating.

Scope tip: a WBR *content doc* that's accurate and complete is a huge win on its own. A fully formatted slide deck is the stretch goal, not the starting point.

## A suggested architecture

- **Glean to gather the inputs.** Project status, experiment updates, and tooling notes live across Notion, Slack, Jira, docs. Glean indexes those — so the system can query "what changed on Project X this week" rather than you wiring up each source individually.
- **Gemini to synthesize.** Feed Gemini the retrieved updates plus the structure of your WBR/MBR (sections, metrics, narrative style). Have it produce the status writeup section by section. Long context lets you pass a whole week of raw updates at once. Give it last cycle's deck as a style example so the output matches house format.
- **Zapier for cadence and delivery.** A scheduled Zap runs the generation every Monday (WBR) or month-start (MBR), and delivers the result — into a Google Doc, a Slides deck, or a Slack post for review.
- **Slides rendering (stretch):** Zapier can create Google Slides from a template; or generate structured content and have a script populate a Slides template. Treat this as the last mile.

## Concrete first steps

1. Collect 2–3 past WBRs/MBRs and reverse-engineer the template: exact sections, what data each needs, the tone.
2. List every input source a human currently checks to build the deck — that list is your Glean retrieval scope.
3. Build the Gemini synthesis step against *one* past period and compare its output to the deck a human actually produced. Tune until close.
4. Add the scheduled Zapier trigger.
5. Tackle slide formatting only if 1–4 are solid.

## Hints & gotchas

- Accuracy beats formatting every time. A perfectly styled deck with a wrong metric is worse than a plain doc that's correct. Have the system cite where each claim came from.
- The system can't invent updates that weren't written down — part of the pitch is that it gently surfaces "Project Y had no updates this week," which is itself useful signal.
- Don't sink the hackathon into pixel-perfect slides. Get the synthesis excellent; render simply.
- Demo tip: generate this week's actual WBR live and let the team compare it to what they'd have written by hand.

---
