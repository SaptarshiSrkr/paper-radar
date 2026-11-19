# paper-radar 📡

`paper-radar` holds `arxiv-digest`, a [Claude skill](https://support.claude.com/en/articles/12512180) that sets up a **personalized, self-updating arXiv literature digest** for researchers in any field.

Following arXiv every day is hard: you forget, the listings are long, and papers outside your niche are full of jargon. This skill turns Claude into a reading assistant that does the filtering and the explaining for you.

## What you get

A scheduled digest (twice a week by default) delivered as a doc, with a phone notification. Each digest contains:

- **Must-reads (3–5 papers)**, each with:
  - the gist in plain language;
  - the jargon decoded;
  - how the method works and its key assumptions;
  - **why it matters to *your* work**, linked to your specific papers and open questions;
  - whether the full paper is worth your time.
- **Citing your work**: new papers that cite you, and how they use your results.
- **Worth knowing**: other relevant papers, one line each.
- **Wider field**: a few headline papers everyone will be discussing.
- **Jargon of the week**: a mini glossary that builds your vocabulary over time.

### It adapts as you publish

Every run rebuilds your research profile from your arXiv author feed. It weights your most recent papers, and their stated open problems, most heavily. When you publish something new, the digest shifts to match. You can mark side-project papers so they never steer it.

## Requirements

- A Claude account with **skills** and **scheduled tasks** available. Use it in the Claude app on web, desktop or mobile.
- **The [alphaXiv](https://www.alphaxiv.org) connector (required).** arxiv.org blocks or rate-limits most automated reading, so the skill reads papers and finds your publications through alphaXiv. Connect it under **Settings → Connectors** before setting up the digest. The skill checks for it, offers to connect it if it's missing, and won't create the digest without it.
- Ideally, an [arXiv author ID](https://info.arxiv.org/help/author_identifiers.html). Your author page looks like `https://arxiv.org/a/<your_id>.html`. If you don't have one, a few arXiv IDs of your papers, or of papers central to your interests, work too.

> Claude Code users: the skill will load (see below), but the scheduled digest relies on features of the Claude app, so set it up from there.

## Install

**Claude app (recommended)**
1. Download this repo's [`arxiv-digest.zip`](arxiv-digest.zip), or zip the `arxiv-digest/` folder yourself.
2. In Claude, go to **Customize → Skills**, click **+ → Create skill → Upload a skill**, and upload the ZIP.
3. Make sure the skill is toggled on.

**Claude Code**
```bash
git clone https://github.com/SaptarshiSrkr/paper-radar
mkdir -p ~/.claude/skills && cp -r paper-radar/arxiv-digest ~/.claude/skills/
```

## Use

In a new chat, say something like:

> Set up my arXiv digest. My author page is https://arxiv.org/a/<your_id>.html

Claude will:
1. ask a few quick questions: side projects to exclude, frequency, delivery, depth;
2. read your papers and show you a draft profile (topics, methods, arXiv categories to scan) for you to correct;
3. schedule the digest in your time zone, shortly after arXiv's daily announcement;
4. tell you how to steer it later.

Afterwards you can steer it at any time:
- "more 21-cm papers, fewer JWST galaxies"
- "I've started working on simulation-based inference"
- "switch to weekly"

To get a sample right away, use **Run now** in the scheduled task's settings.

## How it works

The skill tells Claude how to gather your inputs, build and confirm a profile from your papers, pick a schedule aligned with arXiv's announcement times (Sun–Thu, 20:00 US Eastern), and create a scheduled task from a self-contained prompt template. Each run:

1. checks that the alphaXiv connector is attached (and stops with a notification if not);
2. finds your papers through alphaXiv and builds your current research profile;
3. checks Semantic Scholar for new papers citing you;
4. scans the "pastweek" listings for your categories, keeping only the announcement days in its window, and falls back to an alphaXiv recency search if arXiv blocks a listing;
5. scores papers in three rings: core, adjacent and wider field;
6. reads the abstracts through alphaXiv, and the full text of the must-reads, so its explanations are grounded in the papers;
7. writes the digest as a doc and sends a short notification.

The full logic is in [`arxiv-digest/SKILL.md`](arxiv-digest/SKILL.md). Edit it to change the digest format or the defaults.

## Limitations

- arXiv rate-limits bursts of requests. Papers are read through alphaXiv; only the category listings come from arxiv.org, and if a listing is blocked the run falls back to an alphaXiv recency search, which can miss papers with unusual titles or keywords. The footer says when this happens.
- alphaXiv can merge researchers who share a name. The skill stores your arXiv IDs at setup to keep your profile clean.
- Very large categories (e.g. `cs.LG`, `astro-ph.GA`) are filtered by title first, so a relevant paper with an uninformative title can slip through.
- Summaries are written by an AI. Check anything important against the paper itself.

## License

MIT
