# HTML

Course 1 of 15. Next: [02-css.md](02-css.md).

HTML describes the structure of a page. The browser builds a tree from your tags and shows that tree. Colors, columns, and animation are CSS. Behavior is JavaScript.

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

Every page starts with the same skeleton. `lang` tells assistive tools which language the text is. `charset` tells the browser how to read the bytes.

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

| Piece | Role |
| --- | --- |
| `<!DOCTYPE html>` | This file is modern HTML, not an old dialect |
| `<html>` | The root of the tree |
| `<head>` | Information about the page. Not shown as the main content |
| `<body>` | What the reader sees |
| `<title>` | The tab name and the default bookmark text |
| `charset="utf-8"` | Letters, accents, and symbols decode correctly |
| `viewport` | Phones use the device width instead of a fake desktop width |

The tree is parents and children. `body` is the parent of `h1` and `p`. `h1` and `p` are siblings. A tag that is opened inside a parent must close before that parent closes.

## 2. Elements and attributes

An element is a start tag, content, and an end tag. A void element has no end tag and no children.

```html
<a href="https://example.com" target="_blank" rel="noopener">Example</a>
<img src="house.jpg" alt="Front of a one-story house">
```

| Kind | Examples | Closes? |
| --- | --- | --- |
| Normal | `p`, `h1`, `a`, `div`, `li` | Yes, with an end tag |
| Void | `img`, `br`, `hr`, `input`, `meta`, `link`, `source` | No |
| Attribute | `href`, `src`, `alt`, `class`, `id` | Lives on the start tag |

| Attribute rule | Why |
| --- | --- |
| Quote the value | `href="homes.html"` survives spaces and special characters |
| Lowercase names | The browser accepts mixed case. Lowercase is the style to keep |
| Boolean attributes | `checked`, `disabled`, `required`, and `multiple` are true when present |
| One `id` per page | An id is a unique address. A class may repeat |

Global attributes that work on almost every element:

| Attribute | Meaning |
| --- | --- |
| `class` | One or more labels for CSS and for JavaScript |
| `id` | A unique label |
| `title` | Extra text, often a tooltip. Do not hide required instructions here |
| `hidden` | The element is not shown and is skipped by the accessibility tree |
| `lang` | Language of this subtree, when it differs from the page |
| `tabindex` | Where Tab stops. `0` adds the element to the natural order. Avoid positive numbers |
| `data-*` | Your own data, read later from JavaScript as `dataset` |

`class`, `id`, and `data-*` are the three labels you will mix up.

| | `class` | `id` | `data-*` |
| --- | --- | --- | --- |
| May repeat on the page | Yes | No | Yes |
| CSS hook | `.city` | `#report` | `[data-city="Irvine"]` |
| JavaScript hook | `querySelectorAll(".city")` | `getElementById("report")` | `element.dataset.city` |
| Shows up in the URL | No | Yes, as `#report` | No |
| Several on one element | `class="card featured"` | One `id` | Several `data-` names |

## 3. Text

Headings describe the outline. Use one `h1`. Do not pick `h3` because it looks smaller. Size is CSS.

```html
<h1>Market report</h1>
<h2>Irvine</h2>
<p>The median close is <strong>$1,200,000</strong>.</p>
<p>Days on market are <em>lower</em> than last year.</p>
<p>Line one<br>Line two</p>
<hr>
<blockquote cite="https://example.com/report">
  <p>Prices rose 4 percent.</p>
</blockquote>
<pre><code>SELECT City FROM california_sold;</code></pre>
```

| Element | Use |
| --- | --- |
| `h1`–`h6` | Outline levels. Do not skip from `h1` to `h4` |
| `p` | A paragraph |
| `br` | A line break inside a paragraph, such as an address. Not a way to make paragraphs |
| `hr` | A thematic break |
| `strong` | Important text. Browsers bold it |
| `b` | Bold with no extra importance |
| `em` | Stress. Browsers italicize it |
| `i` | Italic with no extra stress, such as a taxonomic name |
| `mark` | Highlight, like a search hit |
| `small` | Side comment, legal fine print |
| `sub` / `sup` | Subscript and superscript |
| `code` | A short piece of code in a sentence |
| `pre` | Keep spaces and line breaks |
| `blockquote` | A quotation from somewhere else |
| `q` | A short quotation inside a sentence |
| `abbr` | An abbreviation. Put the expansion in `title` |
| `cite` | The title of a work |
| `del` / `ins` | Deleted and inserted text |
| `wbr` | A place the browser may break a long word |

White space between tags collapses into one space. A blank line in the file does not make a blank line on the page. Use `p`, `br`, or CSS.

## 4. Links

`href` is the destination. A link can wrap text or an image. Without `href`, an `a` is a placeholder, not a link.

```html
<a href="reports/irvine.html">Irvine report</a>
<a href="reports/irvine.html#dom">Days on market</a>
<a href="mailto:hello@example.com">Email</a>
<a href="https://example.com" target="_blank" rel="noopener noreferrer">Outside site</a>
<a href="#top">Back to top</a>
```

| `href` shape | Goes to |
| --- | --- |
| `reports/irvine.html` | A file relative to the current page |
| `/reports/irvine.html` | A file from the site root |
| `../index.html` | Up one folder |
| `#dom` | An element with `id="dom"` on this page |
| `https://example.com` | Another site |
| `mailto:` | The mail app |
| `tel:+15551212` | The phone app |

| Attribute | When |
| --- | --- |
| `target="_blank"` | Open a new tab. Pair it with `rel="noopener noreferrer"` |
| `download` | Suggest a download instead of navigation |
| `rel` | Relationship. `noopener` stops the new page from scripting your page |

The visible text should say where the link goes. "Click here" does not.

## 5. Images

The `alt` text is what a screen reader reads, and what remains if the file fails. Decorative images use `alt=""`.

```html
<img src="photos/house.jpg" alt="White house with a red door" width="800" height="600">
<figure>
  <img src="photos/chart.png" alt="Bar chart of median price by month">
  <figcaption>Median close price, last 12 months.</figcaption>
</figure>
<picture>
  <source srcset="banner-wide.jpg" media="(min-width: 800px)">
  <img src="banner-narrow.jpg" alt="Row of houses at sunset">
</picture>
```

| Attribute | Meaning |
| --- | --- |
| `src` | The file |
| `alt` | Replacement text. Required for information-bearing images |
| `width` and `height` | The layout size. Set both so the page does not jump when the file arrives |
| `srcset` and `sizes` | Alternate files for different screen widths |
| `loading="lazy"` | Wait to fetch images far down the page |
| `decoding="async"` | Decode off the main work of painting |

`figure` and `figcaption` tie a caption to the image. The caption is extra. The `alt` still has to stand alone.

SVG can be an `img` or inline markup. Inline SVG can be styled with CSS. An `img` of an SVG cannot.

## 6. Lists

```html
<ul>
  <li>Active</li>
  <li>Pending</li>
</ul>
<ol start="3" reversed>
  <li>Third step</li>
  <li>Second step</li>
</ol>
<dl>
  <dt>DOM</dt>
  <dd>Days on market.</dd>
  <dt>HOA</dt>
  <dd>Monthly association fee.</dd>
</dl>
```

| List | Use |
| --- | --- |
| `ul` | Items with no order |
| `ol` | Items with an order. `start`, `reversed`, and `type` change the counter |
| `li` | One item. Only valid inside `ul` or `ol` |
| `dl` | Names and their descriptions |
| `dt` | The name |
| `dd` | The description. One `dt` may have several `dd` elements |

Nest a list by putting the inner `ul` or `ol` inside an `li`, not between two `li` elements.

## 7. Tables

Use a table for data in rows and columns. Use CSS grid for page layout.

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
    <tr>
      <th scope="row">February</th>
      <td>38</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <th scope="row">Total</th>
      <td>80</td>
    </tr>
  </tfoot>
</table>
```

| Element | Role |
| --- | --- |
| `table` | The grid |
| `caption` | The title. First child of `table` |
| `thead` | Header rows |
| `tbody` | Data rows |
| `tfoot` | Summary rows |
| `tr` | One row |
| `th` | A header cell |
| `td` | A data cell |

| Attribute | Meaning |
| --- | --- |
| `scope="col"` | This header labels the column |
| `scope="row"` | This header labels the row |
| `colspan="2"` | This cell covers two columns |
| `rowspan="2"` | This cell covers two rows |
| `headers` | On a complex cell, the ids of the headers that label it |

Cells in a row should add up to the same number of columns once you count `colspan`.

## 8. Class, id, div, and span

`div` and `span` have no meaning. They are hooks for CSS and JavaScript. Prefer a semantic element when one exists.

```html
<div class="card featured" id="listing-42">
  <h2>Irvine</h2>
  <p>Price: <span class="money">$999,000</span></p>
</div>
```

| Hook | Repeats? | CSS selector | Jump link |
| --- | --- | --- | --- |
| `class="card"` | Yes, many cards | `.card` | No |
| `id="listing-42"` | No | `#listing-42` | `href="#listing-42"` |

Several classes are separated by spaces. `class="card featured"` means both labels apply.

## 9. Block and inline

This is the default layout before your CSS. You will override it constantly. Know the defaults so you know what you changed.

| Display | Elements | Behavior |
| --- | --- | --- |
| Block | `p`, `div`, `h1`–`h6`, `ul`, `ol`, `li`, `section`, `table` | Starts on a new line and stretches across the parent |
| Inline | `a`, `span`, `strong`, `em`, `img`, `code` | Sits in the line of text |
| Inline-block | Replaced elements vary | In the line, but width and height apply |

A block element inside an inline element is invalid. Do not put a `div` inside a `span`. A link may wrap a whole card, including block elements, in modern HTML.

`img` is a replaced element. The file paints inside the box. Width and height apply even though it sits in a line.

## 10. Forms

A form collects input and sends it somewhere. Each control needs a `name` if it should be submitted. Each control needs a label.

```html
<form action="/search" method="get">
  <label for="city">City</label>
  <input id="city" name="city" type="text" required autocomplete="address-level2">

  <label for="max">Max price</label>
  <input id="max" name="max" type="number" min="0" step="1000">

  <fieldset>
    <legend>Type</legend>
    <label><input type="radio" name="type" value="condo"> Condo</label>
    <label><input type="radio" name="type" value="sfr"> Single family</label>
  </fieldset>

  <label for="notes">Notes</label>
  <textarea id="notes" name="notes" rows="4"></textarea>

  <label for="sort">Sort</label>
  <select id="sort" name="sort">
    <option value="price">Price</option>
    <option value="newest">Newest</option>
  </select>

  <button type="submit">Search</button>
</form>
```

| Piece | Meaning |
| --- | --- |
| `action` | Where the data goes. Empty means the current URL |
| `method="get"` | Data in the URL. Good for searches. Bookmarkable |
| `method="post"` | Data in the body. Good for creating or changing something |
| `label for` | Must match the control's `id` |
| Wrapping label | The control inside the label does not need `for` |
| `name` | The key in the submission. No name, no data |
| `fieldset` / `legend` | Groups related controls and names the group |

| `type` | Collects |
| --- | --- |
| `text` | One line of text |
| `password` | Hidden characters. Still sent in clear text unless the page is HTTPS |
| `email` | A rough email shape |
| `number` | A number. `min`, `max`, and `step` constrain it |
| `tel` | A phone number. The check is weak. The mobile keyboard changes |
| `url` | A URL |
| `search` | Search text. Some browsers add a clear button |
| `date`, `time`, `datetime-local` | A date or time picker |
| `checkbox` | On or off. Share a `name` and use different `value`s for a set |
| `radio` | One choice in a group. The group shares a `name` |
| `file` | A file. The form needs `enctype="multipart/form-data"` |
| `hidden` | A value the user does not edit |
| `range` | A slider |
| `color` | A color |
| `submit` | Sends the form. A `button` with `type="submit"` does the same |
| `reset` | Restores defaults. Easy to hit by mistake |
| `button` | Does nothing until JavaScript listens |

The controls that are not a single `input`:

| | `input` | `textarea` | `select` | `button` |
| --- | --- | --- | --- | --- |
| What the user enters | One value, shaped by `type` | Several lines of text | One option, or many with `multiple` | An action |
| Where the starting text lives | The `value` attribute | Between the tags | `selected` on an `option` | Between the tags |
| Submitted under `name` | Yes | Yes | Yes. The option's `value` | Only a submit button, and only its `value` |
| Free text plus suggestions | `list` pointing at a `datalist` | No | No. The choices are the options | No |
| Needs a label | Yes | Yes | Yes | The button's own text is the name of the action |

| Attribute | Effect |
| --- | --- |
| `required` | The browser blocks submit when this is empty |
| `placeholder` | Hint inside the empty box. Not a label |
| `value` | The starting value |
| `readonly` | Shown, submitted, not editable |
| `disabled` | Shown, not editable, not submitted |
| `minlength` / `maxlength` | Length limits |
| `pattern` | A regex the value must match |
| `autocomplete` | Hints the browser's autofill. Use real tokens such as `email` |
| `multiple` | On `email`, `file`, or `select`, allow more than one |
| `selected` | The starting `option` |
| `checked` | The starting checkbox or radio |

`select` holds `option` elements. `optgroup` labels a cluster of options. `datalist` offers suggestions for a text input while still allowing a free-typed value.

```html
<label for="city2">City</label>
<input id="city2" name="city2" list="cities">
<datalist id="cities">
  <option value="Irvine">
  <option value="San Diego">
</datalist>
```

Browser checks are a convenience. The server still has to check the data. A reader can bypass the page.

## 11. Semantic layout

These elements mean something. A `div` does not. Screen readers can jump by landmark.

```html
<body>
  <a href="#main">Skip to content</a>
  <header>
    <p>Acme Realty</p>
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/search">Search</a></li>
      </ul>
    </nav>
  </header>
  <main id="main">
    <article>
      <h1>Irvine, March</h1>
      <p>Sales fell slightly.</p>
    </article>
    <section>
      <h2>Glossary</h2>
      <p>DOM means days on market.</p>
    </section>
  </main>
  <aside>
    <h2>Saved searches</h2>
  </aside>
  <footer>
    <p>Example brokerage.</p>
  </footer>
</body>
```

| Element | Meaning |
| --- | --- |
| `header` | Introductory content. A page can have one for the site and one inside an article |
| `nav` | A block of links for navigation. Not every group of links |
| `main` | The unique main content. One per page, not inside `article` as a repeat |
| `article` | A piece that would make sense alone, such as a post or a listing |
| `section` | A grouped region with a heading |
| `aside` | Related content, not the main thread |
| `footer` | Closing matter for the page or for one article |
| `address` | Contact information for the nearest `article` or the page |
| `search` | A search form landmark |
| `hgroup` | A heading plus a subheading |

Do not wrap every `div` replacement in `section`. If you cannot give the region a heading, it is probably a `div`.

## 12. Media and embeds

```html
<audio controls src="tour.mp3">
  Your browser cannot play this audio.
</audio>
<video controls width="640" poster="cover.jpg">
  <source src="tour.webm" type="video/webm">
  <source src="tour.mp4" type="video/mp4">
</video>
<iframe title="Map of Irvine" src="https://example.com/map" width="600" height="400"></iframe>
```

| Attribute | Meaning |
| --- | --- |
| `controls` | Show play and volume. Without it the user may have no way to start the media |
| `autoplay` | Starts immediately. Often blocked unless `muted` is also set |
| `loop` | Play again at the end |
| `muted` | Silent |
| `poster` | Image shown before the video plays |
| `preload` | `none`, `metadata`, or `auto`. A hint, not a guarantee |

An `iframe` loads another page inside yours. Always give it a `title`. Sandbox it when the other page is not fully trusted: `sandbox` removes scripts and forms until you add tokens such as `allow-scripts`.

## 13. The head

Nothing in `head` is the article, except the title in the browser chrome.

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Irvine report</title>
  <meta name="description" content="Twelve-month sales summary for Irvine.">
  <link rel="icon" href="/favicon.ico">
  <link rel="stylesheet" href="/styles.css">
  <script src="/app.js" defer></script>
</head>
```

| Tag | Role |
| --- | --- |
| `title` | Required. Unique per page |
| `meta name="description"` | A summary. Search engines may show it |
| `link rel="stylesheet"` | A CSS file |
| `link rel="icon"` | The tab icon |
| `script src` | A JavaScript file |
| `script defer` | Run after the document is parsed, in order |
| `script type="module"` | A module. Deferred by default |
| `base href` | Changes how relative URLs resolve. Easy to misuse. Prefer real relative URLs |
| `style` | CSS in the page. Fine for a tiny demo. A file is easier to cache |

`script` in the head without `defer` or `async` blocks parsing. `async` downloads in parallel and runs as soon as it arrives, order not guaranteed. `defer` keeps order and waits for the parse. For a course page, use `defer`.

## 14. Characters, comments, and paths

Some characters are markup. Write them as entities when they are text.

| You want | Write |
| --- | --- |
| `<` | `&lt;` |
| `>` | `&gt;` |
| `&` | `&amp;` |
| `"` inside an attribute | `&quot;` |
| a non-breaking space | `&nbsp;` |
| — | `&mdash;` or the real character, because the page is UTF-8 |

```html
<!-- The next section is the glossary. Comments are not secret. -->
<p>Beds &amp; baths</p>
```

Anyone can view source. Do not put passwords or private notes in comments.

| Path | From a page at `/reports/irvine.html` |
| --- | --- |
| `photo.jpg` | `/reports/photo.jpg` |
| `./photo.jpg` | Same folder |
| `../photo.jpg` | `/photo.jpg` |
| `/photo.jpg` | The site root, regardless of the current folder |
| `//cdn.example.com/a.jpg` | Same protocol as the page, that host |

## 15. A full page

Type this with autocomplete off. Then open it in a browser. Tab through the form. Resize the window.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Search homes</title>
  </head>
  <body>
    <header>
      <p>Example Realty</p>
      <nav>
        <a href="#search">Search</a>
        <a href="#glossary">Glossary</a>
      </nav>
    </header>
    <main>
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

Check yourself:

| Question | Answer |
| --- | --- |
| Why is `for="city"` on the label? | It ties the label to the input with that id |
| Why is the glossary a `section`? | It has a heading and is part of the outline |
| Why is the nav not a list here? | A list is better when there are many links. Two links may be plain anchors. A `ul` is still a good habit |
| Where would color go? | A CSS file, in the next course |
