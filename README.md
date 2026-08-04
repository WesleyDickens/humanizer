# Humanizer

[![skills.sh installs](https://skills.sh/b/blader/humanizer)](https://skills.sh/blader/humanizer)

A portable agent skill that removes signs of AI-generated writing from text, making it sound more natural and human. It is plain Markdown, so it can run in any harness that supports skill-style instructions.

## Installation

### Skills CLI

Install globally with the cross-agent skills CLI so Humanizer is available in every project:

```bash
npx skills add blader/humanizer --global
```

Update an existing install:

```bash
npx skills update humanizer --global
```

To install globally into every supported agent harness:

```bash
npx skills add blader/humanizer --global --agent '*'
```

To target one configured harness, pass its agent name:

```bash
npx skills add blader/humanizer --global --agent <agent-name>
```

Omit `--global` for a project-local install that can be committed and shared with collaborators. Start a new agent session or reload skills after installation.

### Claude Code plugin

Claude Code users can also install Humanizer as a plugin:

```
/plugin marketplace add blader/humanizer
/plugin install humanizer@humanizer
```

The skill is then invoked as `/humanizer:humanizer`.

### Manual

Any agent harness can use the skill directly because the runtime artifact is `SKILL.md`. Install it wherever your harness expects skill directories, or copy `SKILL.md` into an existing skill folder.

For example:

```bash
git clone https://github.com/blader/humanizer.git /path/to/your/skills/humanizer
```

Or, if you already have this repo cloned:

```bash
mkdir -p /path/to/your/skills/humanizer
cp SKILL.md /path/to/your/skills/humanizer/
```

## Usage

Invoke the skill however your agent harness exposes installed skills. Common forms include a slash command or a direct request:

```
/humanizer

[paste your text here]
```

```
Please humanize this text: [your text]
```

Point it at a file and the skill rewrites it in place:

```
Humanize the prose in docs/launch-post.md
```

### Voice Calibration

To match your personal writing style, give the skill a sample of your own writing. Set it up once in a file and every later run picks it up automatically:

```bash
mkdir -p ~/.claude/humanizer
$EDITOR ~/.claude/humanizer/voice.md
```

Paste two or three passages of your own prose under a `## Samples` heading. Aim for 150 words minimum, and include more than one register if you write in more than one:

```markdown
## Samples

[a few paragraphs you wrote]

[another passage, different register]
```

The first run analyzes those samples, appends the derived profile to the same file, and reuses it from then on. Edit your samples later and delete the `## Profile` block to force a re-derive.

The skill looks in this order and stops at the first hit:

| Order | Source | Use it for |
|---|---|---|
| **1st** | A sample pasted into the conversation, or a file you point at | One-off calibration, or borrowing someone else's voice |
| **2nd** | `.humanizer-voice.md` in the project root | A per-repo voice, committable and shared with collaborators |
| **3rd** | `~/.claude/humanizer/voice.md` | Your default voice everywhere |
| **none** | Nothing found | Falls back to the generic house style |

A project file overrides your global one, so docs for a given repo can hold their own voice. The skill names which source it used before drafting, so a fall-through to the house style is visible rather than silent.

For a one-off, skip the file entirely:

```
/humanizer

Here's a sample of my writing for voice matching:
[paste 2-3 paragraphs of your own writing]

Now humanize this text:
[paste AI text to humanize]
```

A sample changes how the skill runs, not just how the output reads. Before touching your text it writes out a profile of the sample: sentence-length range, punctuation rates per 100 words, register, pet words, recurring openers, and a list of **protected habits**. Protected habits are the numbered patterns your own writing already breaks. If you write in triads, the rule-of-three check is switched off for that rewrite. If you use em dashes, so does the output, at roughly your rate. Stripping a protected habit counts as a defect, the same as inventing a fact, and the final audit pass checks for it explicitly.

A sample governs how the writing sounds, not what it claims, so some rules stay put. It never licenses inventing facts. The content patterns (#1 through #6: puffery, vague attribution, and the rest) still apply, since an author fond of a habit does not make it accurate. Emojis, curly quotes, and chatbot artifacts stay banned regardless, because those come from the generating tool rather than from you. Every style rule below that yields to the sample.

Calibration runs in every invocation mode. Only its visibility changes: pasted text shows you the profile, file mode notes the matched habits in its summary, and embedded callers get the calibrated prose with no ceremony around it.

## Overview

Based on [Wikipedia's "Signs of AI writing"](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) guide, maintained by WikiProject AI Cleanup. This comprehensive guide comes from observations of thousands of instances of AI-generated text.

The skill also includes a final "obviously AI generated" audit pass and a second rewrite, to catch lingering AI-isms in the first draft.

Rewrites follow a no-fabrication rule: they never add facts, names, dates, or citations that aren't in the source text. Specificity has to come from the source or the author, not from the rewrite.

### Key Insight from Wikipedia

> "LLMs use statistical algorithms to guess what should come next. The result tends toward the most statistically likely result that applies to the widest variety of cases."

## 33 Patterns Detected (with Before/After Examples)

### Content Patterns

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 1 | **Significance inflation** | "marking a pivotal moment in the evolution of..." | "was established in 1989 as part of a wider decentralization" |
| 2 | **Notability name-dropping** | "cited in NYT, BBC, FT, and The Hindu" | Trim the list; keep only sourced context |
| 3 | **Superficial -ing analyses** | "symbolizing... reflecting... showcasing..." | Remove, or keep only what the source supports |
| 4 | **Promotional language** | "nestled within the breathtaking region" | "is a town in the Gonder region" |
| 5 | **Vague attributions** | "Experts believe it plays a crucial role" | Name a real source or cut the claim |
| 6 | **Formulaic challenges** | "Despite challenges... continues to thrive" | Keep the sourced facts; cut the boosterism |

### Language Patterns

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 7 | **AI vocabulary** | "Actually... additionally... testament... landscape... showcasing" | "also... remain common" |
| 8 | **Copula avoidance** | "serves as... features... boasts" | "is... has" |
| 9 | **Negative parallelisms / tailing negations** | "It's not just X, it's Y", "..., no guessing" | State the point directly |
| 10 | **Rule of three** | "innovation, inspiration, and insights" | Use natural number of items |
| 11 | **Synonym cycling** | "protagonist... main character... central figure... hero" | "protagonist" (repeat when clearest) |
| 12 | **False ranges** | "from the Big Bang to dark matter" | List topics directly |
| 13 | **Passive voice / subjectless fragments** | "No configuration file needed" | Name the actor when it helps clarity |

### Style Patterns

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 14 | **Em/en dashes** | "institutions—not the people—yet this continues—" | Cut them: periods, commas, colons, or parentheses |
| 15 | **Boldface overuse** | "**OKRs**, **KPIs**, **BMC**" | "OKRs, KPIs, BMC" |
| 16 | **Inline-header lists** | "**Performance:** Performance improved" | Convert to prose |
| 17 | **Title Case Headings** | "Strategic Negotiations And Partnerships" | "Strategic negotiations and partnerships" |
| 18 | **Emojis** | "🚀 Launch Phase: 💡 Key Insight:" | Remove emojis |
| 19 | **Curly quotes** | `said “the project”` | `said "the project"` |
| 26 | **Hyphenated word pairs** | “cross-functional, data-driven, client-facing” | Drop hyphens on common word pairs |
| 27 | **Persuasive authority tropes** | "At its core, what matters is..." | State the point directly |
| 28 | **Signposting announcements** | "Let's dive in", "Here's what you need to know" | Start with the content |
| 29 | **Fragmented headers** | "## Performance" + "Speed matters." | Let the heading do the work |
| 30 | **Diff-anchored writing** | "This function was added to replace..." | Describe what it does, not what changed |
| 31 | **Manufactured punchlines / staccato drama** | "It had no preference. No prior. No nostalgia." | Use varied sentence lengths and concrete claims |
| 32 | **Aphorism formulas** | "Symmetry is the language of trust" | Replace the formula with the actual claim |
| 33 | **Conversational rhetorical openers** | "Honestly? It depends..." | Remove the fake-candid setup |

### Communication Patterns

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 20 | **Chatbot artifacts** | "I hope this helps! Let me know if..." | Remove entirely |
| 21 | **Cutoff disclaimers** | "While details are limited in available sources..." | Find sources or remove |
| 22 | **Sycophantic tone** | "Great question! You're absolutely right!" | Respond directly |

### Filler and Hedging

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 23 | **Filler phrases** | "In order to", "Due to the fact that" | "To", "Because" |
| 24 | **Excessive hedging** | "could potentially possibly" | "may" |
| 25 | **Generic conclusions** | "The future looks bright" | Specific plans or facts |

## Full Example

*(Illustration note: the rewrite below adds specifics, like the month and the neighborhoods, that stand in for details the author would supply. In a real session those come from the user; the skill asks rather than invents.)*

**Before (AI-sounding):**
> I recently spent five unforgettable days in Lisbon, and let me tell you — this city completely stole my heart. From the moment I arrived, I knew I was somewhere truly special.
>
> Nestled along the banks of the Tagus River, Lisbon stands as a vibrant testament to Portugal's enduring spirit, where rich history and modern energy intertwine at every turn. Yes, the famous hills are challenging — my legs certainly felt it! — but every climb rewards you with breathtaking, panoramic views that make it all worthwhile.
>
> No trip would be complete without riding the iconic Tram 28, winding through the city's most historic neighborhoods. And the food? Simply divine. The original pastéis de nata at Pastéis de Belém are a beloved national treasure, and savoring one still warm was a moment I will never forget.
>
> But what truly makes Lisbon special isn't just the sights — it's the feeling. Wander a few steps off the main squares and you'll discover a quieter, more authentic side: sun-drenched alleys, charming tiled facades, and friendly locals going about their daily lives, inviting you to slow down and savor every moment.
>
> Of course, the must-see São Jorge Castle offers stunning views over the rooftops below. The queues can be long, but the experience is absolutely worth it for any history buff or curious traveler.
>
> Would I go back? Absolutely. Lisbon isn't just a place to visit — it's a place to fall in love with, again and again. If you're dreaming of your next getaway, this is one destination that promises memories to last a lifetime. ✨

**After (Humanized):**
> I spent five days in Lisbon last October and still have mixed feelings about it. Beautiful, yes. Also harder on the knees than anyone warned me.
>
> The hills are the whole story and somehow never make the brochures. My hotel was up in Alfama, which photographs beautifully and translates, in practice, to climbing what felt like a six-story staircase every time I wanted coffee. By the second day my calves had opinions. I started planning each walk around which way was downhill, which is a strange way to see a city but probably an honest one.
>
> Everyone says to ride Tram 28, so I did, wedged against a stranger's backpack for forty minutes while three tour groups filmed the same corner. I would walk the route next time, or go before breakfast. The custard tarts, though, earn the fuss. I had one at a plain little place in Graça, still warm, and for about thirty seconds I understood why people build trips around pastry.
>
> What I did not expect was how quiet the city gets away from the main squares. Two blocks uphill from any plaza it turns into laundry lines, chipped tile, open windows, and old men watching football with the sound turned up. That is the Lisbon I keep thinking about, not the castle.
>
> The castle is fine. The view is great, the queue is long, and I spent more time shuffling toward the entrance than looking at anything once I got inside. If I had only two days, I would trade it for an afternoon of getting lost.
>
> I would go back, but in spring and with better shoes. Lisbon does not bend over backward to make things easy for you. I think I liked that, even when my legs disagreed.

## References

- [Wikipedia: Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) - Primary source
- [WikiProject AI Cleanup](https://en.wikipedia.org/wiki/Wikipedia:WikiProject_AI_Cleanup) - Maintaining organization

## Version History

- **2.11.1** - Resolved a contradiction in the voice file spec: `SKILL.md` banned samples the user did not author, which blocked the documented case of targeting another writer's voice. Third-party samples are now allowed when the user picks that target, provided the writing is real and attributed; inventing prose to fill the file stays banned. Also defined what happens to sections other than `## Samples` and `## Profile`: they are honored as standing instruction but do not move in the precedence order. No change to the 33 patterns.
- **2.11.0** - Gave voice calibration somewhere to live. It previously had to be re-pasted every session, so in practice it rarely ran. The skill now checks `.humanizer-voice.md` in the project root and `~/.claude/humanizer/voice.md`, in that order, even when the user says nothing about voice, and names the source it used so a fall-through to the house style is visible. The file holds raw samples; the first run derives the profile and caches it in the same file, and deleting that block forces a re-derive. Placeholder or very short files are ignored rather than treated as a voice. No change to the 33 patterns.
- **2.10.0** - Made voice calibration binding instead of advisory. It had been an optional aside that the runtime loop never referenced, so the 33 pattern rules ran over a provided sample. It is now step 0 of the process, it requires a written profile with a protected-habit list before drafting, it carries an explicit precedence order (no-fabrication first, then a hard floor of emojis/curly quotes/chatbot artifacts, then the sample, then everything else), and a third audit question checks the rewrite against the profile so a scrubbed protected habit is caught as a defect. `PERSONALITY AND SOUL` is now subordinate to a real sample rather than competing with it. No change to the 33 patterns.
- **2.9.1** - Improved distribution and portability: removed nonportable frontmatter and tool preapprovals, made global installation the documented default, added package validation, and removed the duplicated long-form example from the runtime prompt. No change to the 33 patterns.
- **2.9.0** - Added a no-fabrication rule: rewrites may not invent facts, names, dates, or citations not present in the source, and every example that modeled invented specifics was re-cut to use only source information (fixes #187). Replaced paragraph-count parity with an information-over-shape rule, made a user's voice sample outrank the em dash ban, and added invocation modes (pasted text / file / embedded). No change to the 33 patterns.
- **2.8.3** - Moved the skill version from the unsupported top-level frontmatter key to `metadata.version` for Agent Skills and Claude compatibility. No change to the 33 patterns.
- **2.8.2** - Replaced the full before/after example with a first-person Lisbon trip recap. The after now keeps the same topic, perspective, and rough length as the before while removing the AI tells without becoming clipped or slogan-like. No change to the 33 patterns.
- **2.8.1** - Added cross-agent installation docs, optional Claude Code plugin packaging, and a compact secondhand-text false-positive guard. No change to the 33 patterns.
- **2.8.0** - Added style/cadence patterns #31-33 for manufactured punchlines, aphorism formulas, and conversational rhetorical openers; expanded #20 to catch offer-to-continue chatbot closers. 33 patterns total.
- **2.7.0** - Added pattern #30 (diff-anchored writing); made em/en dashes a hard cut rather than "overuse"; expanded #21 to cover speculative gap-filling ("maintains a low profile"). 30 patterns total.
- **2.6.0** - Cleanup pass: consolidated the duplicated workflow sections, gated the personality guidance to content where voice is wanted, removed the model-fingerprinting subsection, and condensed the worked example. No change to the 29 patterns.
- **2.5.1** - Added a passive-voice / subjectless-fragment rule, raising the total to 29 patterns
- **2.5.0** - Added patterns for persuasive framing, signposting, and fragmented headers; expanded negative parallelisms to cover tailing negations; tightened wording around em dash overuse; fixed frontmatter wording to use "filler phrases"
- **2.4.0** - Added voice calibration: match the user's personal writing style from samples
- **2.3.0** - Added pattern #25: hyphenated word pair overuse
- **2.2.0** - Added a final "obviously AI generated" audit + second-pass rewrite prompts
- **2.1.1** - Fixed pattern #18 example (curly quotes vs straight quotes)
- **2.1.0** - Added before/after examples for all 24 patterns
- **2.0.0** - Complete rewrite based on raw Wikipedia article content
- **1.0.0** - Initial release

## License

MIT
