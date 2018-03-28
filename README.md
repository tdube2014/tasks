# tasks
# markdown-toc [![NPM version](https://img.shields.io/npm/v/markdown-toc.svg?style=flat)](https://www.npmjs.com/package/markdown-toc) [![NPM monthly downloads](https://img.shields.io/npm/dm/markdown-toc.svg?style=flat)](https://npmjs.org/package/markdown-toc)  [![NPM total downloads](https://img.shields.io/npm/dt/markdown-toc.svg?style=flat)](https://npmjs.org/package/markdown-toc) [![Linux Build Status](https://img.shields.io/travis/jonschlinkert/markdown-toc.svg?style=flat&label=Travis)](https://travis-ci.org/jonschlinkert/markdown-toc) [![Windows Build Status](https://img.shields.io/appveyor/ci/jonschlinkert/markdown-toc.svg?style=flat&label=AppVeyor)](https://ci.appveyor.com/project/jonschlinkert/markdown-toc)

> Generate a markdown TOC (table of contents) with Remarkable.

## Table of Contents

- [Install](#install)
- [CLI](#cli)
- [Highlights](#highlights)
- [Usage](#usage)
- [API](#api)
  * [toc.plugin](#tocplugin)
  * [toc.json](#tocjson)
  * [toc.insert](#tocinsert)
  * [Utility functions](#utility-functions)
- [Options](#options)
  * [options.append](#optionsappend)
  * [options.filter](#optionsfilter)
  * [options.slugify](#optionsslugify)
  * [options.bullets](#optionsbullets)
  * [options.maxdepth](#optionsmaxdepth)
  * [options.firsth1](#optionsfirsth1)
  * [options.stripHeadingTags](#optionsstripheadingtags)
- [About](#about)

_(TOC generated by [verb](https://github.com/verbose/verb) using [markdown-toc](https://github.com/jonschlinkert/markdown-toc))_

## Install

Install with [npm](https://www.npmjs.com/):

```sh
$ npm install --save markdown-toc
```

## CLI

```
Usage: markdown-toc [options] <input>

  input:        The Markdown file to parse for table of contents,
                or "-" to read from stdin.

  -i:           Edit the <input> file directly, injecting the TOC at <!-- toc -->;
                (Without this flag, the default is to print the TOC to stdout.)

  --json:       Print the TOC in JSON format

  --append:     Append a string to the end of the TOC

  --bullets:    Bullets to use for items in the generated TOC
                (Supports multiple bullets: --bullets "*" --bullets "-" --bullets "+")
                (Default is "*".)

  --maxdepth:   Use headings whose depth is at most maxdepth
                (Default is 6.)

  --no-firsth1: Include the first h1-level heading in a file

  --no-stripHeadingTags: Do not strip extraneous HTML tags from heading
                         text before slugifying
```

## Highlights

**Features**

* Can optionally be used as a [remarkable](https://github.com/jonschlinkert/remarkable) plugin
* Returns an object with the rendered TOC (on `content`), as well as a `json` property with the raw TOC object, so you can generate your own TOC using templates or however you want
* Works with [repeated headings](https://gist.github.com/jonschlinkert/ac5d8122bfaaa394f896)
* Uses sane defaults, so no customization is necessary, but you can if you need to.
* [filter](#filter-headings) out headings you don't want
* [Improve](#titleize) the headings you do want
* Use a custom [slugify](#optionsslugify) function to change how links are created

**Safe!**

* Won't mangle markdown in code examples in gfm code blocks that other TOC generators mistake as being actual headings (this happens when markdown headings are show in _examples_, meaning they arent' actually headings that should be in the toc. Also happens with yaml and coffee-script comments, or any comments that use `#`)
* Won't mangle front-matter, or mistake front-matter properties for headings like other TOC generators

## Usage

```js
var toc = require('markdown-toc');

toc('# One\n\n# Two').content;
// Results in:
// - [One](#one)
// - [Two](#two)
```

To allow customization of the output, an object is returned with the following properties:

* `content` **{String}**: The generated table of contents. Unless you want to customize rendering, this is all you need.
* `highest` **{Number}**: The highest level heading found. This is used to adjust indentation.
* `tokens` **{Array}**: Headings tokens that can be used for custom rendering

## API

### toc.plugin

Use as a [remarkable](https://github.com/jonschlinkert/remarkable) plugin.

```js
var Remarkable = require('remarkable');
var toc = require('markdown-toc');

function render(str, options) {
  return new Remarkable()
    .use(toc.plugin(options)) // <= register the plugin
    .render(str);
}
```

**Usage example**

```js
var results = render('# AAA\n# BBB\n# CCC\nfoo\nbar\nbaz');
```

Results in:

```
- [AAA](#aaa)
- [BBB](#bbb)
- [CCC](#ccc)
```

### toc.json

Object for creating a custom TOC.

```js
toc('# AAA\n## BBB\n### CCC\nfoo').json;

// results in
[ { content: 'AAA', slug: 'aaa', lvl: 1 },
  { content: 'BBB', slug: 'bbb', lvl: 2 },
  { content: 'CCC', slug: 'ccc', lvl: 3 } ]
```

### toc.insert

Insert a table of contents immediately after an _opening_ `<!-- toc -->` code comment, or replace an existing TOC if both an _opening_ comment and a _closing_ comment (`<!-- tocstop -->`) are found.

_(This strategy works well since code comments in markdown are hidden when viewed as HTML, like when viewing a README on GitHub README for example)._

**Example**

```
<!-- toc -->
- old toc 1
- old toc 2
- old toc 3
<!-- tocstop -->

## abc
This is a b c.

## xyz
This is x y z.
```

Would result in something like:
