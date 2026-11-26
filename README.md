# knof-v15

**English and Chinese Wikipedia as of 2 June 2015, cut into about 28 million
interlinked knowledge fragments.** Every article (5.71 million) is split along
its own section tree. Each fragment is a piece of Markdown with a title, a place
in its article and a source URL, and links to other articles.

| | Articles | Section-level fragments |
|---|---|---|
| English | 4.89 M | ~25 M |
| Chinese | 0.82 M | ~2.7 M |

> **Status:** not yet released. Download links will be posted here.

## Why this dataset

Wikipedia is written as articles, for people to read. Its knowledge sits in whole
pages, and a page is too big to cite, retrieve or link to precisely. This
dataset breaks the pages into fragments and keeps the structure that connects
them.

**That structure is a graph, and the graph carries semantic relations that
nobody wrote down as data.** Take *Yagodnoye, Magadan Oblast*:

- The section tree says what each fragment is about, at every depth: the text
  under *Yagodnoye → History* is the town's history, and in another article
  *比賽成績 → 決賽 → 決賽A* (results → final → final A) is the result of one
  race.
- A link says two things are related, and the section it sits in says how.
  Links in *Geography* go to the Chersky Range and the Kolyma River; links in
  *Notable residents* go to people. Neither relation appears in the article as a
  fact; both follow from where the link sits.
- Categories put the fragment among others of the same kind (*Urban-type
  settlements in Magadan Oblast*).

None of this needs an ontology. There is no schema to agree on and nothing is
inferred: every edge is something an editor wrote. Projects like DBpedia and
Wikidata keep only infobox fields or curated statements, and the rest of the
article, which is most of it, stays an opaque block of text. Here all of it is
addressable.

Two more things make this possible:

- **Rendered by Wikipedia itself, in 2015.** A dump holds wikitext, where even a
  release date is a template call such as `{{Film date|1988|12|16}}`, and
  templates nest and run Lua. The structure only becomes certain once MediaWiki
  has rendered the page, and today you can render only with today's templates.
  This crawl saved the pages as Wikipedia rendered them at the time.
- **Complete, in two languages.** Every non-redirect article in both editions on
  that day, in the same format.

## What you can do with it

- **A knowledge graph without an ontology.** Build the fragment graph from the
  section tree, the links and the categories. Rank sections rather than pages to
  find the hubs, or cluster fragments by what links to them.
- **Retrieval for RAG and agents.** Retrieve a fragment, follow its links to
  neighbouring fragments, and cite each step down to the section. Or write each
  article out as a Markdown file with metadata in front matter, so `grep`,
  editors and agents can read it directly.
- **Training data with citations.** Every fragment has a stable source, so you
  can generate question–evidence–answer triples for cited QA, fact-checking or
  retrieval training.
- **A decade of change.** Compare an article with today's version to see which
  sections grew, which were removed, and which facts changed.

## Files

| File | Content |
|---|---|
| `en.jsonl` | English Wikipedia |
| `zh.jsonl` | Chinese Wikipedia (June 2015 crawl) |
| `zh-supplement.jsonl` | Chinese articles missing from `zh.jsonl`, taken from an earlier 2015 crawl |

`zh.jsonl` and `zh-supplement.jsonl` never share an `id`; use both for the
fullest Chinese coverage. Files are published zstd-compressed, in parts.

## Record format

Two complete records, pretty-printed: [`samples/en.json`](samples/en.json)
(*Operation Flashpoint (series)*) and [`samples/zh.json`](samples/zh.json)
(*2012年夏季奧林匹克運動會划船男子雙人單槳比賽*). In the release files each
record is one line. Shortened:

```json
{
  "id": "en:Operation Flashpoint (series)",
  "title": "Operation Flashpoint (series)",
  "lang": "en",
  "url": "https://en.wikipedia.org/wiki/Operation_Flashpoint_(series)",
  "description": "Operation Flashpoint is a series of military simulation games. …",
  "intro": "…",
  "sections": [
    {"title": "Bohemia Interactive series", "markdown": "", "sections": [
      {"title": "Operation Flashpoint: Cold War Crisis (2001)", "markdown": "Main article: [Operation Flashpoint: Cold War Crisis](https://en.wikipedia.org/wiki/Operation_Flashpoint:_Cold_War_Crisis \"Operation Flashpoint: Cold War Crisis\") …", "sections": [
        {"title": "Expansion pack", "markdown": "…", "sections": []}
      ]}
    ]},
    {"title": "Codemasters series", "markdown": "", "sections": [
      {"title": "Operation Flashpoint: Dragon Rising (2009)", "markdown": "…", "sections": []},
      {"title": "Operation Flashpoint: Red River (2011)", "markdown": "…", "sections": []}
    ]},
    {"title": "References", "markdown": "…", "sections": []}
  ],
  "categories": ["Video game franchises", "Military combat simulators", "…"],
  "links": [{"anchor": "…", "url": "https://en.wikipedia.org/wiki/…"}],
  "images": ["https://upload.wikimedia.org/…"]
}
```

`sections` is the article's own section tree, nested as on the page. A heading
that only groups subsections (*Codemasters series* above) is kept, with empty
`markdown`, so the tree is never broken.

| Field | Meaning |
|---|---|
| `id` | `<lang>:<title>`, unique within the dataset |
| `title` | article title |
| `lang` | `en` or `zh` |
| `url` | the article's Wikipedia URL (today's page, not the 2015 revision) |
| `description` | short plain-text abstract; may be empty |
| `intro` | lead section, Markdown |
| `sections` | the rest of the article as a tree: each node has `title`, `markdown` (may be empty for a heading that only groups subsections) and its own `sections`, in reading order |
| `categories` | category names, without the `Category:` prefix; includes Wikipedia's maintenance categories (e.g. *All articles with unsourced statements*) |
| `links` | links in the lead section, with anchor text, as the 2015 parser recorded them; **always empty in `zh-supplement.jsonl`**. Links in every part of the article stay inline in its Markdown, so you can recover them per section |
| `images` | image URLs from the page; the same image may appear at more than one resolution. Images are not embedded in the Markdown, only their alt text |

```python
import json

with open("en.jsonl", encoding="utf-8") as f:
    for line in f:
        a = json.loads(line)
        stack = [([a["title"]], sec) for sec in reversed(a["sections"])]
        while stack:                      # depth-first, in reading order
            path, sec = stack.pop()
            path = path + [sec["title"]]
            print(" / ".join(path))       # e.g. "Operation Flashpoint (series) / Codemasters series / Operation Flashpoint: Red River (2011)"
            stack.extend((path, sub) for sub in reversed(sec["sections"]))
```

## Things to know

- **It is 2015.** Many articles have since been rewritten, merged or deleted.
  Don't use it to answer questions about the present.
- Pages came from Wikipedia's **mobile** site, so infoboxes usually appear as
  lists inside `intro`, not as key–value fields.
- **Navigation boxes are kept.** The link grids at the foot of many articles
  end up at the end of the last section (see *References* in `samples/en.json`). Drop
  them if you want prose only.
- Image files are not included, only their URLs.
- Chinese articles keep whatever script (simplified or traditional) they were
  written in; no conversion has been applied.

## Where it comes from

The crawl was made in 2015 by **knof**, a project to turn Wikipedia into a
knowledge graph. To respect Wikipedia's one-request-per-second crawl limit,
115 small cloud machines each fetched a slice of the titles, for a total cost
of about US$55. The project never shipped, but the crawl survived, and this
is its first public release. The `v15` in the name stands for 2015.

The raw crawl (~550 GB of HTML and parsed JSON) is not distributed. If you have
it, `run_release.sh` regenerates the three files (`pip install -r
requirements.txt` first; set `SRC`, `OUT` and `CLEANER` as needed).

## License and attribution

The text comes from Wikipedia, which in 2015 was licensed under
[CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/). That license
allows adaptations to use later versions, and this dataset is released under
[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
Attribution: *Wikipedia contributors, crawled June 2015 by the knof project.*

The code in this repository is MIT-licensed (see `LICENSE`).
