# HTML

Course 1 of 15. Next: [02-css.md](02-css.md).

HTML describes the structure of a page. The browser builds a tree from the tags and shows that tree. Colors, columns, and animation are CSS. Behavior is JavaScript.

Every section has the same parts: the concept, the terms, the elements and attributes, several cases with what the page does, common mistakes, a tip, and a practice with a solution.

## Contents

1. [A document](#1-a-document)
2. [Elements and attributes](#2-elements-and-attributes)
3. [Text](#3-text)
4. [Links](#4-links)
5. [Images](#5-images)
6. [Lists](#6-lists)
7. [Tables](#7-tables)
8. [Class, id, div, and span](#8-class-id-div-and-span)
9. [Block and inline](#9-block-and-inline)
10. [Forms](#10-forms)
11. [Semantic layout](#11-semantic-layout)
12. [Media and embeds](#12-media-and-embeds)
13. [The head](#13-the-head)
14. [Characters, comments, and paths](#14-characters-comments-and-paths)
15. [A full page](#15-a-full-page)

## 1. A document

### Concept

Every page is one tree. `<html>` is the root. `<head>` holds information about the page. `<body>` holds what the reader sees. A tag that opens inside a parent must close before that parent closes. `<!DOCTYPE html>` tells the browser this is modern HTML, so it does not fall into an old layout mode.

### Terms

| Piece | Role |
| --- | --- |
| `<!DOCTYPE html>` | Modern HTML, not an old dialect |
| `<html lang="en">` | The root. `lang` tells assistive tools which language the text is |
| `<head>` | Information about the page. Not the article |
| `<body>` | What the reader sees |
| `<title>` | The tab name and the default bookmark text |
| Parent | The element that contains another |
| Sibling | Two elements with the same parent |
| `charset="utf-8"` | How to read the bytes as letters |
| `viewport` | Phones use the device width instead of a fake desktop width |

### Properties

| Tag or attribute | What it changes |
| --- | --- |
| `<meta charset="utf-8">` | Accents and symbols decode correctly |
| `<meta name="viewport" content="width=device-width, initial-scale=1">` | The layout width matches the phone |
| `<title>` | Text in the browser tab |
| `lang` | Language of the page, or of one subtree |

### Examples

**Case: a complete small page**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Homes</title>
  </head>
  <body>
    <h1>Homes</h1>
    <p>Active listings.</p>
  </body>
</html>
```

Result: the tab says `Homes`. The page shows a heading and one paragraph. `h1` and `p` are siblings. `body` is their parent.

Why: the doctype, the charset, and the viewport are not visible text. They change how the browser reads and sizes the page.

**Case: a tag closed in the wrong order**

```html
<body>
  <p>Active <strong>listings.</p></strong>
</body>
```

Result: the browser rewrites the tree. The strong text may end at the `</p>`, and the stray `</strong>` is ignored. The outline you typed is not the outline the browser kept.

Why: the inner tag must close before the outer tag. The repair is ` <p>Active <strong>listings.</strong></p>`.

**Case: no `lang`**

```html
<html>
  <head><meta charset="utf-8"><title>Homes</title></head>
  <body><h1>Homes</h1></body>
</html>
```

Result: the page still shows `Homes`. A screen reader has to guess the language.

Why: `lang="en"` is the hint. It does not translate the page. It tells the tool how to pronounce it.

**Case: a missing charset with a non-ASCII character**

```html
<p>Café</p>
```

Saved as UTF-8, but the head has no `charset`.

Result: some browsers show `Café`. An older guess can show `CafÃ©`.

Why: the bytes of `é` are only unambiguous when the file says `charset="utf-8"`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| No doctype | The browser may use old layout rules. Widths and centering shift | Start with `<!DOCTYPE html>` |
| Title typed in the body | The tab stays blank or shows the URL. The text appears in the page | Put `<title>` in `<head>` |
| Closing the parent before the child | The browser repairs the tree in a way you did not write | Close the inner tag first |
| `lang` left off | The page looks the same. Pronunciation is a guess | `lang` on `<html>` |

### Tip

Draw the tree before you debug a layout. If you cannot name the parent of an element, the tags are crossed.

### Practice

Write a page whose tab is `Irvine`, whose language is English, and whose body is one heading `Irvine` and one paragraph `3 beds`.

### Solution

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Irvine</title>
  </head>
  <body>
    <h1>Irvine</h1>
    <p>3 beds</p>
  </body>
</html>
```

Result: the tab says `Irvine`. The page shows the heading and the paragraph.

## 2. Elements and attributes

### Concept

An element is a start tag, content, and an end tag. A void element has no end tag and no children. An attribute lives on the start tag and changes that element. Boolean attributes are true when they are present. Their value is not the word `true`.

`class`, `id`, and `data-*` are the three labels that get mixed up.

### Terms

| Term | Meaning |
| --- | --- |
| Normal element | Has an end tag. `p`, `h1`, `a`, `li` |
| Void element | No end tag and no children. `img`, `br`, `hr`, `input`, `meta`, `link` |
| Attribute | A name and a value on the start tag |
| Boolean attribute | Present means true. `checked`, `disabled`, `required`, `multiple`, `hidden` |
| Global attribute | Works on almost every element |

### Properties

| Attribute | Meaning |
| --- | --- |
| `class` | One or more labels. May repeat on the page. CSS hook is `.card` |
| `id` | One unique label. CSS hook is `#report`. The URL can end in `#report` |
| `title` | Extra text, often a tooltip. Do not hide required instructions here |
| `hidden` | Not shown, and skipped by the accessibility tree |
| `lang` | Language of this subtree when it differs from the page |
| `tabindex="0"` | Adds the element to the natural Tab order. Avoid positive numbers |
| `data-*` | Your own data. JavaScript reads it as `dataset` |

| | `class` | `id` | `data-*` |
| --- | --- | --- | --- |
| May repeat on the page | Yes | No | Yes |
| CSS hook | `.city` | `#report` | `[data-city="Irvine"]` |
| JavaScript hook | `querySelectorAll(".city")` | `getElementById("report")` | `element.dataset.city` |
| Shows up in the URL | No | Yes, as `#report` | No |
| Several on one element | `class="card featured"` | One `id` | Several `data-` names |

### Examples

**Case: a normal element and a void element**

```html
<a href="https://example.com">Example</a>
<img src="house.jpg" alt="Front of a one-story house">
```

Result: the word `Example` is a link. The image has no inside and no `</img>`. Writing `</img>` is ignored.

Why: `a` wraps content, so it needs an end tag. `img` is void. Its information is in the attributes.

**Case: a boolean attribute**

```html
<input type="checkbox" name="pool" checked>
<input type="checkbox" name="garage">
```

Result: the pool box is checked when the page loads. The garage box is not. `checked="false"` would still check the box, because the attribute is present.

Why: for a boolean attribute, presence is true and absence is false. The value is not consulted the way a Java boolean is.

**Case: two classes on one element**

```html
<p class="card featured" id="listing-42" data-city="Irvine">Irvine</p>
```

Result: the paragraph is both `card` and `featured`. Its id is `listing-42`. Script can read `dataset.city` as `Irvine`. A second element with `id="listing-42"` makes `getElementById` and the URL fragment ambiguous.

Why: a class is a label you may reuse. An id is an address, so it has to be unique. `data-city` is neither a class nor an id. It is data you attached.

**Case: `hidden`**

```html
<p hidden>Internal note</p>
<p>Public note</p>
```

Result: only `Public note` is visible. The first paragraph is not in the accessibility tree either.

Why: `hidden` is stronger than a CSS color trick. Do not use it for text the reader must see.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `checked="false"` | The box is still checked | Remove the attribute |
| Two elements with the same `id` | Fragments and `getElementById` hit one of them, not both | Use a class when the label repeats |
| Instructions only in `title` | Touch screens often never show the tooltip | Put the instruction in the page |
| `</img>` or text inside `img` | The text is not the image's content | Use `alt` |

### Tip

If you can imagine two of them on the same page, it is a class. If a URL should jump to that one element, it is an id.

### Practice

Mark up a paragraph that is a featured card, has the unique id `listing-7`, and stores the city `Tustin` for script. The visible text is `Tustin`.

### Solution

```html
<p class="card featured" id="listing-7" data-city="Tustin">Tustin</p>
```

Result: one paragraph, two classes, one id, and `dataset.city` equal to `Tustin`.

## 3. Text

### Concept

Headings describe the outline. Use one `h1`. Do not pick `h3` because it looks smaller. Size is CSS. `strong` means important. `b` is only bold. `em` means stress. `i` is only italic. White space between tags collapses into one space. A blank line in the file does not make a blank line on the page.

### Terms

| Term | Meaning |
| --- | --- |
| Outline | The heading levels, not the font sizes |
| Collapsed white space | Newlines and repeated spaces become one space |
| Thematic break | `hr`. A change of topic, not a horizontal rule you style first |
| Preformatted | `pre` keeps the spaces and line breaks you typed |

### Properties

| Element | Use |
| --- | --- |
| `h1`–`h6` | Outline levels. Do not skip from `h1` to `h4` |
| `p` | A paragraph |
| `br` | A line break inside a paragraph, such as an address |
| `hr` | A thematic break |
| `strong` / `b` | Important, or bold with no extra importance |
| `em` / `i` | Stress, or italic with no extra stress |
| `mark` | A highlight, such as a search hit |
| `small` | A side comment, such as legal fine print |
| `sub` / `sup` | Subscript and superscript |
| `code` | A short piece of code in a sentence |
| `pre` | Keep spaces and line breaks |
| `blockquote` | A quotation from somewhere else |
| `q` | A short quotation inside a sentence |
| `abbr` | An abbreviation. Put the expansion in `title` |
| `del` / `ins` | Deleted and inserted text |
| `wbr` | A place the browser may break a long word |

### Examples

**Case: importance versus a bold look**

```html
<p>The median close is <strong>$1,200,000</strong>.</p>
<p>The street name is <b>Oak</b>.</p>
```

Result: both phrases look bold in a typical browser. A screen reader can treat `strong` as important and `b` as just a different voice.

Why: pick the element for the meaning. CSS can bold either one later.

**Case: white space collapses**

```html
<p>Line one
   Line two</p>
<p>Line one<br>Line two</p>
```

Result: the first paragraph is `Line one Line two` on one line. The second paragraph breaks after `Line one`.

Why: a newline in the file is white space. `br` is an element that asks for a break. Two paragraphs are two `p` elements, not two `br` elements.

**Case: a skipped heading level**

```html
<h1>Market report</h1>
<h4>Irvine</h4>
```

Result: `Irvine` looks small. The outline jumps from level 1 to level 4.

Why: the level is the structure. If Irvine is the next section of the report, it is an `h2`. Make it smaller with CSS if the design asks for that.

**Case: code keeps its spaces only inside `pre`**

```html
<p><code>SELECT City FROM sold;</code></p>
<pre><code>SELECT City
FROM sold;</code></pre>
```

Result: the paragraph shows the query on one line. The `pre` block shows the line break you typed.

Why: `code` marks the text as code. `pre` is what preserves the formatting.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Choosing `h3` for a smaller font | The outline lies | Use the next heading level, then CSS |
| Blank lines in the file used as paragraphs | The page shows one space | Separate `p` elements |
| `br` after every sentence | A stack of line breaks, not a document | `p` for paragraphs |
| `b` where the words are important | The importance is lost | `strong` |

### Tip

Read the page with CSS turned off. The heading sizes should still tell the outline. If they do not, the levels are wrong.

### Practice

Mark a report title `Market report`, a section title `Irvine`, and a sentence whose price `$999,000` is important.

### Solution

```html
<h1>Market report</h1>
<h2>Irvine</h2>
<p>The price is <strong>$999,000</strong>.</p>
```

Result: one top heading, one section heading, and the price marked important.

## 4. Links

### Concept

`href` is the destination. Without `href`, an `a` is a placeholder, not a link. The visible text should say where the link goes. A link can wrap text or an image. `target="_blank"` opens a new tab. Pair it with `rel="noopener noreferrer"` so the new page cannot reach back and script yours.

### Terms

| Term | Meaning |
| --- | --- |
| Relative URL | Resolved from the current page |
| Root URL | Starts with `/`. Resolved from the site root |
| Fragment | Starts with `#`. An element on this page |
| `mailto:` / `tel:` | The mail app or the phone app |
| Placeholder anchor | An `a` with no `href` |

### Properties

| `href` shape | Goes to |
| --- | --- |
| `reports/irvine.html` | A file relative to the current page |
| `/reports/irvine.html` | A file from the site root |
| `../index.html` | Up one folder |
| `#dom` | An element with `id="dom"` on this page |
| `https://example.com` | Another site |
| `mailto:hello@example.com` | The mail app |
| `tel:+15551212` | The phone app |

| Attribute | When |
| --- | --- |
| `target="_blank"` | A new tab. Pair it with `rel="noopener noreferrer"` |
| `download` | Suggest a download instead of showing the file |
| `rel` | The relationship. `noopener` stops the new page from scripting yours |

### Examples

**Case: four kinds of destination**

```html
<a href="reports/irvine.html">Irvine report</a>
<a href="reports/irvine.html#dom">Days on market</a>
<a href="mailto:hello@example.com">Email</a>
<a href="#top">Back to top</a>
```

Result: the first link opens another file. The second opens that file and scrolls to `id="dom"`. The third opens the mail app. The fourth scrolls to `id="top"` on this page.

Why: the shape of `href` is the whole decision. The element is `a` in every case.

**Case: a new tab**

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">Outside site</a>
```

Result: the current page stays. `example.com` opens in a new tab. Your page's `window.opener` is not available to that site.

Why: `target="_blank"` alone used to let the new page navigate your tab. `noopener` closes that hole.

**Case: "click here"**

```html
<a href="reports/irvine.html">Click here</a>
```

Result: the link works. A list of links read out of context is "Click here", "Click here".

Why: the visible words are the name of the destination. `Irvine report` still points at the same `href`.

**Case: an anchor with no `href`**

```html
<a>Irvine report</a>
```

Result: the words are not a link. Tab does not stop there. The browser does not navigate.

Why: `a` without `href` is a placeholder you might turn into a link later. It is not a broken link. It is not a link yet.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `target="_blank"` without `rel` | The new page may be able to script the old one | Add `rel="noopener noreferrer"` |
| Link text `click here` | The destination is unnamed | Name the page |
| `#dom` when nothing has `id="dom"` | The click does nothing useful | Match the id |
| A root path `/reports/...` on a page that lives in a subfolder of a project site | The link jumps to the host root, not the project folder | Use a relative path, or know the real root |

### Tip

Hover the link and read the status bar. The text you see and the URL you are about to open are two different things, and both have to be right.

### Practice

Link the words `Irvine report` to `reports/irvine.html`, and add a second link `Days on market` that goes to the element `dom` on that same page.

### Solution

```html
<a href="reports/irvine.html">Irvine report</a>
<a href="reports/irvine.html#dom">Days on market</a>
```

Result: the first opens the report at the top. The second opens it scrolled to `id="dom"`.

## 5. Images

### Concept

`src` is the file. `alt` is the replacement text: what a screen reader reads, and what remains if the file fails to load. Decorative images use `alt=""`, which is different from leaving `alt` off. `width` and `height` reserve the box so the page does not jump when the file arrives. The caption in `figcaption` is extra. The `alt` still has to stand alone.

### Terms

| Term | Meaning |
| --- | --- |
| Replaced element | The file paints inside the element's box |
| Informative image | Needs a real `alt` |
| Decorative image | `alt=""` so assistive tools skip it |
| `figure` | Groups the image and its caption |
| `picture` | Chooses a file. The `img` inside is still the fallback |

### Properties

| Attribute | Meaning |
| --- | --- |
| `src` | The file |
| `alt` | Replacement text |
| `width` and `height` | The layout size |
| `srcset` and `sizes` | Alternate files for different screen widths |
| `loading="lazy"` | Wait to fetch images far down the page |
| `decoding="async"` | Decode off the main painting work |

### Examples

**Case: an informative image**

```html
<img src="photos/house.jpg" alt="White house with a red door" width="800" height="600">
```

Result: the photo appears in an 800 by 600 box. If the file 404s, the reader still gets "White house with a red door".

Why: `alt` replaces the image. It is not a caption and not a tooltip.

**Case: decorative versus forgotten**

```html
<img src="swirl.png" alt="">
<img src="swirl.png">
```

Result: the first image is skipped by a screen reader. The second is announced as the file name, or as "image", because the text was forgotten.

Why: empty `alt` means "this image adds no information". A missing `alt` means "nobody wrote the text".

**Case: a caption that is not a substitute for `alt`**

```html
<figure>
  <img src="photos/chart.png" alt="Bar chart of median price by month">
  <figcaption>Median close price, last 12 months.</figcaption>
</figure>
```

Result: the chart, then the caption under it. A screen reader still reads the `alt` when it reaches the image.

Why: the caption is for everyone who can see the chart. The `alt` has to make sense if the chart never loads.

**Case: `picture` picks a file, `img` is required**

```html
<picture>
  <source srcset="banner-wide.jpg" media="(min-width: 800px)">
  <img src="banner-narrow.jpg" alt="Row of houses at sunset">
</picture>
```

Result: a viewport at least 800 pixels wide uses the wide file. A narrower viewport uses `banner-narrow.jpg`. The `alt` is on the `img` in both cases.

Why: `source` only offers a candidate. The `img` is the element that displays and the element that carries `alt`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `alt` left off | The file name is announced | Write the text, or `alt=""` if it is decorative |
| Alt text `image` or `photo.jpg` | The reader learns nothing | Describe the information in the picture |
| No `width` or `height` | The layout jumps when the file arrives | Set both to the image's real ratio |
| Caption used instead of `alt` | The image itself has no replacement text | Write both |

### Tip

Cover the image with your hand and read the `alt`. If you would be lost, the text is too thin.

### Practice

Add an image `photos/condo.jpg` of a condo balcony, 640 by 480, with a caption `North-facing balcony`.

### Solution

```html
<figure>
  <img src="photos/condo.jpg" alt="Condo balcony with a chair" width="640" height="480">
  <figcaption>North-facing balcony.</figcaption>
</figure>
```

Result: the photo, the reserved box, and the caption. The alt still describes the picture if the file fails.

## 6. Lists

### Concept

A list is a group of items. `ul` is unordered. `ol` is ordered. `dl` is a set of names and descriptions. `li` is valid only inside `ul` or `ol`. Nest a list by putting the inner list inside an `li`, not between two `li` elements.

### Terms

| Term | Meaning |
| --- | --- |
| `ul` | Items with no order |
| `ol` | Items with an order |
| `li` | One item of a `ul` or `ol` |
| `dl` | A description list |
| `dt` | The name |
| `dd` | The description. One `dt` may have several `dd` elements |

### Properties

| Attribute | Effect |
| --- | --- |
| `start` | The first number of an `ol` |
| `reversed` | Count down |
| `type` | `1`, `A`, `a`, `I`, or `i`. Prefer CSS when you only want a look |

### Examples

**Case: unordered versus ordered**

```html
<ul>
  <li>Active</li>
  <li>Pending</li>
</ul>
<ol>
  <li>Sign in</li>
  <li>Search</li>
</ol>
```

Result: bullets for the statuses, then the numbers 1 and 2 for the steps.

Why: the element says whether order matters. CSS can change the marker later.

**Case: `start` and `reversed`**

```html
<ol start="3" reversed>
  <li>Third step</li>
  <li>Second step</li>
</ol>
```

Result: the markers are 3, then 2.

Why: `start` sets the first marker. `reversed` walks downward from that start. The document order of the `li` elements does not change.

**Case: a nested list in the wrong place**

```html
<ul>
  <li>California</li>
  <ul>
    <li>Irvine</li>
  </ul>
</ul>
```

Result: the browser repairs this. The inner list may get wrapped into a generated item, and the outline is no longer the one you typed.

Why: the inner `ul` has to be inside the `li` for California:

```html
<ul>
  <li>California
    <ul>
      <li>Irvine</li>
    </ul>
  </li>
</ul>
```

**Case: names and descriptions**

```html
<dl>
  <dt>DOM</dt>
  <dd>Days on market.</dd>
  <dt>HOA</dt>
  <dd>Monthly association fee.</dd>
</dl>
```

Result: two terms, each followed by its description. They are not bullets and they are not a table.

Why: use `dl` when each item is a name plus an explanation. Use `ul` when the items are peers.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `li` outside a list | The browser invents a list, or the item is invalid | Put it in `ul` or `ol` |
| Nested list between items | The tree is repaired | Put the inner list inside the parent `li` |
| `ol` for a set of filters with no order | Numbers imply a sequence | `ul` |
| A description list used as a two-column layout | The meaning is "term and definition", and the layout is fragile | A table for data, or CSS for layout |

### Tip

If you would say "first, second, third" while reading the items, it is an `ol`. If you would not, it is a `ul`.

### Practice

Make an ordered list of two steps, `Sign in` and `Search`, starting at 1. Under `Search`, nest an unordered list with `By city`.

### Solution

```html
<ol>
  <li>Sign in</li>
  <li>Search
    <ul>
      <li>By city</li>
    </ul>
  </li>
</ol>
```

Result: markers 1 and 2. `By city` is a bullet nested under step 2.

## 7. Tables

### Concept

A table is data in rows and columns. It is not a page layout. CSS grid does layout. Each row should cover the same number of columns once you count `colspan`. Header cells use `th` and say whether they label a column or a row. `caption` is the title of the table.

### Terms

| Term | Meaning |
| --- | --- |
| `caption` | The title. First child of `table` |
| `thead` / `tbody` / `tfoot` | Header rows, data rows, summary rows |
| `tr` | One row |
| `th` | A header cell |
| `td` | A data cell |
| `scope` | Which cells this header labels |
| `colspan` / `rowspan` | How many columns or rows this cell covers |

### Properties

| Attribute | Meaning |
| --- | --- |
| `scope="col"` | This header labels the column |
| `scope="row"` | This header labels the row |
| `colspan="2"` | This cell covers two columns |
| `rowspan="2"` | This cell covers two rows |
| `headers` | On a complex cell, the ids of the headers that label it |

### Examples

**Case: a table with a column header and a row header**

```html
<table>
  <caption>Sales in Irvine, 2025</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Sales</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">January</th>
      <td>42</td>
    </tr>
  </tbody>
</table>
```

Result: a titled grid. January is the header of its row. Sales is the header of its column. A screen reader can announce "January, Sales, 42".

Why: `td` would show the same numbers and would not name the row.

**Case: `colspan` has to keep the row rectangular**

```html
<table>
  <tr>
    <th scope="col">Month</th>
    <th scope="col">Sales</th>
    <th scope="col">Pending</th>
  </tr>
  <tr>
    <th scope="row">Total</th>
    <td colspan="2">80</td>
  </tr>
</table>
```

Result: the total cell stretches under Sales and Pending. The row still accounts for three columns: one header plus a cell of width 2.

Why: a data row with two plain `td` elements under three headers leaves a hole. `colspan` is how one value covers two columns.

**Case: layout that is not a table**

```html
<table>
  <tr>
    <td><nav>Home</nav></td>
    <td><p>Article text</p></td>
  </tr>
</table>
```

Result: the page can look like two columns. The meaning is "a data row whose cells are a nav and a paragraph", which is false.

Why: use CSS grid or flex for the page. Keep `table` for data that still makes sense as a spreadsheet.

**Case: `caption` is not a heading above the table**

```html
<h2>Sales</h2>
<table>
  <caption>Sales in Irvine, 2025</caption>
  <tr><th scope="col">Month</th><th scope="col">Sales</th></tr>
  <tr><td>January</td><td>42</td></tr>
</table>
```

Result: the page shows a heading and, with the caption, a second title tied to the table.

Why: the `h2` is the section outline. The `caption` is the name of this grid. Use the caption when the table needs a name of its own.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Page layout built as a table | The reading order follows cells, not the article | CSS for layout |
| Header cells written as `td` | The numbers have no announced label | `th` and `scope` |
| `colspan` that does not add up | A ragged grid | Count the columns in every row |
| `caption` placed after the rows | Some browsers move it. The spec wants it first | Put `caption` first inside `table` |

### Tip

Count the columns with your finger, treating `colspan="2"` as two. Every row should reach the same count.

### Practice

Make a one-row table whose caption is `January sales`, with column headers `City` and `Sales`, and one data row `Irvine`, `42`.

### Solution

```html
<table>
  <caption>January sales</caption>
  <thead>
    <tr>
      <th scope="col">City</th>
      <th scope="col">Sales</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Irvine</td>
      <td>42</td>
    </tr>
  </tbody>
</table>
```

Result: a titled two-column table. The headers label the columns.

## 8. Class, id, div, and span

### Concept

`div` and `span` have no meaning. They are hooks for CSS and JavaScript. Prefer a semantic element when one exists. `div` is a block box. `span` is a stretch inside a line. Several classes are separated by spaces. `class="card featured"` means both labels apply. An id is unique and can be a jump target.

### Terms

| Term | Meaning |
| --- | --- |
| `div` | A block with no meaning |
| `span` | An inline stretch with no meaning |
| Class | A reusable label |
| Id | A unique address |
| Semantic element | An element that already means something, such as `p` or `nav` |

### Properties

| Hook | Repeats? | CSS selector | Jump link |
| --- | --- | --- | --- |
| `class="card"` | Yes | `.card` | No |
| `id="listing-42"` | No | `#listing-42` | `href="#listing-42"` |

### Examples

**Case: a card that is only a hook**

```html
<div class="card featured" id="listing-42">
  <h2>Irvine</h2>
  <p>Price: <span class="money">$999,000</span></p>
</div>
```

Result: a heading and a paragraph. The price can be styled through `.money` without styling every paragraph. `#listing-42` can be the target of a link.

Why: `div` groups the card because no single semantic element means "card" until you decide `article` fits. `span` marks the price inside the sentence. A `div` inside the `p` would be invalid.

**Case: `span` used to make a paragraph**

```html
<span class="intro">Active listings.</span>
<span class="intro">Pending listings.</span>
```

Result: both sentences sit in one line, separated by a space, unless CSS changes them.

Why: `span` does not start a new block. Two `p` elements do.

**Case: a div where `nav` already exists**

```html
<div class="nav">
  <a href="/">Home</a>
  <a href="/search">Search</a>
</div>
```

Result: the links work. They are not a navigation landmark.

Why: `nav` means this group of links is navigation. The class `nav` is only a label. Use `<nav>` and add a class if you still need a style hook.

**Case: the same id twice**

```html
<div id="listing-42">One</div>
<div id="listing-42">Two</div>
```

Result: the document is invalid. A link to `#listing-42` scrolls to the first one. `getElementById` returns one element.

Why: an id is an address. Two houses cannot share a street number if you want the URL to pick one.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `div` for every region | The page has no landmarks | Use `nav`, `main`, `article` when they fit |
| `div` inside `span` or inside `p` | Invalid. The browser closes the `p` early | Keep blocks out of inline elements and out of `p` |
| Id reused | Jump links and script hit one element | A class |
| Class used as the only name of a unique landmark | No `#` link | Add an id when something must be addressable |

### Tip

Start with the element that already means the thing. Add a `div` or a `span` only when you need a box and no meaningful element fits.

### Practice

Wrap a heading `Irvine` and a paragraph in one card div with class `card` and id `listing-9`. Mark `$999,000` inside the paragraph with a span class `money`.

### Solution

```html
<div class="card" id="listing-9">
  <h2>Irvine</h2>
  <p>Price: <span class="money">$999,000</span></p>
</div>
```

Result: one addressable card. The price is a span, not a second paragraph.

## 9. Block and inline

### Concept

This is the default layout before your CSS. You will override it. Know the defaults so you know what you changed. A block starts on a new line and stretches across the parent. An inline element sits in the line of text. A block inside an inline element is invalid, with one modern exception: an `a` may wrap a whole card, including blocks.

`img` is a replaced element. Width and height apply even though it sits in a line.

### Terms

| Term | Meaning |
| --- | --- |
| Block | New line, stretches across the parent |
| Inline | Sits in the line. Width and height do not apply the way they do on blocks |
| Replaced | The file paints inside the box. `img` |
| Invalid nesting | A combination the parser will rewrite |

### Properties

| Display | Elements | Behavior |
| --- | --- | --- |
| Block | `p`, `div`, `h1`–`h6`, `ul`, `ol`, `li`, `section`, `table` | Starts on a new line and stretches |
| Inline | `span`, `strong`, `em`, `a`, `code` | Sits in the line |
| Replaced | `img` | Sits in the line, and width and height apply |

### Examples

**Case: two blocks stack**

```html
<p>Active listings.</p>
<p>Pending listings.</p>
```

Result: two lines. Each paragraph starts on its own line and uses the width of the parent.

Why: `p` is a block. You did not need a `<br>`.

**Case: inline elements share a line**

```html
<p><strong>Irvine</strong> <em>3 beds</em></p>
```

Result: `Irvine 3 beds` on one line, with Irvine strong and `3 beds` stressed.

Why: `strong` and `em` are inline. They do not break the paragraph.

**Case: a block inside a span**

```html
<span>
  <div>Irvine</div>
</span>
```

Result: the browser does not keep a `div` inside a `span`. The tree is rewritten.

Why: inline elements do not contain blocks. If you need a box that contains a heading, use `div` or a semantic block.

**Case: a link wrapping a card**

```html
<a href="listing.html">
  <h2>Irvine</h2>
  <p>3 beds</p>
</a>
```

Result: the heading and the paragraph are both one link. This is valid in modern HTML.

Why: `a` is the exception that may wrap blocks. A `span` still may not.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `div` inside `span` | The parser rewrites the tree | Swap them, or drop the span |
| `br` used to stack blocks | Extra empty lines and a fragile layout | Let block elements stack |
| Expecting `width` on a `span` to work with no CSS | Inline width is not a box width | CSS `display`, or use a block |
| Wrapping a card in `span` because the link worked that way | Invalid if the card contains blocks | Wrap with `a` |

### Tip

When the tree looks right in the file and wrong in the inspector, look for a block inside an inline element. The inspector shows the repaired tree.

### Practice

Put the words `Irvine` and `3 beds` in one paragraph, with `Irvine` strong. Do not use `br`.

### Solution

```html
<p><strong>Irvine</strong> 3 beds</p>
```

Result: one line. `Irvine` is important. The bed count is the rest of the paragraph.

## 10. Forms

### Concept

A form collects input and sends it somewhere. Each control needs a `name` if it should be submitted. Each control needs a label. `method="get"` puts the data in the URL. `method="post"` puts it in the body. Browser checks such as `required` are a convenience. The server still has to check the data.

`disabled` is not submitted. `readonly` is submitted and is not editable. `placeholder` is not a label.

### Terms

| Term | Meaning |
| --- | --- |
| Control | One input, textarea, select, or button |
| Successful control | A control that is named, not disabled, and included in the submit |
| `label for` | Must match the control's `id` |
| Wrapping label | The control sits inside the label, so `for` is optional |
| Radio group | Several radios that share a `name` |

### Properties

| Piece | Meaning |
| --- | --- |
| `action` | Where the data goes. Empty means the current URL |
| `method="get"` | Data in the URL. Bookmarkable. Good for a search |
| `method="post"` | Data in the body. Good for creating or changing |
| `name` | The key in the submission. No name, no data |
| `fieldset` / `legend` | Groups related controls and names the group |
| `enctype="multipart/form-data"` | Required when the form sends a file |

| `type` | Collects |
| --- | --- |
| `text` | One line |
| `password` | Hidden characters. Still clear text unless the page is HTTPS |
| `email` / `url` / `tel` | A rough shape, and often a different phone keyboard |
| `number` | A number. `min`, `max`, and `step` constrain it |
| `checkbox` | On or off |
| `radio` | One choice in a group that shares a `name` |
| `file` | A file. Needs `multipart/form-data` |
| `hidden` | A value the user does not edit |
| `submit` | Sends the form |

| | `input` | `textarea` | `select` | `button` |
| --- | --- | --- | --- | --- |
| What the user enters | One value, shaped by `type` | Several lines | One option, or many with `multiple` | An action |
| Where the starting text lives | The `value` attribute | Between the tags | `selected` on an `option` | Between the tags |
| Submitted under `name` | Yes | Yes | Yes, the option's `value` | Only a submit button's `value` |
| Needs a label | Yes | Yes | Yes | The button's text names the action |

| Attribute | Effect |
| --- | --- |
| `required` | The browser blocks submit when this is empty |
| `placeholder` | A hint inside the empty box. Not a label |
| `readonly` | Shown, submitted, not editable |
| `disabled` | Shown, not editable, not submitted |
| `pattern` | A regex the whole value must match |
| `multiple` | On `email`, `file`, or `select`, allow more than one |

### Examples

**Case: get puts the names in the URL**

```html
<form action="/search" method="get">
  <label for="city">City</label>
  <input id="city" name="city" type="text" required>
  <button type="submit">Search</button>
</form>
```

Result: submitting `Irvine` goes to `/search?city=Irvine`. An empty submit is blocked by `required`.

Why: `for="city"` matches `id="city"`. Clicking the word City focuses the box. `name` is the query key. `id` is not sent.

**Case: no `name`**

```html
<form action="/search" method="get">
  <label for="city">City</label>
  <input id="city" type="text" value="Irvine">
  <button type="submit">Search</button>
</form>
```

Result: the URL is `/search` with no `city`. The value was visible and was not submitted.

Why: the successful control needs a `name`. The id only connects the label.

**Case: radios share a name, checkboxes can share one too**

```html
<fieldset>
  <legend>Type</legend>
  <label><input type="radio" name="type" value="condo"> Condo</label>
  <label><input type="radio" name="type" value="sfr" checked> Single family</label>
</fieldset>
```

Result: one of the two can be selected. The page loads with Single family selected. Submitting sends `type=sfr`.

Why: the shared `name` makes them one group. Different `value`s tell them apart. The wrapping label needs no `for`.

**Case: `disabled` drops the value, `readonly` keeps it**

```html
<input name="note" value="keep" readonly>
<input name="secret" value="drop" disabled>
```

Result: a get submit includes `note=keep` and omits `secret`.

Why: disabled controls are not successful controls. Readonly controls are.

**Case: a file input without the right encoding**

```html
<form action="/upload" method="post">
  <input type="file" name="photo">
  <button type="submit">Upload</button>
</form>
```

Result: the file is not sent as a file. The default encoding is URL-style form data, which is the wrong container.

Why: add `enctype="multipart/form-data"` on the form. The server still has to accept that encoding.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `placeholder` used as the only label | The hint disappears when the user types | A real `label` |
| `id` without `name` | The field is not in the submission | Add `name` |
| `disabled` on a value you meant to send | The key is missing | `readonly`, or a hidden input |
| Two submit buttons and no `type` on a `button` | Inside a form, `button` defaults to submit | `type="button"` for a button that must not submit |
| Trusting `required` as the only check | A client can skip it | Check again on the server |

### Tip

Submit the form once and read the URL, or the request body. Every key you see is a `name`. Every key you expected and do not see is a missing `name` or a disabled control.

### Practice

Build a get form that searches `/search` for a required city. The label is `City`. The input id is `city`. Include a submit button labeled `Search`.

### Solution

```html
<form action="/search" method="get">
  <label for="city">City</label>
  <input id="city" name="city" type="text" required>
  <button type="submit">Search</button>
</form>
```

Result: an empty submit is blocked. `Irvine` requests `/search?city=Irvine`.

## 11. Semantic layout

### Concept

These elements mean something. A `div` does not. Screen readers can jump by landmark. `main` is the unique main content. `article` is a piece that would make sense alone. `section` is a region with a heading. If you cannot give the region a heading, it is probably a `div`. Do not wrap every region in `section`.

### Terms

| Term | Meaning |
| --- | --- |
| Landmark | A region assistive tools can jump to. `header`, `nav`, `main`, `aside`, `footer`, `search` |
| `article` | Standalone content, such as a post or a listing |
| `section` | A grouped region that has a heading |
| Skip link | A link to `#main` so keyboard users can skip the nav |

### Properties

| Element | Meaning |
| --- | --- |
| `header` | Introductory content. The page can have one, and an article can have its own |
| `nav` | A block of navigation links. Not every group of links |
| `main` | The unique main content. One per page |
| `article` | A piece that would make sense syndicated alone |
| `section` | A region with a heading |
| `aside` | Related, not the main thread |
| `footer` | Closing matter for the page or for one article |
| `address` | Contact for the nearest article or the page |
| `search` | A search landmark |
| `hgroup` | A heading plus a subheading |

### Examples

**Case: landmarks and a skip link**

```html
<a href="#main">Skip to content</a>
<header>
  <p>Acme Realty</p>
  <nav>
    <a href="/">Home</a>
    <a href="/search">Search</a>
  </nav>
</header>
<main id="main">
  <h1>Irvine, March</h1>
  <p>Sales fell slightly.</p>
</main>
<footer>
  <p>Example brokerage.</p>
</footer>
```

Result: the first Tab stop can be the skip link. Activating it moves to `main`. The nav is a landmark. The footer is a landmark.

Why: the skip link is an ordinary fragment link. `id="main"` is what makes the target exist.

**Case: `section` without a heading**

```html
<section>
  <p>Sales fell slightly.</p>
</section>
```

Result: a region with no name in the outline.

Why: a `section` should have a heading. This paragraph can live in `main` with no extra wrapper, or in a `div` if you only need a style hook.

**Case: `article` versus `section`**

```html
<article>
  <h1>Irvine, March</h1>
  <p>Sales fell slightly.</p>
  <section>
    <h2>Glossary</h2>
    <p>DOM means days on market.</p>
  </section>
</article>
```

Result: one article that could stand alone, with a glossary section inside it.

Why: the report makes sense in a feed by itself, so it is an `article`. The glossary is a headed part of that report, so it is a `section`.

**Case: two `main` elements**

```html
<main><h1>Search</h1></main>
<main><h1>Results</h1></main>
```

Result: the document is invalid. Assistive tools expect one main landmark.

Why: one `main` per page. The results belong inside that `main`, under a heading.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `section` around every `div` | A pile of unnamed or weakly named regions | `section` only with a heading |
| More than one `main` | Invalid landmarks | One `main` |
| `nav` around a footer license link | A landmark for something that is not navigation | Leave it in `footer` |
| Skip link whose target id does not exist | The link goes nowhere | Put the id on `main` |

### Tip

A landmark is a door. If you would not want a screen-reader user to jump there, do not give it a landmark element.

### Practice

Write a page body with a skip link to `#main`, a `header` containing a `nav`, a `main` with id `main` and a heading `Search`, and a `footer`.

### Solution

```html
<a href="#main">Skip to content</a>
<header>
  <nav><a href="/search">Search</a></nav>
</header>
<main id="main">
  <h1>Search</h1>
</main>
<footer><p>Practice page.</p></footer>
```

Result: four regions. The skip link lands on `main`.

## 12. Media and embeds

### Concept

`audio` and `video` can play a file. Without `controls`, the user may have no way to start it. `autoplay` is often blocked unless the media is also `muted`. An `iframe` loads another page inside yours. Give it a `title`. Sandbox it when you do not fully trust that page.

### Terms

| Term | Meaning |
| --- | --- |
| `controls` | The play and volume UI |
| `poster` | The image shown before the video plays |
| `source` | One candidate file. The browser picks the first type it can play |
| `iframe` | A nested browsing context |
| `sandbox` | Removes scripts and forms until you add tokens back |

### Properties

| Attribute | Meaning |
| --- | --- |
| `controls` | Show play and volume |
| `autoplay` | Start immediately. Often blocked unless `muted` |
| `loop` | Play again at the end |
| `muted` | Silent |
| `poster` | Image before playback |
| `preload` | `none`, `metadata`, or `auto`. A hint |
| `sandbox` | Lock the iframe down. Add `allow-scripts` only if you mean to |

### Examples

**Case: audio with a fallback**

```html
<audio controls src="tour.mp3">
  Your browser cannot play this audio.
</audio>
```

Result: a player with controls. If the browser cannot play `tour.mp3`, the sentence inside is shown.

Why: the text is the fallback, not a caption. `controls` is what makes the player usable without script.

**Case: video with two files**

```html
<video controls width="640" poster="cover.jpg">
  <source src="tour.webm" type="video/webm">
  <source src="tour.mp4" type="video/mp4">
</video>
```

Result: the cover image shows until play. The browser uses the first source it understands.

Why: `source` order is the preference order. `width` reserves layout space. `poster` is not `alt`. Video still needs a transcript nearby if the words matter.

**Case: autoplay without mute**

```html
<video autoplay src="tour.mp4"></video>
```

Result: many browsers refuse to start the sound. There are also no controls, so the user cannot start it either.

Why: add `controls`. Add `muted` only when silent autoplay is actually what you want.

**Case: an iframe**

```html
<iframe title="Map of Irvine" src="https://example.com/map" width="600" height="400" sandbox></iframe>
```

Result: a 600 by 400 frame titled for assistive tools. Scripts and form submission inside the frame are blocked.

Why: `sandbox` with no tokens is the strict setting. `title` names the frame. A frame with no title is announced as a bare frame.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Media with no `controls` and no script | The user cannot press play | Add `controls` |
| `autoplay` expected to start with sound | The browser blocks it | `controls`, or `muted` if silence is acceptable |
| Iframe with no `title` | The frame has no accessible name | Add `title` |
| Untrusted page in an unsandboxed iframe | That page runs script in the frame | `sandbox`, then add only the tokens you need |

### Tip

Turn script off and reload. If the audio or video cannot be started, `controls` is missing.

### Practice

Embed `tour.mp4` with controls, a width of 640, and the poster `cover.jpg`.

### Solution

```html
<video controls width="640" poster="cover.jpg" src="tour.mp4"></video>
```

Result: a player, a reserved width, and the cover image before playback.

## 13. The head

### Concept

Nothing in `head` is the article, except the title in the browser tab. Stylesheets and scripts are linked from here. A script in the head without `defer` or `async` blocks parsing. `defer` downloads while the browser parses, keeps the scripts in order, and runs them after the document is parsed. `async` runs as soon as that one file arrives, so order is not guaranteed. For a course page, use `defer`.

### Terms

| Term | Meaning |
| --- | --- |
| `defer` | Run after parsing, in document order |
| `async` | Run as soon as the file arrives |
| Blocking script | No `defer` and no `async`. Parsing waits |
| `type="module"` | A module. Deferred by default |
| `base href` | Rewrites how relative URLs resolve. Easy to misuse |

### Properties

| Tag | Role |
| --- | --- |
| `title` | Required. Unique per page |
| `meta name="description"` | A summary search engines may show |
| `link rel="stylesheet"` | A CSS file |
| `link rel="icon"` | The tab icon |
| `script src` | A JavaScript file |
| `script defer` | After parsing, in order |
| `style` | CSS in the page. A file is easier to cache |

### Examples

**Case: a head that does not block**

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Irvine report</title>
  <meta name="description" content="Twelve-month sales summary for Irvine.">
  <link rel="stylesheet" href="/styles.css">
  <script src="/app.js" defer></script>
</head>
```

Result: the tab is `Irvine report`. CSS is loaded. `app.js` runs after the body has been parsed, so it can find the elements.

Why: `defer` is the ordinary choice for a script that needs the tree.

**Case: a blocking script**

```html
<head>
  <script src="/slow.js"></script>
</head>
<body>
  <h1>Irvine</h1>
</body>
```

Result: the heading is not parsed until `slow.js` has downloaded and run.

Why: a classic script in the head stops the parser. `defer` removes that wait.

**Case: `async` and order**

```html
<script src="/one.js" async></script>
<script src="/two.js" async></script>
```

Result: if `two.js` arrives first, it runs first. That is allowed.

Why: `async` means "as soon as possible", not "in this order". Use `defer` when `two.js` calls something defined in `one.js`.

**Case: `base`**

```html
<head>
  <base href="https://example.com/reports/">
</head>
<body>
  <a href="irvine.html">Irvine</a>
</body>
```

Result: the link goes to `https://example.com/reports/irvine.html`, even if this file is served from somewhere else.

Why: `base` rewrites every relative URL on the page, including images and scripts. Prefer real relative URLs unless you mean that rewrite.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Script in the head with no `defer` | Parsing waits on the download | `defer` |
| `async` on scripts that depend on each other | They run in arrival order | `defer` |
| A shared `title` on every page | Tabs and bookmarks are identical | A title that names this page |
| `base` left over from a demo | Links leave the site you think you are on | Remove it |

### Tip

If the script says it cannot find an element that is clearly in the body, the script ran before the parser reached that element. Add `defer`.

### Practice

Write a head with a UTF-8 charset, a viewport, the title `Search`, a stylesheet `/styles.css`, and a deferred script `/app.js`.

### Solution

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Search</title>
  <link rel="stylesheet" href="/styles.css">
  <script src="/app.js" defer></script>
</head>
```

Result: the tab says `Search`. The script runs after the document is parsed.

## 14. Characters, comments, and paths

### Concept

`<`, `>`, and `&` are markup. When they are text, write them as entities. Comments are not secret. Anyone can view source. Paths are resolved from the current page, unless they start with `/`.

### Terms

| Term | Meaning |
| --- | --- |
| Entity | A typed replacement, such as `&amp;` |
| Comment | `<!-- -->`. Visible in view source |
| Relative path | From the current file's folder |
| Root path | From the site root. Starts with `/` |
| Parent path | `../` goes up one folder |

### Properties

| You want | Write |
| --- | --- |
| `<` | `&lt;` |
| `>` | `&gt;` |
| `&` | `&amp;` |
| `"` inside an attribute | `&quot;` |
| a non-breaking space | `&nbsp;` |
| an em dash | `&mdash;`, or the real character because the page is UTF-8 |

From a page at `/reports/irvine.html`:

| Path | Resolves to |
| --- | --- |
| `photo.jpg` | `/reports/photo.jpg` |
| `./photo.jpg` | The same folder |
| `../photo.jpg` | `/photo.jpg` |
| `/photo.jpg` | The site root |
| `//cdn.example.com/a.jpg` | The same protocol as the page, on that host |

### Examples

**Case: an ampersand in text**

```html
<p>Beds &amp; baths</p>
<p>Beds & baths</p>
```

Result: the first paragraph shows `Beds & baths`. The second is invalid. Browsers often repair it and show the same words, until the `&` starts something that looks like an entity.

Why: `&amp;` is the text ampersand. A bare `&` starts an entity.

**Case: a comment is public**

```html
<!-- Password reset token: 42 -->
<p>Reset your password.</p>
```

Result: the page shows the sentence. View source shows the token.

Why: comments are not access control. Do not put secrets in them.

**Case: relative and root paths**

The file is `/reports/irvine.html`.

```html
<img src="photo.jpg" alt="House">
<img src="/photo.jpg" alt="House">
```

Result: the first requests `/reports/photo.jpg`. The second requests `/photo.jpg`.

Why: a path with no leading slash is relative to the current folder. A leading slash jumps to the host root.

**Case: `../`**

The file is `/reports/irvine.html`.

```html
<a href="../index.html">Home</a>
```

Result: the link requests `/index.html`.

Why: `..` leaves `reports`. One `..` from `/reports/` is the root of the site.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A bare `&` in text | Invalid markup. Sometimes it displays, sometimes it eats characters | `&amp;` |
| A secret in a comment | Anyone can read it | Keep secrets on the server |
| `/photo.jpg` when the file sits next to the page | A request to the host root, often a 404 | `photo.jpg` |
| `../` counted from the URL you wish you had | The file is one folder off | Count from the page's real folder |

### Tip

Copy the page's URL, delete the file name, and paste the relative path onto what remains. That is the request the browser will make.

### Practice

From a page at `/reports/irvine.html`, link `Home` to `/index.html` using a relative path, and show the text `Beds & baths`.

### Solution

```html
<a href="../index.html">Home</a>
<p>Beds &amp; baths</p>
```

Result: the link requests `/index.html`. The paragraph shows an ampersand.

## 15. A full page

### Concept

This section puts the earlier pieces into one document you can open. Tab through the form. The label should focus the field. The skip link should move to `main`. Resize the window. The viewport meta is what lets the phone use its own width.

### Terms

| Term | Meaning |
| --- | --- |
| Practice page | One file you open directly in a browser |
| Tab order | Links and controls, in tree order, plus `tabindex` |
| Outline | The headings: one `h1`, then `h2` for the glossary |

### Properties

The page uses elements from the earlier sections. No new attributes are required beyond `action`, `method`, `for`, `id`, `name`, `required`, and `type`.

### Examples

**Case: the page**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Search homes</title>
  </head>
  <body>
    <a href="#main">Skip to content</a>
    <header>
      <p>Example Realty</p>
      <nav>
        <a href="#search">Search</a>
        <a href="#glossary">Glossary</a>
      </nav>
    </header>
    <main id="main">
      <h1>Search homes</h1>
      <form id="search" action="/search" method="get">
        <label for="city">City</label>
        <input id="city" name="city" required>
        <button type="submit">Search</button>
      </form>
      <section id="glossary">
        <h2>Glossary</h2>
        <dl>
          <dt>DOM</dt>
          <dd>Days on market.</dd>
        </dl>
      </section>
    </main>
    <footer><p>Practice page.</p></footer>
  </body>
</html>
```

Result: the tab is `Search homes`. The skip link jumps to the heading's `main`. Submitting `Irvine` requests `/search?city=Irvine`. The glossary is a headed section.

Why: the form's `id` is `search`, which is also the nav target. The input's `id` is `city`, which matches the label. Those are different ids because they name different elements.

**Case: the label id does not match**

```html
<label for="city">City</label>
<input id="town" name="city" required>
```

Result: clicking `City` does not focus the field. The control still submits as `name="city"`.

Why: `for` matches `id`, not `name`.

**Case: the glossary heading skipped to `h4`**

```html
<h1>Search homes</h1>
<h4>Glossary</h4>
```

Result: the page still shows the word Glossary. The outline jumps from `h1` to `h4`.

Why: the next level after the page title is `h2`.

**Case: color added in the HTML**

```html
<h1 style="color: navy">Search homes</h1>
```

Result: the heading is navy. The structure is harder to read, and every page that wants the same color repeats the attribute.

Why: color belongs in the CSS course. This page should stay structural.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `for` and `id` differ | The label does not focus the control | Make them the same string |
| Two elements with `id="search"` | The nav and the label target one of them | The form and the field need different ids |
| No `name` on the city field | `/search` has no query | Add `name="city"` |
| Glossary marked with a `div` | No section in the outline | `section` and an `h2` |

### Tip

Open the file, then use only the keyboard. Tab, type a city, and press Enter. The address bar is the form's grade.

### Practice

Save the page above as `search.html` and open it. Submit the city `Tustin`. Write down the URL the browser requests.

### Solution

The request is `/search?city=Tustin`.

`action="/search"` is the path. `method="get"` puts the fields in the query string. `name="city"` is the key. The typed value is `Tustin`.

## Drill

1. Write a page with a UTF-8 charset, the title `Irvine`, one `h1`, and a paragraph.
2. Link `Irvine report` to `reports/irvine.html#dom`.
3. Build a required city search form whose get URL will contain `city`.
4. Show the text `Beds & baths`.
5. Nest `By city` under the ordered step `Search`.

### Solution 1

The document in section 1's practice, with the title and heading set to `Irvine`.

### Solution 2

```html
<a href="reports/irvine.html#dom">Irvine report</a>
```

### Solution 3

The form in section 10's solution. `Irvine` requests `/search?city=Irvine`.

### Solution 4

```html
<p>Beds &amp; baths</p>
```

### Solution 5

The list in section 6's solution.
