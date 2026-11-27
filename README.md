# knof-v15

**English and Chinese Wikipedia as of 2 June 2015, cut into about 31 million
interlinked knowledge fragments.** Every article (5.95 million) is split along
its own section tree. Each fragment is a piece of Markdown with a title, a place
in its article and a source URL, and links to other articles.

| | Articles | Fragments (lead + sections) | Size, uncompressed |
|---|---|---|---|
| English | 4,882,789 | 26.0 M | 128 GB |
| Chinese | 1,065,845 | 4.77 M | 26 GB |

> **Download:** [GitHub Releases, v1.0](https://github.com/baojie/knof-v15/releases/tag/v1.0).
> To look first, get [`knof-v15-sample.zip`](https://github.com/baojie/knof-v15/releases/download/v1.0/knof-v15-sample.zip) (11 MB).

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

| File | Content | Articles | Parts | Download |
|---|---|---|---|---|
| `knof-v15-en-2015-06` | English Wikipedia, crawled 2–22 June 2015 | 4,882,789 | 33 | 17.8 GB |
| `knof-v15-zh-2015-06` | Chinese Wikipedia, same crawl | 822,794 | 6 | 2.3 GB |
| `knof-v15-zh-2015-03` | Chinese articles missing from the June crawl, from an earlier crawl (revisions up to mid-March 2015) | 243,051 | 2 | 0.7 GB |
| `knof-v15-sample.zip` | 2,300 random articles from the three files | 2,300 | 1 | 11 MB |

The two Chinese files never share an `id`; use both for the
fullest Chinese coverage.

**Download in parts.** Each file is published as parts of at most 4 GB of JSONL
(300–600 MB compressed; 20.8 GB for everything), split between records and
compressed with zstd, for example `knof-v15-en-2015-06-00001-of-00033.jsonl.zst`. Every part is complete
on its own: decompress one and start using it while the rest download.
`SHA256SUMS` and `manifest.json` list each part's checksum and article count.

```sh
zstd -dc knof-v15-zh-2015-06-00001-of-00006.jsonl.zst | head -1
```

**Sample.** `knof-v15-sample.zip` (11 MB) holds 1,000 random English
articles, 1,000 from the June Chinese crawl and 300 from the March one, plus two
pretty-printed records, so you can see the data before downloading the rest.

## Record format

Two complete records, pretty-printed: [`samples/en.json`](samples/en.json)
(*Operation Flashpoint (series)*) and [`samples/zh.json`](samples/zh.json)
(*2012年夏季奧林匹克運動會划船男子雙人單槳比賽*). In the release files each
record is one line. Shortened:

```json
{
  "id": "en:Operation Flashpoint (series)",
  "title": "Operation Flashpoint (series)",
  "display_title": null,
  "lang": "en",
  "url": "https://en.wikipedia.org/wiki/Operation_Flashpoint_(series)",
  "uuid": "wikien-FkUXbhApfNJxN0QDZ4gD9w",
  "super": null,
  "context": "wikien",
  "last_modified": "2015-06-06T21:14:00Z",
  "description": "Operation Flashpoint is a series of military simulation games. …",
  "intro": "<table>…infobox…</table>\n\n**Operation Flashpoint** is a series of …",
  "links": [{"anchor": "Military", "url": "https://en.wikipedia.org/wiki/Military",
             "type": "link", "target_uuid": "wikien-fSlX39O4iPxAh_YMPGUJUw"}, "…"],
  "main_image": null,
  "images": [],
  "sections": [
    {"title": "Bohemia Interactive series", "uuid": "wikien-…", "markdown": "", "links": [], "images": [], "sections": [
      {"title": "Operation Flashpoint: Cold War Crisis (2001)", "uuid": "wikien-oWsVgZK0yGvE5PiwaFix5w",
       "super": ["Operation Flashpoint (series)", "Bohemia Interactive series"],
       "markdown": "Main article: [Operation Flashpoint: Cold War Crisis](https://en.wikipedia.org/wiki/…) …",
       "links": [{"anchor": "Operation Flashpoint: Cold War Crisis",
                  "url": "https://en.wikipedia.org/wiki/Operation_Flashpoint:_Cold_War_Crisis",
                  "type": "main", "target_uuid": "wikien-O0LOV4-z3RQSrsoBdZSVUw"}, "…"],
       "images": [],
       "sections": [{"title": "Expansion pack", "…": "…"}]}
    ]},
    {"title": "Codemasters series", "uuid": "wikien-…", "markdown": "", "links": [], "images": [], "sections": ["…"]},
    {"title": "References", "…": "…"}
  ],
  "categories": ["Video game franchises", "Military combat simulators", "…"]
}
```

The article and every section are nodes of the same kind: text, links,
images, child sections. The tree is the article's own section tree. A heading
that only groups subsections (*Codemasters series* above) is kept with empty
`markdown`, so the tree is never broken.

**Article fields**

| Field | Meaning |
|---|---|
| `id` | `<lang>:<title>`, unique within the dataset |
| `title` | canonical article title |
| `display_title` | the title as the page showed it, when different (lowercase first letter, Chinese script variant, leading or trailing underscores as in *_NSAKEY*…); otherwise `null` |
| `lang` | `en` or `zh` |
| `context` | the site the page was downloaded from: `wikien` (en.wikipedia.org) or `wikizh` (zh.wikipedia.org) |
| `url` | the article's Wikipedia URL (today's page, not the 2015 revision) |
| `uuid` | stable id of the article; other articles' links point to it through `target_uuid`. `null` in the 2015-03 file |
| `last_modified` | time of the revision that was crawled, UTC (`null` only for two pages that show none) |
| `description` | the lead as plain text |
| `intro` | the lead as Markdown |
| `links`, `main_image`, `images` | the lead's links and images (see below) |
| `sections` | the section tree, in reading order |
| `categories` | category names, without the `Category:` prefix; includes Wikipedia's maintenance categories (e.g. *All articles with unsourced statements*) |

**Section fields**: `title`, `uuid`, `super`, `markdown`, `links`, `images`,
`main_image` (only when present), `sections`. `super` is the list of ancestor
titles, from the article down to the parent section, as recorded in the source
(`null` for the article itself, and for every section in the 2015-03 file,
whose crawl did not record it).

**Links** (`links` of the article and of each section) are the links in that
part of the page, as recorded in 2015:

| Field | Meaning |
|---|---|
| `anchor` | the link text |
| `url` | target URL |
| `type` | `link` for an ordinary link, `main` for a "Main article:" pointer, `see_also` for a "See also:" pointer; any other label is kept as written |
| `target_uuid` | `uuid` of the target article, so links join to articles without title matching |

All links also stay inline in the Markdown, at their place in the text.
The 2015-03 file has inline links only; its `links` lists are empty.

**Images**: `main_image` is the page's lead image URL. Each entry of `images`
has `src` (the image as displayed), `file` (its File: page, when known), `alt`
and `caption`. Images inside the text stay inline as `![alt](src)`.

**Markdown.** Paragraphs, line breaks, lists, emphasis, links and images are
plain Markdown. A few things Markdown cannot express are kept as inline HTML so
nothing is lost: superscript and subscript (`m<sup>2</sup>`), line breaks
inside table cells (`<br>`), and emphasis glued to a word. Tables with one
header row and no merged cells are Markdown tables; all others (infoboxes,
tables with merged cells or lists inside cells) are kept as HTML with only
their structure attributes, so no row, column or cell boundary is lost.

```python
import json

with open("knof-v15-en-2015-06.jsonl", encoding="utf-8") as f:
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
- Infoboxes are kept as HTML tables at the start of `intro`.
- **Navigation boxes are kept.** The link grids at the foot of many articles
  end up at the end of the last section (see *References* in `samples/en.json`). Drop
  them if you want prose only.
- Image files are not included, only their URLs.
- The crawl fetched about a thousand pages twice, under a redirect or a variant
  name. Each `id` appears once; where the two fetches differ, the later
  revision is kept.
- Chinese articles keep whatever script (simplified or traditional) they were
  written in; no conversion has been applied.

## Where it comes from

The crawl was made in 2015 by **knof**, a project to turn Wikipedia into a
knowledge graph. To respect Wikipedia's one-request-per-second crawl limit,
115 small cloud machines each fetched a slice of the titles, for a total cost
of about US$55. The project never shipped, but the crawl survived, and this
is its first public release. The `v15` in the name stands for 2015.

The raw crawl (~550 GB of HTML and parsed JSON) is not distributed.

## License and attribution

The text comes from Wikipedia, which in 2015 was licensed under
[CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/). That license
allows adaptations to use later versions, and this dataset is released under
[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
Attribution: *Wikipedia contributors, crawled June 2015 by the knof project.*
