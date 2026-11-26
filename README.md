# knof-v15

Cleaned and repackaged release of the **knof** Wikipedia corpus crawled in 2015.

## Background

In 2015 the knof project (Memect) crawled the full English and Chinese Wikipedias
using many short-lived cloud instances, each respecting Wikipedia's one-request-per-second
`Crawl-delay`. The rendered mobile HTML of every article was parsed by
`wikipedia_to_json.py` into a recursive "meme" JSON structure
(`__title__` / `__content__` / `__part__` / `__relation__` / …).

The raw crawl is roughly 550 GB and 13 million files across `en-html/`, `en-json/`,
`zh-html/` and `zh-json/`. This repository holds the tooling that turns the JSON
part into a clean, self-describing dataset for open release.

## What "cleaning" means

The raw meme JSON mixes content with parser bookkeeping and a few schema-level
artifacts. The cleaner (`clean_knof.py`) performs these transformations:

| Raw field / issue | Action |
|---|---|
| `__relation__[].label` (always the Chinese word for "link") | field dropped |
| `__id__` prefix `维基百科:` (used even for English articles) | rebuilt as `<lang>:<title>` |
| `__description__` parser residue (`^`, `^^`, `【References】`, `Contents :Top…` navigation text) | removed |
| `__data__.date_str` (`Last modified on …` page-footer text) | dropped |
| `__uuid__`, `__super__`, `__date__`, `__type__`, `__context__` | internal bookkeeping, not emitted |
| `__content__` and section bodies (HTML fragments) | converted to Markdown; links and images made absolute `https://`; `script`/`style`/empty anchors removed |
| Non-Latin text in the body (Japanese/Chinese names inside enwiki articles, etc.) | **kept** — this is original Wikipedia content, not noise |

## Chinese duplicates: 2015-05 vs 2015-06-02

The Chinese crawl exists in two vintages: the first pass (`zh-json/2015-05`, 4
machines) and the re-crawl (`zh-json/2015-06-02`, 21 machines). The re-crawl is
the better copy. The release merges them with the re-crawl taking priority:

```bash
# 1) full re-crawl first
python3 clean_knof.py /path/to/zh-json/2015-06-02 zh.jsonl --workers 16
# 2) first pass only where the re-crawl has no such article id
python3 clean_knof.py /path/to/zh-json/2015-05 zh-supplement.jsonl \
    --workers 16 --skip-ids zh.jsonl --skip-lang zh
```

Note that the first pass uses an older schema (no `__context__`, `__uuid__`,
`__relation__`), so it carries no links; its language is taken from the source
directory, not from the file content.

## Output schema

One JSON object per line (JSONL), UTF-8:

```json
{
  "id": "en:Yagodnoye, Magadan Oblast",
  "title": "Yagodnoye, Magadan Oblast",
  "lang": "en",
  "url": "https://en.wikipedia.org/wiki/Yagodnoye,_Magadan_Oblast",
  "description": "Yagodnoye (Russian: Я́годное) is an urban-type settlement in Magadan Oblast, Russia…",
  "intro": "Markdown lead section",
  "sections": [
    {"title": "History", "level": 1, "markdown": "…"}
  ],
  "categories": ["Urban-type settlements in Magadan Oblast", "…"],
  "links": [{"anchor": "Magadan Oblast", "url": "https://en.wikipedia.org/wiki/Magadan_Oblast"}],
  "images": ["https://upload.wikimedia.org/wikipedia/commons/thumb/…"]
}
```

- `sections` is flattened depth-first; `level` is the nesting depth in the original
  section tree (both the English and Chinese crawls are mostly depth 1).
- `description` is the plain-text abstract extracted by the 2015 parser.
- `intro` is the article lead; the Markdown body may include infobox-like tables in
  list form because the source pages are rendered mobile HTML.

## Usage

```bash
pip install -r requirements.txt

# a directory tree containing meme JSON files (*.json)
python3 clean_knof.py /path/to/en-json/2015-06-02 out.jsonl --workers 16

# quick smoke test on the first 200 files
python3 clean_knof.py /path/to/en-json/2015-06-02 out.jsonl --limit 200
```

Each input file is one article; malformed files are counted and skipped, never fatal.

## Source data

- English: `en-json/2015-06-02/` — 90 machine subdirectories, ~4.9 M articles
- Chinese: `zh-json/2015-06-02/` — 20 machine subdirectories, ~0.8 M articles (re-crawl)
- Chinese: `zh-json/2015-05/` — first-pass crawl, flat directory

The raw corpus is not part of this repository.

## License

- Code in this repository: MIT (see `LICENSE`).
- The original Wikipedia content is licensed under
  [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/);
  the cleaned dataset derived from it is released under the same license.
  Attribution: Wikipedia contributors, 2015 dumps, via the knof crawl.

## Status

Work in progress. The 2015 raw corpus is being migrated and cleaned;
release artifacts and dataset download links will be announced here.
