---
name: natural-writing
description: Enforces natural, human-sounding prose by avoiding AI tells (overused vocabulary, em dashes, inflated phrasing, formulaic structures). Use when creating or editing markdown files, documentation, READMEs, articles, blog posts, proposals, or reports. Also use when reviewing or rewriting text to sound more natural. Applies to prose output only, not to code, commit messages, or conversational replies.
---

# Natural Writing

AI-generated text has recognizable fingerprints. Readers, editors, and detection tools pick up on them quickly, and they erode trust in the writing. This skill exists to produce prose that reads like a competent human wrote it, drawing on patterns catalogued by Wikipedia editors who review thousands of AI-generated submissions.

The guidance below is organized from most to least impactful. It applies to prose and documentation output only, not to code, commit messages, or conversational replies.

## Vocabulary

Certain words spiked dramatically in usage after 2022 because language models over-select them. Their presence in clusters is one of the strongest AI signals. Avoid these unless the context genuinely demands them:

**High-signal words (avoid in almost all cases):**
delve, tapestry, testament, vibrant, pivotal, crucial, landscape, meticulous/meticulously, intricate/intricacies, interplay, underscore, bolster/bolstered, garner, enduring, showcasing, highlighting, fostering, encompassing, nestled, groundbreaking, renowned

**Medium-signal words (use sparingly, never in clusters):**
additionally, key, valuable, enhance, align with, diverse array, in the heart of, commitment to, exemplifies, profound, rich (when not literal), boasts

The problem is not any single word. It is the co-occurrence pattern. A paragraph containing "delve into the intricate tapestry" is unmistakable. One containing "intricate" by itself, where the complexity is real, is fine. Use judgment: if a simpler word works just as well, prefer it.

## Copulative avoidance

AI text systematically avoids "is" and "are," replacing them with elaborate alternatives. This is one of the most measurable AI tells (a roughly 10% drop in copulative usage post-2022).

**The pattern:**
- "serves as" / "stands as" / "marks" / "represents" instead of "is"
- "boasts" / "features" / "maintains" / "offers" instead of "has"

**Why it matters:** When everything "serves as a testament" or "stands as a beacon," the writing feels inflated. Simple subjects deserve simple verbs. A bridge is a bridge. It does not need to "stand as a symbol of engineering progress."

Write "X is Y" when that is what you mean. Save the elaborate phrasing for cases where the indirection adds real meaning.

## Structural formulas

### Negative parallelisms

The construction "not just X, but also Y" (and its variants "not only...but," "it's not...it's") is a rhetorical tic that AI uses far more often than human writers. It creates an artificial sense of correcting a misconception that nobody had.

Before writing "It's not just a tool, it's a paradigm shift," ask: who thought it was just a tool? If nobody, drop the construction and state the point directly.

### Rule of three

AI loves triple-adjective lists and three-item enumerations: "robust, scalable, and efficient" or "clarity, precision, and depth." Humans use them too, but AI reaches for them reflexively. Vary your list lengths. Sometimes two items are enough. Sometimes four is right. Do not default to three.

### Elegant variation

Repetition-penalty mechanisms cause AI to swap synonyms unnecessarily: a character becomes "the protagonist," then "the key figure," then "the central player," all in the same paragraph. Human writers repeat words when clarity demands it. If you introduced someone as "the engineer," keep calling them "the engineer." Forced synonym rotation confuses readers and signals AI authorship.

### Outline-like conclusions

AI articles tend to end with a rigid formula: "Despite [positives], [subject] faces challenges such as [vague list]. However, [subject] continues to [optimistic speculation]." This reads like a template because it is one. If a conclusion is needed, make it specific to the actual content. If there is nothing meaningful to conclude, just stop.

## Content patterns

### Inflating significance

AI connects subjects to grand themes using stock phrases: "setting the stage for," "marking a key turning point," "shaping the evolving landscape," "leaving an indelible mark." This happens because language models default to emphasizing importance regardless of whether the subject warrants it.

Write proportionally. A local library renovation is a local library renovation. It does not need to "reflect broader societal shifts in community engagement." Let the facts carry their own weight.

### Promotional tone

AI defaults to a press-release register: "boasts a diverse array of amenities," "showcasing a commitment to excellence," "nestled in the heart of downtown." Even when prompted for neutral tone, the output gravitates toward advertising copy.

Write as an informed observer, not a publicist. Describe what something does or is, not how impressive it supposedly is.

### Superficial analysis

Sentences that end with participial phrases claiming significance without evidence: "highlighting the importance of community involvement," "underscoring the need for continued innovation," "reflecting a broader trend toward sustainability."

These phrases substitute for actual analysis. If something highlights importance, explain how. If it reflects a trend, name the trend and give evidence. If you cannot, the claim probably does not belong in the text.

### Vague attribution

"Experts argue," "industry reports suggest," "observers have cited": attributions to unnamed authorities create the illusion of sourcing without substance. If you have a source, name it. If you do not, reframe the statement as your own analysis or drop it.

## Style

### Em dashes and double dashes

Do not use em dashes (—), en dashes (–), or double hyphens (--) as punctuation in prose. These are among the most recognizable AI writing tells, and substituting double hyphens for em dashes is the same pattern with different characters.

Every em dash can be replaced by a comma, a colon, a period, or parentheses. Restructure the sentence to use one of those instead. For example:

- "The system validates tokens, even expired ones, before proceeding" (instead of "tokens -- even expired ones -- before proceeding")
- "One thing was clear: the migration had to happen" (instead of "One thing was clear -- the migration had to happen")

There are no exceptions to this rule in document output.

### Excessive boldface

Bolding "key takeaways" or emphasizing every instance of an important term reads like a slide deck, not a document. Use bold sparingly. If everything is emphasized, nothing is.

### Title case in headings

Prefer sentence case for section headings ("How the system works") over title case ("How The System Works") unless the style guide explicitly requires title case.

### Emoji in prose

Do not decorate headings, bullet points, or paragraphs with emoji unless the context specifically calls for it (e.g., a casual Slack message template). Emoji in technical documentation or articles is a strong AI signal.

## How to apply this

These are not rules to mechanically check off. They are patterns to internalize. When writing or editing prose:

1. Write the content first, focusing on clarity and accuracy
2. Reread with fresh eyes, watching for clusters of the patterns above
3. When you spot one, ask whether simpler, more direct language would work just as well
4. It usually will

The goal is not to avoid every word on the vocabulary list. It is to write the way a careful, experienced human writer would: plainly, specifically, and without reaching for grandiosity that the subject does not support.
