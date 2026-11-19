---
name: "arxiv-digest"
description: "Set up a personalized, self-updating arXiv literature digest as a scheduled task, built from the user's own papers. Use when someone wants to follow new arXiv papers in their field."
---

# arXiv digest

This skill sets up a recurring digest of new arXiv papers for a researcher in any field. It is tailored to them and adapts as they publish. Each run rebuilds the researcher's profile from their own arXiv papers, scans the relevant new listings, and writes a short digest. Each paper in the digest gets plain-language explanations, its jargon decoded, and a concrete note on why it matters to *their* work. The digest goes out as a doc, with a push notification.

The deliverable is a **scheduled task**, not a one-off summary. If the user only wants today's papers once, do that directly instead.

## Step 1: Gather inputs (one AskUserQuestion round, plus free text)

You need:
1. **Their arXiv identity.** The best source is an arXiv author page or feed, e.g. `https://arxiv.org/a/<id>.html`; the feed is `https://arxiv.org/a/<id>.atom2`. If they have none (new researcher, unclaimed author ID), take a list of 2–5 arXiv IDs of their papers, or papers they consider central to their interests (e.g. their thesis topic).
2. **Papers to exclude**, such as side projects that shouldn't shape the profile. Ask once: "Any of your papers that are side projects and shouldn't steer the digest?"
3. **Frequency:** twice a week (recommended), every weekday, or weekly.
4. **Delivery:** doc + push notification (recommended), notification + email, or notification only.
5. **Depth:** tiered (recommended: 3–5 must-reads explained properly, the rest one line each), short for all, or deep on few.
6. **Breadth:** optional extra topics, and whether they want a few headline papers from the wider field.
Also use their time zone, which is usually in context. Ask only if it's unknown.

## Step 2: Verify the source and confirm the profile

Fetch their author feed (or the abstracts of the listed papers) **once**. arXiv rate-limits bursts, so fetch arxiv.org pages one at a time, never in parallel. If you're blocked, wait about 60 s and retry once. If it still fails, carry on with what the user told you, and say so.

From the papers (excluding side projects), draft a short profile and show it to the user in 4–6 lines:
- core topics, methods/codes, and the observables or datasets they work with;
- **arXiv categories to scan**: the primary and cross-list categories of their papers (e.g. `astro-ph.CO`, `hep-th`, `cs.LG`, `q-bio.NC`);
- adjacent areas worth watching;
- the wider field for headline papers.
Ask them to correct it. This confirmation is the main quality lever.

Note very large categories (e.g. `cs.LG`, `cs.CV`, `cs.CL`, `quant-ph`, `astro-ph.GA`). These get hundreds of papers a day, so the run must filter by title first and may need to page through listings.

## Step 3: Pick the schedule

New arXiv listings are announced Sunday–Thursday at 20:00 US Eastern time. That is 00:00 UTC during US daylight time and 01:00 UTC otherwise. Schedule runs 1–3 hours after announcement **in the user's local time zone**, using `CRON_TZ=<IANA zone>`, on an off-the-hour minute.
- Twice a week: two days about 3–4 days apart. For time zones east of UTC, announcements land the next local morning, so e.g. Monday and Thursday mornings work. For the Americas, e.g. Monday and Thursday evenings after 20:00 ET, or the following mornings.
- Every weekday: every morning after announcement, local Monday–Friday for zones east of UTC.
- Weekly: one morning, covering the past week.
Work out the coverage window for each run (which announcement days it covers) and write it into the prompt explicitly.

## Step 4: Create the scheduled task

Use the session's scheduling tool (create_trigger or equivalent; find it with ToolSearch if needed). Check the existing scheduled tasks first so you don't create a duplicate digest. Set push notifications on. Turn email on only if the user chose email. Name it "arXiv literature digest". Fill in the prompt template below. Every firing starts a fresh session, so the prompt must stand alone.

After creating it, tell the user in a few lines: when it runs, what each digest contains, and the task's approval setting. Also tell them they can steer it any time ("more X", "drop Y", "I started working on Z") and that you'll update the task's prompt. Mention that the task's settings have **Run now** if they want a sample immediately.

## Prompt template (fill every <placeholder>)

```
You are producing a <frequency> arXiv literature digest for <name>, a <career stage> working on <one-line research summary>. Keep them up to date without effort. Filter hard, explain clearly, and say why each paper matters to THEM. Their interests change as they publish, so rebuild their research profile from their own papers at the start of every run.

IMPORTANT: arXiv rate-limits requests. Fetch arxiv.org pages ONE AT A TIME (never several in one turn) with a short pause between them. If a fetch is rate-limited, run `sleep 60` in the shell and retry, up to 3 times, before moving on. The shell usually cannot reach arxiv.org; use WebFetch.

## Step 1: Research profile (every run)
1. WebFetch <author feed URL, e.g. https://arxiv.org/a/<id>.atom2>. If that fails, use <author HTML page>. [If no author page: read the abstracts of <arXiv IDs>.] List their papers (ID, title, coauthors, date).
2. EXCLUDE: <side-project papers/topics, or "nothing">. Never use these to shape the profile, list them as their work, or track their citations.
3. Use the feed's abstracts. For their 3 most recent papers, also skim the intro and conclusions (https://arxiv.org/html/<id>) for the open problems and future work they mention.
4. Write a working profile: core topics, methods/codes, observables/datasets, open questions, coauthors. Weight recent papers more heavily. If a paper newer than the baseline appears, note "profile updated with arXiv:XXXX" in the footer.
Baseline if nothing can be read: <the confirmed profile from setup>. Fields where they may need jargon explained: <adjacent fields>.

## Step 2: Who cited them
For each of their (non-excluded) papers, fetch https://api.semanticscholar.org/graph/v1/paper/arXiv:<id>/citations?fields=title,externalIds,authors,publicationDate&limit=100 and list citing papers from about the last <window length + a few days>, with one sentence each on how they use the work. Omit the section if there are none.

## Step 3: Coverage window
<explicit per-run window, e.g. "The Monday run covers announcements since Thursday's run; the Thursday run covers announcements since Monday's run. If triggered manually, cover the last ~3 announcement days."> Use the announcement dates on the pastweek listing pages.

## Step 4: Scan listings
For each category in <categories>: https://arxiv.org/list/<cat>/pastweek?show=2000 (for very large categories, page with &skip=N and filter by title first). Ask WebFetch for every arXiv ID + title + authors in the window. If the output looks truncated, fetch the remainder. Read https://arxiv.org/abs/<id> for each candidate. For must-reads, skim https://arxiv.org/html/<id> so the explanation reflects the actual method and results. Never invent results, numbers or authors. Say when something couldn't be read.

## Relevance rings (score against the Step 1 profile)
- Ring 1, core: directly touches their topics, methods, data or open questions: <core topics>. Also papers by their coauthors on these topics.
- Ring 2, adjacent: <adjacent areas>.
- Ring 3, wider field: at most 3 headline papers from <broad field> that people will be discussing, chosen by likely impact. [Omit if not wanted.]
<extra user instructions, e.g. topics to avoid>

## Digest structure
Title: "Literature digest – <date range of announcements>"
1. Must-reads (<N>). For each: title, first author et al., arXiv link. Then: the gist (2–3 plain sentences); jargon decoded (2–5 terms, one line each); how it works (method + key assumption); why it matters to you (a concrete link to their specific papers or open questions, honest if weak); read it if… (one line).
2. Citing your work (if any).
3. Worth knowing (Ring 1–2, up to ~12): one line each.
4. Wider field (Ring 3, ≤3): two sentences each.
5. Jargon of the week: a 5–8 term mini glossary.
6. Footer: listings scanned, window, papers used for the profile, anything inaccessible.
If the window is thin, keep the digest short. Never pad it.

## Delivery
Create the digest as a Claude Docs document (mcp__Claude_Docs__ tools; call its guide with topic.instructions first). If unavailable, publish a self-contained HTML page via the claude-artifacts tools (read its 'pages' guide first). If neither works, put the full digest in the final message. The final message must be short: the window, the must-read titles, any citing-your-work hits, and the link. It becomes the push notification. Do not ask questions; nobody is watching. Note your choices in the footer.
```

## Adjusting later

When the user asks for changes ("more X", "fewer Y", new interests, a different schedule), find the task with list_triggers and update its prompt or schedule with update_trigger. Keep the task's identity; don't delete and recreate it. Change only the relevant lines and resend the full prompt.