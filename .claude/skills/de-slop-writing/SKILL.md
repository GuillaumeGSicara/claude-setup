---
name: "de-slop-writing"
description: "Check a text for signs of AI writing and return a severity-ranked table of fixes. Use when reviewing or finalising text for publication or when asked to de-slop or humanise it."
---
 
# De-slop writing
 
## Procedure
 
1. **Context.** Detect the text's language and publishing surface. Name the intended reader.
2. **Facts first.** Check the central claim and rows W01–W02 against primary sources before looking at wording. If a source cannot be fetched, report the claim as unverified; never pass it.
3. **Scan** the remaining rows in ID order.
4. **Guards.**
   - Every finding cites a row. Correct grammar, formal register, a lone transition word or unsourced text are not findings on their own.
   - Propose fixes; do not rewrite the text. Keep the author's voice and never make the text longer.
   - Never invent facts, sources or examples. When a fix needs one, the suggestion is a question to the author (use the human column).
5. **Output** the priority table below, and nothing else beyond one line on omissions.
## Output
 
| Item | Reference | Description | Suggestion |
|---|---|---|---|
 
- **Item**: the quoted span (under 15 words) or its location ("last paragraph").
- **Reference**: the control ID. IDs are numbered by severity (W01–W06 blocker, W07–W18 major, W19–W34 minor), so rows sorted by ID are ranked by severity.
- **Description**: what is wrong, one line.
- **Suggestion**: the fix, or a question for the author.
**Threshold.** At most 20 rows. One row per control: group repeated hits and give the count. Severity 1 and 2 are always reported; severity 3 only when it hits twice or more. If still over 20, cut from the bottom and add one line: "N minor findings omitted". If nothing is found, say so in one line.
 
## Severity 1: blockers
 
A reader can prove the text wrong or machine-made.
 
| ID | Control point | Reject when | Countermeasure: AI | Countermeasure: human |
|---|---|---|---|---|
| W01 | Unsupported attribution | A source is credited with a claim it does not make; a paper cited from its abstract or a search snippet | Quote the supporting sentence from the full text (arXiv HTML at a version number, not the abstract page); no sentence, no attribution | Read the source passage for the claims the text stands on |
| W02 | Broken identifiers | DOI resolving to an unrelated paper, ISBN with a bad checksum, book cited without a page | Resolve each DOI and match the title; validate ISBN checksums; require page numbers | Open the cited page |
| W03 | Chatbot residue and external references | "I hope this helps", "Would you like", knowledge-cutoff notes, [Name], (add URL here), xx dates; pointers to what produced the text: "as discussed above", "based on the transcript", "the document you shared", the prompt or the conversation | Scan for brackets, placeholders, chat phrases and pointers; inline what the reader needs or cut | Read as a stranger holding only the text; cut what you cannot resolve |
| W04 | Abrupt cutoff | Text stops mid-sentence; a word split by a formatting span ("informatio·n") | Check the last sentence is complete and bold or italic spans end on word boundaries | Read the rendered version once |
| W05 | Markdown residue | Literal *, ###, ---, backticks or [text](url) on a surface that shows them raw | Use the surface's own formatting; on LinkedIn: line breaks, • or → bullets, links in the first comment | Paste into the surface preview and read it there |
| W06 | Tracking parameters | utm_source=chatgpt.com, utm_source=openai, utm_source=copilot.com, referrer=grok.com | Strip utm_ and referrer parameters | Glance at pasted URLs |
 
## Severity 2: major
 
The argument reads as generated.
 
| ID | Control point | Reject when | Countermeasure: AI | Countermeasure: human |
|---|---|---|---|---|
| W07 | Inflated significance | Facts framed as pivotal, a testament, reflecting broader trends, setting the stage, an evolving landscape; "X is new" when its parts already exist | Delete the framing, keep the fact; attach a number, date or consequence; for "new", state what exists and what exactly is new | Ask "says who?"; challenge every "new" |
| W08 | Vague or inflated attribution | industry reports, experts argue, some critics, several sources when there is one | Name the source or remove the sentence; never pluralize one source | If no source exists, own the claim ("I think") |
| W09 | Promotional register | boasts, vibrant, rich, showcases, groundbreaking, renowned, nestled, diverse array, commitment to; a team praised before the result | Rewrite neutral: problem, result, follow-up | Reject anything that reads like a press release |
| W10 | Canned notability | Relevance proven by describing coverage: independent coverage, trade publications, featured in, active social media presence | Replace the description of the source with what it said | Confirm the source says that |
| W11 | Trailing -ing analysis | Sentences end on highlighting, ensuring, fostering, reflecting, contributing to, valuable insights, align with | Split into a sourced claim or delete the clause | Cut last clauses that add opinion without evidence |
| W12 | Hedging disclaimers | "should be treated as X rather than Y", "does not by itself establish", "should not be confused with", "not publicly available", "important to note" | State what is known; flag unknowns to the author in one line | Fill the gap or drop the sentence |
| W13 | Formula opening and closing | Opens on a narrative hook ("nobody ordered this"), a slogan or a definition; ends on "In conclusion", "Despite these challenges", a restating paragraph, Challenges or Future outlook sections, or a generic question to readers | Open on a dated event, a number or the author's anecdote; end on the last point or one direct question to one named person | Supply the anecdote; name the person |
| W14 | Abstract where concrete would do | Definitions and claims with no situation; the argument stays at principles | Ask the author for one case; never invent one | Supply a case from your own work |
| W15 | AI vocabulary cluster | Several hits from either list. Earlier: Additionally, align with, boasts, bolstered, crucial, deep dive, delve, emphasizing, enduring, enhance, fostering, garner, highlight (verb), interplay, intricate, landscape, meticulous, pivotal, robust, showcase, tapestry, testament, underscore, valuable, vibrant. 2026: challenge, clearer, dependable, echoed, foster, leverage, matters, multifaceted, practical, prioritize, quietly, steady, universally | Literal word check; report the count; rewrite hits in plain words; synonyms are not flagged | Judge density: one literal use is fine. Refresh both lists quarterly from Wikipedia's "Signs of AI writing" |
| W16 | Negative parallelism | "not just X but Y", "it's not X, it's Y", "no X, no Y, just Z", "Y rather than X", mirrored pairs ("Detection was never the problem. Read speed is.") | State the point once | If it sounds like a tagline, cut it |
| W17 | Mirror aphorisms and slogans | "Four dates, one mechanism", "The acceleration has a name: X", paired punchlines, "That's the point" pivots | Delete | Let the reader draw the slogan |
| W18 | Writer-side vocabulary | Internal project names, team jargon, tool names or acronyms the reader does not own; undefined terms stacked | Use the reader's terms; explain one in passing; drop the rest | Name the reader; reject words you would have to explain to them |
 
## Severity 3: minor
 
Style tells. Report only when they recur.
 
| ID | Control point | Reject when | Countermeasure: AI | Countermeasure: human |
|---|---|---|---|---|
| W19 | Rule of three | Adjective triads, three short phrases, three stats in a row | Keep only the items that carry content | Check the content really has three parts |
| W20 | Copula avoidance | serves as, stands as, functions as, represents, features/offers for has, "refers to" in a lead | serves as → is, features → has, refers to → is | Accept the plain version |
| W21 | Uniform rhythm | Every sentence 8 to 20 words, commas and periods only | Vary length; allow colons, semicolons, parentheses, questions | Read aloud; rework if it ticks like a metronome |
| W22 | Elegant variation | Synonyms rotated to avoid repeating a word | Repeat the term | Accept repetition |
| W23 | Stiff synonyms | authored, relocated, utilized, attempted, passed away, commence | wrote, moved, used, tried, died, start | Prefer the plain verb |
| W24 | Vague association | "connected with", "associated with" instead of the relation | Rewrite as who, what role, when; flag the gap if unknown | Supply the fact or accept the cut |
| W25 | Fake range | "from X to Y" spanning unrelated items to suggest breadth | Name the items or drop the span | Cut |
| W26 | Name-dropping | Concepts explained through a parade of people and companies | Keep the names that carry the argument | Cut names that are there to impress |
| W27 | Missing human signals | No superlatives, qualifiers or definitive statements anywhere; the author's ones stripped by an edit | Never remove "the first", "the only", "very", "perhaps", "tends to" the author wrote | Keep them when true |
| W28 | Em dashes | Spaced em dashes for emphasis or parallelism, several per piece | Replace with comma, colon, period or parentheses, keeping rhythm varied (W21) | Check it still reads with rhythm |
| W29 | Boldface overuse | Key phrases bolded mechanically | At most one bold phrase per piece | Pick the one, or none |
| W30 | Inline-header lists | Bullet, bold term, colon, description, down the page | Convert to prose or a plain list | Accept prose where the list was decoration |
| W31 | Emoji as formatting | Emoji opening headings or bullets, or marking sections | Remove, unless the brief names a series signature | Name your signature in the brief |
| W32 | Title Case headings | Every Main Word Capitalized | Sentence case | Spot-check rendered headings |
| W33 | Decorative tables | A two-column table for three facts | Tables only for items × attributes; otherwise a sentence | Nothing beyond a read |
| W34 | Quote mix | Curly and straight quotes or apostrophes mixed in one text | Normalize to one style | Pick the style once per surface |