---
name: everything-query
description: "Build and run complex Everything file searches on Windows: names, paths, dates, content, duplicates, and metadata, with version-aware ES arguments."
---

# Everything Query

Check `es.exe -version` and `es.exe -get-everything-version`. Shared examples target 1.4/1.5; marked features need 1.5. For older builds, consult installed **Help > Search Syntax**. Use lowercase function names for 1.4 compatibility.

## Parsing

Space = AND, `|` = OR, `!` = NOT, `<...>` = grouping. **OR binds before AND by default**; write `<red blue>|green` for `(red AND blue) OR green`. Quotes preserve spaces; they do not imply whole-filename matching. Local modifiers override saved matching options; use `nopath:` when filename-only matching matters.

Use `regex:` on individual terms; global regex mode disables normal Everything operators/functions. A `regex:` term consumes `|` as regex alternation. Put modifiers before functions: `regex:content:`, not `content:regex:`. [Syntax](https://www.voidtools.com/support/everything/search_syntax/) / [matching](https://www.voidtools.com/support/everything/searching/).

## Shared searches

| Intent | Query fragment |
|---|---|
| Types | `file: ext:pdf;docx` / `folder:` |
| Whole filename | `nopath:wfn:"Release Notes.md"` |
| Filename pattern | `nopath:regex:^report-\d{4}\.pdf$` |
| Subtree / direct children | `"D:\Work Files\"` / `parent:"D:\Work Files"` |
| Exclude a directory segment | `!path:\node_modules\` |
| Folders with / without matching direct children | `folder: child:*.flac !child:*.jpg` |
| Direct child count / absolute path depth | `folder: childcount:>100` / `depth:>10` |
| Empty folders / zero-byte files | `empty:` / `file: size:0` |
| Modified dates | `dm:today` / `dm:7days` / `dm:2026-08-01..2026-08-31` |
| Created / accessed dates | `dc:2026-08` / `da:2026-08` |
| Size | `size:>500mb` / `size:100mb..1gb` |
| Exclude hidden or system items | `!attrib:h !attrib:s` |
| Extracted text / raw UTF-8 text | `content:"build failed"` / `utf8content:"build failed"` |

A trailing `\` prevents a subtree prefix from matching similarly named neighbors. `empty:` and child counts describe the index, not necessarily disk contents. Attribute letters combine with AND: `attrib:hs` requires both flags. Scope content searches by path/type first; in 1.4, put content terms last. `content:` uses an associated iFilter; `utf8content:` reads raw text. [1.4 content behavior](https://www.voidtools.com/forum/viewtopic.php?t=5936).

## 1.5 searches

| Intent | Query fragment |
|---|---|
| Match directory part only | `pathpart:archive` |
| Folders containing matches at any depth | `folder: descendant:*.pdf` |
| Videos missing same-stem subtitles | `file: ext:mp4 !fileexists:$stem:.srt` |
| Exact path list | `filelist:"D:\one.txt";"D:\two.txt"` |
| Ignore punctuation and spaces | `nopunc:nows:projectfinal` |
| Full / initial Pinyin (1.5.0.1387+) | `pinyin-quan:wenjian` / `pinyin-jian:wj` |
| Duration / exact duration | `length:50secs..70secs` / `length:=60secs` |
| Dimensions / orientation / frame rate | `width:3840 height:2160` / `aspect-ratio:portrait` / `frame-rate:59..61` |
| Photo capture / media encoding dates | `date-taken:2026-08` / `date-encoded:2026-08` |
| Document metadata | `title:invoice` / `authors:Smith` / `page-count:>20` |
| Audio metadata | `artist:Abba` / `album:Gold` |

`$stem:` substitutes each candidate's name without its extension; `fileexists:` resolves relative paths from that candidate's folder. For `filelist:`, 1.4 requires `|` separators instead of `;`. Under default 1.5 path wildcards, `*` stays within a path component; `**` crosses separators. [Functions](https://www.voidtools.com/support/everything/search_functions/) / [1.4 list differences](https://www.voidtools.com/forum/viewtopic.php?t=12220) / [Pinyin builds](https://www.voidtools.com/forum/viewtopic.php?t=12073).

Numeric units affect precision: `length:1min` also matches 70.5 seconds; use an explicit range or `:=`. Encoding dates may reflect export rather than capture. Unknown properties use `length:unknown`; scoped `notindexed:length:50secs..70secs` reads unindexed values on demand. Property indexing: **Tools > Options > Properties > Add**. [Properties](https://www.voidtools.com/support/everything/properties/).

## Duplicates

- **1.4:** `dupe:` compares names; `sizedupe:` compares sizes across the **entire index**, even alongside a folder filter. Combining them does not establish a matching name-size pair. For scoped comparison, export filtered results as EFU and open that file list first. [1.4 behavior](https://www.voidtools.com/forum/viewtopic.php?p=48458).
- **1.5:** `dupe:name;size` compares pairs within current results; `dupe:size;sha256` hashes only equal-size candidates. `distinct:name` keeps one per name; `unique:name` keeps only names without duplicates. Same size alone is not content equality; hash predicates read files. Press F5 to include newly appearing items in a GUI duplicate view. [Duplicates](https://www.voidtools.com/support/everything/find_duplicates/).

## ES execution

Pass separate search terms as separate native arguments; putting the whole GUI expression into one argument can return no matches. Use `-path` / `-parent` for paths, preserving phrases as single terms. Build arguments from conditions, not a blind space split.

```powershell
& es.exe -path 'D:\Work Files' -n 20 -sort date-modified-descending 'file:' 'ext:pdf;docx' 'dm:7days'
```

Check installed `-help` for output support: `-csv`, `-json`, `-export-csv <file>`, `-add-columns <properties>`. Rich columns require 1.5; JSON and `-argv` also depend on ES version. In ES 1.1.0.37 with Everything 1.5.0.1423, compact OR `'<dimensions:3840x2160|dimensions:2160x3840>'` works; splitting a multi-term group across native arguments did not.

- JSON `length` is in 100-nanosecond units: divide by `10000000.0`; preserve null. `frame_rate` is fps.
- `-date-format 1` = local ISO-style timestamps; `3` = UTC; `-size-format 1` = bytes.
- `-n` limits results, not reads required by content/property/hash predicates. `-get-result-count` requests a count when supported.
- Empty stdout with exit 0 can mean no matches. Exit 6 = unsupported option; exit 8 = unreachable IPC endpoint. Check the desktop client, `-instance <name>`, and desktop/sandbox session access; the indexing service alone is not the query endpoint.

Use installed help for ES option differences and the official [CLI](https://www.voidtools.com/support/everything/command_line_interface/), [modifiers](https://www.voidtools.com/support/everything/search_modifiers/), and function reference for unlisted searches.
