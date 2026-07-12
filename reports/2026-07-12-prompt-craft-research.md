# Prompt-Craft Research — how to build prompts that work and sell

Date: 2026-07-12
Scope per master file amendment: findings feed v2 and products #2/#3 ONLY.
The v1.3 Honest Analyst stays frozen.

Sources: Anthropic's official prompting best-practices documentation
(platform.claude.com/docs), Anthropic prompt-engineering blog (claude.com/blog),
PromptBase seller guides and community seller write-ups (promptbase.com/sell,
Medium seller reports, Gold Penguin PromptBase review, Young Urban Project
6-part framework).

## Part 1 — What the AI makers themselves teach (Anthropic docs)

1. **Be explicit and direct.** Vague prompts get average output. If you want
   exceptional behavior, ask for it in plain words.
   → Validation: v1.3 already does this ("exactly these sections", "never break").

2. **Explain WHY a rule exists.** Models follow rules better when given the
   motivation, not just the order.
   → Validation: our "one unbreakable professional ethic" framing is exactly
   this. Keep the pattern in products #2/#3.

3. **Examples are the strongest steering tool.** 3–5 worked examples inside the
   prompt improve consistency more than more instructions do. Wrap them in
   `<example>` tags so the AI knows they're examples, not instructions.
   → v2 idea: embed ONE fully-worked mini example (tiny fake company, 5 numbers,
   3-line output) inside the prompt so every buyer sees the pattern before
   their first run.

4. **Structure with XML-style tags.** Separating <instructions>, <data>,
   <examples> reduces the AI mixing them up. Helps Claude especially.
   → v2 idea: wrap the pasted financials in <data> tags, task in <task> tags.

5. **Role prompting works.** A defined expert role in the first line measurably
   improves domain performance.
   → Validation: "senior equity research analyst, 15+ years" is textbook.

6. **BIG ONE — long data at the TOP, task at the END.** Anthropic's docs:
   putting large pasted documents at the top and the question/instructions at
   the end improves response quality by up to ~30% on data-heavy prompts.
   → Our v1.3 does the opposite (persona → data → task). For v2 and product #2,
   test the flipped structure: data first, all instructions after. This is the
   single most promising structural upgrade we found.

7. **Ground answers in quotes.** For long documents, asking the AI to first
   quote the relevant lines, then analyze, cuts hallucination.
   → Validation: our "each traced to an exact number or sentence" rule is this
   idea. v2 could strengthen it: "quote the line before judging it."

## Part 2 — What successful PromptBase sellers teach

8. **Test 10–15 times before listing; consistency IS the product.** Buyers pay
   for reliable output, not one lucky run.
   → Action for #2/#3: before listing, run each product on 3+ different
   companies (big/small, healthy/ugly numbers) and keep the outputs as a
   consistency log in the repo. (v1.3 had 3 strong runs; its log exists.)

9. **Specialize; don't generalize.** "Technical SEO blog generator for SaaS"
   outsells "blog writer." Specific problem + defined buyer = higher price.
   → Validation: finance niche + DIY investor buyer is exactly this play.

10. **The listing must teach.** Clear how-to-use steps, sample output, short
    formatted paragraphs. Confused buyers don't return; unclear listings
    don't convert.
    → Validation: our listing has HOW TO USE (60 seconds) + example PDF +
    screenshots. Matches the guidance fully.

11. **"25–50 words sell best" — TRUE FOR IMAGE PROMPTS, NOT FOR US.** That
    stat comes from image-prompt sellers. Complex text systems like ours sell
    on depth and reliability. Do not shorten the product to chase this number.

12. **Marketing loop.** Sellers who post output screenshots on social media
    2x/week report meaningfully more profile visits.
    → Already in master file section 8, stage 2 (after first 5 sales).

## Summary for Ali (plain words)

Most findings CONFIRM what we already built — role, explicit rules, reasons,
traceability, teaching listing. Three genuinely new upgrades were found:

1. Flip the structure: pasted data at top, instructions at bottom (up to ~30%
   quality gain per Anthropic) — test in product #2 first.
2. Embed one tiny worked example inside the prompt (few-shot) for consistency.
3. Formal consistency testing: 3+ different companies per product before
   listing, logged in the repo.

All three go to v2 / product #2. Nothing touches frozen v1.3.
