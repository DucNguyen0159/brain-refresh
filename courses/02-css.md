# CSS

Course 2 of 15. Previous: [01-html.md](01-html.md). Next: [03-javascript.md](03-javascript.md).

CSS describes how the HTML tree looks. A rule is a selector and a list of declarations. The cascade decides which declaration wins when several match the same element.

Every section has the same parts: the concept, the terms, the properties, several cases with what the page does, common mistakes, a tip, and a practice with a solution. Attach the samples to a small HTML file and open it. The result lines describe that page.

## Contents

1. [Rules and where they live](#1-rules-and-where-they-live)
2. [Selectors](#2-selectors)
3. [The cascade, inheritance, and specificity](#3-the-cascade-inheritance-and-specificity)
4. [Colors and units](#4-colors-and-units)
5. [The box model](#5-the-box-model)
6. [Display](#6-display)
7. [Position and stacking](#7-position-and-stacking)
8. [Flexbox](#8-flexbox)
9. [Grid](#9-grid)
10. [Typography](#10-typography)
11. [Backgrounds, borders, and effects](#11-backgrounds-borders-and-effects)
12. [Pseudo-classes and pseudo-elements](#12-pseudo-classes-and-pseudo-elements)
13. [Transitions and transforms](#13-transitions-and-transforms)
14. [Variables and media queries](#14-variables-and-media-queries)
15. [A layout you can rebuild](#15-a-layout-you-can-rebuild)

## 1. Rules and where they live

### Concept

A rule selects elements and sets properties on them. A declaration is one property, one value, and a semicolon. The browser ignores a declaration it does not understand and keeps the rest of the rule. A missing semicolon can swallow the next declaration because the two lines are read as one broken value.

### Terms

| Piece | In `p { color: navy; }` |
| --- | --- |
| Selector | `p` |
| Declaration | `color: navy;` |
| Property | `color` |
| Value | `navy` |
| Comment | `/* note */`. Not `//` |

### Properties

| Where | How | When |
| --- | --- | --- |
| External file | `<link rel="stylesheet" href="styles.css">` | Normal pages. Cached. One place to edit |
| `style` in the head | `<style> p { color: navy; } </style>` | A single demo file |
| Inline | `<p style="color: navy">` | One element. Hard to maintain |

### Examples

**Case: a rule in a linked file**

```html
<link rel="stylesheet" href="styles.css">
<p>Irvine</p>
```

```css
p { color: navy; }
```

Result: the paragraph is navy. A second page that links the same file gets the same color.

Why: the HTML names the element. The CSS file names the look. The link belongs in `head`.

**Case: a missing semicolon**

```css
p {
  color: navy
  margin: 1rem;
}
```

Result: the paragraph does not turn navy, and it does not gain the margin. Both declarations are dropped as invalid.

Why: without the semicolon, the value of `color` is read as `navy margin: 1rem`, which is not a color. The next rule in the file is still fine.

**Case: an unknown property**

```css
p {
  colour: navy;
  margin: 1rem;
}
```

Result: the paragraph is not navy. It does have a 1rem margin.

Why: `colour` is not a CSS property. The browser skips that declaration only.

**Case: a CSS comment versus a JavaScript comment**

```css
p {
  /* color: red; */
  color: navy;
}
```

Result: the paragraph is navy. The red line does not apply.

Why: CSS comments are `/* */` only. `//` is not a comment. It makes the declaration invalid.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A missing semicolon | That declaration and often the next one are ignored | End every declaration with `;` |
| `//` comments | The line is invalid | `/* */` |
| A misspelled property | Only that line is ignored | Check the name. The rest of the rule can still work |
| Inline styles on every element | One change means editing every tag | A class in the stylesheet |

### Tip

When a rule seems to do nothing, look at the declaration above it before you look at specificity. A missing semicolon hides the next line.

### Practice

Write a rule that sets every paragraph's text color to `#1a1a1a` and its line-height to `1.5`. Put it in `styles.css` and link that file.

### Solution

```html
<link rel="stylesheet" href="styles.css">
```

```css
p {
  color: #1a1a1a;
  line-height: 1.5;
}
```

Result: paragraphs are dark gray with a line height one and a half times the font size. Headings are unchanged.

## 2. Selectors

### Concept

A selector is a question the browser asks of every element. The simple questions are the tag, the class, and the id. Combinators ask about family: a descendant anywhere inside, a child directly inside, the next sibling, or any later sibling. Attribute selectors ask about the start tag. A comma is "or". The universal selector `*` matches everything.

The HTML for every case in this section is this tree:

```html
<article class="card" id="hero">
  <h2>Irvine</h2>
  <p>Active listings.</p>
  <p class="note">3 beds</p>
</article>
<p>Outside the card.</p>
<a href="https://example.com/report.pdf">Report</a>
<a href="/local.html">Local</a>
```

### Terms

| Term | Meaning |
| --- | --- |
| Type selector | The tag name. `h2` |
| Class selector | `.note`. Matches `class="note"` and also `class="card note"` |
| Id selector | `#hero`. One element |
| Descendant | `article p`. A `p` anywhere inside an `article` |
| Child | `.card > h2`. An `h2` whose parent is `.card` |
| Adjacent sibling | `h2 + p`. The first `p` immediately after an `h2` |
| General sibling | `h2 ~ p`. Every `p` after an `h2`, with the same parent |
| Attribute selector | Looks at an attribute's presence or value |
| Group | `h1, h2`. The same declarations for either |

### Properties

| Selector | Matches in the tree above |
| --- | --- |
| `h2` | The heading `Irvine` |
| `.card` | The `article` |
| `#hero` | The same `article` |
| `*` | Every element |
| `article p` | Both paragraphs inside the article. Not "Outside the card." |
| `.card > h2` | The heading, because its parent is the card |
| `.card > p` | Both paragraphs inside the card. They are direct children |
| `h2 + p` | `Active listings.` only |
| `h2 ~ p` | Both paragraphs inside the card |
| `a[href]` | Both links |
| `a[href^="https"]` | The Report link |
| `a[href$=".pdf"]` | The Report link |
| `a[href*="report"]` | The Report link |
| `h1, h2` | The `h2`. There is no `h1` |

### Examples

**Case: class, id, and type**

```css
h2 { color: navy; }
.note { font-style: italic; }
#hero { border: 1px solid #ccc; }
```

Result: `Irvine` is navy. `3 beds` is italic. The whole article has a gray border. `Outside the card.` is unchanged.

Why: `h2` matches the tag. `.note` matches the class token, not the whole class attribute string. `#hero` matches the id. One element can match all three kinds at once. The article matches `#hero` and `.card` together.

**Case: descendant versus child**

```css
article p { background: #eef; }
.card > h2 { color: navy; }
```

Result: both paragraphs inside the article get a pale background, including ones wrapped in a further `div` if you add one. The heading is navy only because it is a direct child of `.card`.

Add this inside the article, after the heading:

```html
<div><p>Wrapped.</p></div>
```

`article p` still matches `Wrapped.` `article > p` does not, because the parent of that `p` is the `div`.

Why: a space means "somewhere inside". `>` means "the parent is exactly this".

**Case: the next sibling versus every later sibling**

```css
h2 + p { font-weight: 700; }
h2 ~ p { color: #444; }
```

Result: `Active listings.` is bold, because it is the element immediately after the `h2`. Both `Active listings.` and `3 beds` are gray, because both are `p` elements after the `h2` with the same parent. `Outside the card.` is neither. It does not share the heading's parent.

Why: `+` is one neighbor. `~` is the rest of the later siblings of that type. Order matters. A `p` before the `h2` would match neither.

**Case: attribute prefixes**

```css
a[href^="https"] { color: green; }
a[href$=".pdf"] { font-weight: 700; }
a[href*="local"] { text-decoration: none; }
```

Result: `Report` is green because the value starts with `https`, and bold because it ends with `.pdf`. `Local` is not green and not bold. `Local` has no underline because `/local.html` contains `local`.

Why: `^=` is the start of the value. `$=` is the end. `*=` is a substring anywhere inside the value. `=` with no symbol is the whole value.

**Case: a group shares declarations, and `*` is everything**

```css
h2, .note { letter-spacing: 0.02em; }
```

Result: the heading and `3 beds` get the extra letter spacing. `Active listings.` does not. It is a `p` without the class `note`.

Why: the comma is or. It is not a descendant combinator. `h2 .note` with a space would mean a `.note` inside an `h2`, which this tree does not have.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A space in `.card > p` written as `.card>p` is fine, but a space in `.card p` changes the meaning | You match descendants you did not mean | Use `>` when you mean the parent |
| `h2 + p` expected to match every paragraph | Only the next sibling matches | `h2 ~ p` |
| `.card.note` versus `.card .note` | The first is one element with both classes. The second is a `.note` inside a `.card` | Look for the space |
| `a[href^=https]` without quotes | The selector can be invalid | Quote the value: `[href^="https"]` |
| Styling `*` with margins | Every element, including `html` and `head`'s display, gets that margin | Style `body` and the elements you mean |

### Tip

Read a selector from right to left. `article p` means "a `p`, then check that an `article` is an ancestor." The rightmost piece is the element that receives the declarations.

### Practice

Using the tree at the top of this section, write one rule that italicizes only `3 beds`, and one rule that boldfaces only `Active listings.`

### Solution

```css
.note { font-style: italic; }
h2 + p { font-weight: 700; }
```

Result: `3 beds` is italic. `Active listings.` is bold. `Outside the card.` is neither.

## 3. The cascade, inheritance, and specificity

### Concept

When two declarations set the same property on the same element, one wins. The order of the decision is importance, then origin, then specificity, then source order. Specificity is a triple: ids, then classes and attributes and pseudo-classes, then elements and pseudo-elements. Twelve elements do not beat one class. An id beats a pile of classes.

Some properties inherit. `color` does. `margin`, `padding`, `border`, `background`, `width`, and `display` do not. A child does not grow a border because its parent has one.

### Terms

| Term | Meaning |
| --- | --- |
| Cascade | The contest between declarations |
| Specificity | How pointed the selector is |
| Source order | Later in the file wins a tie |
| `!important` | Beats a normal declaration. Avoid it |
| `inherit` | The parent's computed value |
| `initial` | The property's default |
| `unset` | `inherit` if the property inherits, otherwise `initial` |
| `revert` | The user-agent style |

### Properties

| Selector | Specificity | Why |
| --- | --- | --- |
| `p` | 0, 0, 1 | One element |
| `.card` | 0, 1, 0 | One class |
| `#hero` | 1, 0, 0 | One id |
| `.card p` | 0, 1, 1 | One class and one element |
| `.card.featured` | 0, 2, 0 | Two classes on the same element |
| Inline `style` | Beats selectors | Heavier than an id |

| Step | Rule |
| --- | --- |
| 1 | `!important` beats a normal declaration |
| 2 | Author styles beat browser defaults |
| 3 | Higher specificity wins |
| 4 | If specificity ties, the later rule wins |

### Examples

**Case: a class beats a pile of elements**

```html
<article class="card"><p>Irvine</p></article>
```

```css
article p { color: gray; }
.card p { color: navy; }
```

Result: `Irvine` is navy.

Why: `.card p` is one class plus one element. `article p` is two elements. `0, 1, 1` beats `0, 0, 2`.

**Case: the same specificity, later rule wins**

```css
.card { color: navy; }
.card { color: green; }
```

Result: the card's text is green. Inherited color makes the paragraphs green too, unless they set their own.

Why: both selectors are `0, 1, 0`. Source order breaks the tie.

**Case: an id beats a class, and that is a trap**

```html
<p class="note" id="intro">3 beds</p>
```

```css
#intro { color: red; }
.note { color: navy; }
```

Result: the text is red, even though `.note` comes later.

Why: `1, 0, 0` beats `0, 1, 0`. Adding another class will not overturn the id. Change the HTML so the id is not doing the styling job.

**Case: inheritance stops at the border**

```css
body { color: #222; }
.card { border: 1px solid #ccc; }
```

```html
<body>
  <div class="card"><p>Irvine</p></div>
</body>
```

Result: the paragraph is `#222` even though it never mentioned `color`. The paragraph has no border. The card does.

Why: `color` inherits. `border` does not. `p { color: inherit; }` is the same as the default for `color`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Adding elements to beat a class | The class still wins | Raise the selector on purpose, or remove the competing rule |
| `!important` to win once | The next override needs another `!important` | Fix the specificity |
| Expecting a child to inherit `margin` or `background` | The child keeps its own default | Set the property on the child |
| Two classes and you only look at order | An id or an inline style is still winning | Count ids, then classes, then elements |

### Tip

Write the triple next to the two selectors before you change either one. Most "it ignores my CSS" bugs are `0, 1, 0` versus `0, 0, 1`, or an id you forgot.

### Practice

Make the paragraph inside `.card` navy, and a later element selector `p` must not turn it gray.

### Solution

```css
p { color: gray; }
.card p { color: navy; }
```

Result: paragraphs outside the card are gray. The one inside `.card` is navy, because the class wins and source order is not required for that win.

## 4. Colors and units

### Concept

A color is a value. Hex, `rgb`, and `hsl` are the same idea written differently. The last pair of an eight-digit hex, or the slash in `rgb(255 204 0 / 0.5)`, is alpha. `currentcolor` means "whatever `color` computed to".

`px` is a CSS pixel. `rem` is the root font size. `em` is this element's font size, and it compounds when nested. `%` is relative to the parent for that property. A unitless `0` does not need a unit. A unitless `line-height` is a multiplier, which is what you want.

`box-sizing: border-box` makes `width` include padding and border. The content-box default makes `width: 100%` overflow as soon as you add padding.

### Terms

| Term | Meaning |
| --- | --- |
| `rem` | Root `em`. Usually 16px until the user changes it |
| `em` | This element's font size |
| Alpha | How opaque the color is |
| `border-box` | The declared width includes padding and border |
| `content-box` | The declared width is the content only. The default |

### Properties

| Form | Example |
| --- | --- |
| Named | `navy`, `transparent`, `currentcolor` |
| Hex | `#fc0`, `#ffcc00`, `#ffcc0080` |
| `rgb` | `rgb(255 204 0 / 0.5)` |
| `hsl` | `hsl(48 100% 50%)` |

| Unit | Means |
| --- | --- |
| `px` | A CSS pixel. Good for hairline borders |
| `rem` | Root font size. Good for type and spacing |
| `em` | This element's font size. Compounds |
| `%` | The parent, for that property |
| `vw` / `vh` | The viewport |
| `ch` | About the width of `0` |
| `fr` | A fraction of free space in a grid |

### Examples

**Case: the same color written three ways**

```css
.a { color: #1d4e89; }
.b { color: rgb(29, 78, 137); }
.c { color: hsl(210, 65%, 33%); }
```

Result: the three elements are the same blue.

Why: hex, rgb, and hsl are notations. The computed color is what the cascade compares, not the spelling.

**Case: alpha**

```css
.panel { background: rgb(29 78 137 / 0.5); }
```

Result: a blue panel you can partly see through. Text on it is not faded unless you also set the text color's alpha.

Why: the alpha belongs to that color. `opacity: 0.5` on the element would fade the text and the children too.

**Case: `em` compounds, `rem` does not**

```html
<ul>
  <li>One
    <ul><li>Two</li></ul>
  </li>
</ul>
```

```css
li { font-size: 1.2em; }
```

Result: "Two" is larger than "One". Each `li` multiplies by 1.2 against its parent.

```css
li { font-size: 1.2rem; }
```

Result: "One" and "Two" are the same size, 1.2 times the root font size.

Why: `em` is local. `rem` always looks at `html`.

**Case: `border-box` keeps the declared width**

```css
.card {
  box-sizing: border-box;
  width: 20rem;
  padding: 1rem;
  border: 1px solid #ccc;
}
```

Result: the card's outer width is 20rem. The content area is 20rem minus 2rem of horizontal padding minus 2px of border.

Without `border-box`, the outer width is 20rem plus the padding plus the border, and a `width: 100%` card overflows its parent.

Why: the default `content-box` adds padding outside the width. Set `box-sizing: border-box` on `*, *::before, *::after` at the top of the file.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `font-size: 1.2em` on nested lists | Each level gets bigger | `rem` |
| `width: 100%` plus padding under `content-box` | Horizontal overflow | `border-box` |
| `line-height: 24px` | The line box ignores a user font-size change | A unitless `1.5` |
| `opacity` to fade a background | The text fades too | An alpha color on `background` |

### Tip

Use `rem` for spacing and type, `px` for a 1px border, and a unitless number for `line-height`.

### Practice

Make `.card` 20rem wide, with 1rem of padding and a 1px solid `#ccc` border, and keep the outer width at 20rem.

### Solution

```css
.card {
  box-sizing: border-box;
  width: 20rem;
  padding: 1rem;
  border: 1px solid #ccc;
}
```

Result: the border edges are 20rem apart. The content is narrower than 20rem by the padding and the border.

## 5. The box model

### Concept

Every element is four nested boxes: content, then padding, then border, then margin. Padding is inside the border and is painted with the background. Margin is outside the border and is transparent. Vertical margins of adjacent blocks collapse into the larger one. Horizontal margins do not collapse. Padding never collapses. `outline` is drawn outside the border and does not take up space. Do not remove a focus outline unless you replace it.

### Terms

| Term | Meaning |
| --- | --- |
| Content | The text or the replaced image |
| Padding | Space inside the border |
| Border | The line. It takes up space |
| Margin | Space outside the border |
| Collapse | Two vertical margins become one, the larger |
| Clockwise | Top, right, bottom, left |

### Properties

| Shorthand | Meaning |
| --- | --- |
| One value | All four sides |
| Two values | Vertical, then horizontal |
| Three values | Top, horizontal, bottom |
| Four values | Top, right, bottom, left |
| `margin-block` / `padding-inline` | Logical sides. They follow the writing direction |

| Fact | Detail |
| --- | --- |
| Negative margin | Pulls neighbors closer. Allowed |
| `min-height` / `max-width` | Limits. `width: 100%; max-width: 40rem` is a reading column |
| `overflow: auto` | A scrollbar only when needed |
| `overflow: hidden` | Clips. Also creates a scroll container and can trap focus styles |
| `outline` | Does not affect layout |

### Examples

**Case: the four sides of a shorthand**

```css
.card {
  margin: 1rem 2rem 3rem 4rem;
  padding: 1rem 2rem;
}
```

Result: margin is 1rem top, 2rem right, 3rem bottom, 4rem left. Padding is 1rem top and bottom, 2rem left and right.

Why: four values go clockwise from the top. Two values are vertical, then horizontal.

**Case: vertical margins collapse**

```css
h2 { margin-bottom: 2rem; }
p { margin-top: 1rem; }
```

```html
<h2>Irvine</h2>
<p>Active listings.</p>
```

Result: the gap between the heading and the paragraph is 2rem, not 3rem.

Why: adjoining vertical margins combine into the larger one. If you need the sum, use padding on one of them, or flex/grid `gap`, which does not collapse.

**Case: horizontal margins do not collapse**

```css
span { margin-right: 1rem; margin-left: 1rem; }
```

```html
<span>A</span><span>B</span>
```

Result: the gap between A and B is 2rem.

Why: collapse is a vertical, block-formatting behavior. Side margins add.

**Case: outline versus border**

```css
a:focus { outline: 2px solid navy; }
a.box:focus { outline: none; border: 2px solid navy; }
```

Result: the outline ring does not push the link's neighbors. The border version grows the link by 2px on each side and the line of text can shift.

Why: outline is painted outside the layout. If you remove it, replace it with something the keyboard user can see. `:focus-visible` is the usual selector, in section 12.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Adding top and bottom margins and expecting the sum | You get the larger one | `gap`, or padding |
| `outline: none` on `:focus` | Keyboard users lose the cursor | `:focus-visible` with a visible style |
| Four values remembered as left-first | The left and right swap roles | Clockwise from the top |
| `overflow: hidden` as a clearfix habit | It clips shadows and sticky descendants | Use it only when you mean to clip |

### Tip

Margin collapses. Padding does not. When a gap is mysteriously too small, you are looking at two vertical margins.

### Practice

Give `.card` a bottom margin of `1rem`, padding of `1rem` on all sides, and a `1px solid #ccc` border.

### Solution

```css
.card {
  margin: 0 0 1rem;
  padding: 1rem;
  border: 1px solid #ccc;
}
```

Result: no extra space above the card from this rule, 1rem under it, and 1rem of padding inside a gray border. Two stacked cards have a 1rem gap, because the collapsed margins are both 1rem and 0.

## 6. Display

### Concept

`display` sets the formatting context. `block` starts a new line and fills the width. `inline` sits in the line and ignores width and height. `inline-block` sits in the line and honors width, height, and vertical margins. `none` removes the element from layout and from the accessibility tree. `visibility: hidden` keeps the gap and hides the paint. `opacity: 0` keeps the element clickable.

`flex` and `grid` are formatting contexts for the children. `gap` is the space between those children. It is not margin, so the first and last children do not grow an outer gap. `display: contents` removes the element's own box and lets its children participate in the parent's layout.

### Terms

| Term | Meaning |
| --- | --- |
| Formatting context | The rules the children are laid out by |
| `none` | Gone from layout and from assistive tech |
| `hidden` | The box remains. The paint does not |
| `gap` | Space between flex or grid items only |

### Properties

| Value | The element |
| --- | --- |
| `block` | New line, fills the width |
| `inline` | In the line. Width and height are ignored |
| `inline-block` | In the line. Width, height, and vertical margins work |
| `none` | Removed from the layout |
| `flex` / `inline-flex` | A flex container. `inline-flex` sits in a line |
| `grid` | A grid container |
| `contents` | The box disappears. The children stay |

| | In flow | Takes a gap | Typical use |
| --- | --- | --- | --- |
| `display: block` | Yes | Yes, the full line | A paragraph |
| `display: inline` | Yes, in the line | The line box | A link in a sentence |
| `display: none` | No | No | Hide it completely |
| `visibility: hidden` | Yes | Yes | Keep the gap |
| `opacity: 0` | Yes | Yes | Invisible and still hit-testable |

### Examples

**Case: `inline` ignores width**

```css
span { width: 10rem; background: #eee; }
```

Result: the span is only as wide as its text. The 10rem does nothing.

Why: width does not apply to a non-replaced inline box. `display: inline-block` makes the 10rem real.

**Case: `none` versus `visibility`**

```css
.gone { display: none; }
.blank { visibility: hidden; }
```

Result: `.gone` leaves no hole, and a screen reader skips it. `.blank` leaves a hole the size of the element and is also skipped by many assistive tools, but the layout gap remains.

Why: hiding with color, or with `opacity: 0`, still leaves a clickable box. `display: none` is the one that removes it.

**Case: `gap` does not pad the edges**

```css
.row { display: flex; gap: 1rem; }
```

```html
<div class="row"><span>A</span><span>B</span></div>
```

Result: 1rem between A and B. No extra 1rem before A or after B from `gap`.

Why: gap is between items. Outer space is padding or margin on the container.

**Case: `contents`**

```css
.wrap { display: contents; }
```

```html
<ul>
  <li class="wrap"><a href="/">Home</a></li>
</ul>
```

Result: the `li` box disappears. The link participates in the `ul` as if the `li` were not there, which can also drop the list-item role. Use this only when you understand that the element is gone for layout and often for accessibility.

Why: `contents` is not `none`. The children remain. The element you selected does not generate a box.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `width` on an inline `span` | No change | `inline-block`, block, flex, or grid |
| `opacity: 0` to disable a button | It can still be clicked | `display: none`, `hidden`, or the `disabled` attribute |
| `gap` expected to inset the container | The edges stay flush | Padding on the container |
| `display: contents` on a list item or a button | The accessible role can disappear | Do not use it on interactive elements |

### Tip

Decide what should happen to the gap and to the keyboard. If both must remain, do not use `display: none`.

### Practice

Lay out three spans in a row with 0.75rem between them and no extra gap at the ends from that spacing.

### Solution

```css
.row {
  display: flex;
  gap: 0.75rem;
}
```

```html
<div class="row"><span>A</span><span>B</span><span>C</span></div>
```

Result: A, B, and C in a row. 0.75rem between each pair. The row's edges are the edges of A and C.

## 7. Position and stacking

### Concept

`position` chooses what `top`, `right`, `bottom`, and `left` are measured against. `static` is the default, and the offsets do nothing. `relative` moves the element from where it would have been and leaves the original gap. `absolute` takes it out of flow and measures from the nearest ancestor that is not `static`. `fixed` measures from the viewport. `sticky` is in flow until a scroll threshold, then pinned inside its parent.

`z-index` only applies to positioned elements and to flex or grid items. A larger value paints later. The comparison happens inside a stacking context. A child cannot climb out of a parent that created a context with `opacity` less than 1, a `transform`, or its own `z-index`.

`float` pulls a box to the side and lets text wrap. Flex and grid cover the jobs floats used to do.

### Terms

| Term | Meaning |
| --- | --- |
| Containing block | The box absolute offsets are measured from |
| In flow | The element still occupies its normal gap |
| Stacking context | A local z-index world. Children are compared inside it |
| `z-index` | Paint order inside that context. Bigger paints later |

### Properties

| `position` | The offsets are relative to |
| --- | --- |
| `static` | Normal flow. `top` does nothing |
| `relative` | Where the element would have been. The gap stays |
| `absolute` | The nearest ancestor that is not `static`. Removed from flow |
| `fixed` | The viewport, unless a transformed ancestor changes that |
| `sticky` | Flow, until it sticks inside its parent |

### Examples

**Case: absolute is trapped by a relative parent**

```css
.card { position: relative; }
.badge {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
}
```

```html
<div class="card">
  <span class="badge">New</span>
  <h2>Irvine</h2>
</div>
```

Result: `New` sits 0.5rem from the card's top and right. The heading does not move aside for it.

Why: `.card` is the containing block because it is `relative`. Without that, the badge looks for another ancestor and often ends up against the page.

**Case: relative keeps the gap**

```css
h2 { position: relative; top: 8px; }
```

Result: the heading paints 8px lower. The space it originally occupied stays. The next paragraph does not slide up.

Why: `relative` changes where it is painted, not the gap reserved in the flow.

**Case: sticky stops at the parent**

```css
h2 { position: sticky; top: 0; background: white; }
```

Result: the heading scrolls with the page until it would pass the top of the viewport, then it pins. When its parent scrolls away, the heading goes with the parent.

Why: sticky is limited by the parent's box. A parent with `overflow: hidden` often prevents the stick.

**Case: z-index cannot escape a context**

```css
.card { position: relative; z-index: 1; opacity: 0.99; }
.badge { position: absolute; z-index: 1000; }
.other { position: relative; z-index: 2; }
```

Result: a badge with `z-index: 1000` still paints under `.other` if `.other` comes later and the card created a stacking context. The 1000 is only "1000 inside the card".

Why: `opacity` less than 1 creates a stacking context. The child's z-index is compared with its siblings inside the card, then the card as a whole is compared with `.other`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `absolute` with no positioned ancestor | The badge jumps to the page edge | `position: relative` on the card |
| `top` on a `static` element | Nothing moves | Set `position` |
| A huge `z-index` that still loses | The parent is the thing being stacked | Raise the parent, or remove the extra context |
| `sticky` inside `overflow: hidden` | It never pins | Let the parent overflow visibly |

### Tip

Absolute children need a positioned parent. Say which element is the card before you write `top` on the badge.

### Practice

Put a `New` badge in the top-right corner of `.card`, 0.5rem from both edges, without pushing the heading.

### Solution

```css
.card { position: relative; }
.badge {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
}
```

Result: the badge overlays the corner. The heading keeps its normal place.

## 8. Flexbox

### Concept

Flex is for one row or one column of items that should share space. The main axis follows `flex-direction`. In a row, main is horizontal and cross is vertical. `justify-content` aligns the main axis. `align-items` aligns the cross axis. `flex: 1` means grow, shrink, and a basis of 0. `margin-left: auto` on an item eats the free space before it and pushes that item to the end. `order` changes visual order only. The keyboard still follows the HTML.

### Terms

| Term | Meaning |
| --- | --- |
| Container | The element with `display: flex` |
| Item | A child of that element. Text nodes are wrapped |
| Main axis | The direction of `flex-direction` |
| Cross axis | The other direction |
| Basis | The size before grow and shrink |

### Properties

| Container property | Values you will use |
| --- | --- |
| `flex-direction` | `row`, `row-reverse`, `column`, `column-reverse` |
| `flex-wrap` | `nowrap`, `wrap` |
| `justify-content` | `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-items` | `stretch`, `flex-start`, `flex-end`, `center`, `baseline` |
| `align-content` | Extra cross-axis space when there are multiple lines |
| `gap` | Between items |

| Item property | Meaning |
| --- | --- |
| `flex-grow` | Share of extra space. `0` means do not grow |
| `flex-shrink` | How willingly it shrinks. `0` means do not shrink |
| `flex-basis` | Starting size. `auto` uses width or height |
| `flex` | Shorthand: grow, shrink, basis. `flex: 1` means `1 1 0` |
| `align-self` | Override `align-items` for one item |
| `order` | Visual order only |

### Examples

**Case: space between the ends**

```css
.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 0.75rem;
}
```

```html
<div class="toolbar">
  <strong>Acme</strong>
  <a href="/search">Search</a>
</div>
```

Result: the name sits on the left, the link on the right, both vertically centered. If you add a third item, `space-between` puts the first at the start, the last at the end, and equal space between the pairs.

Why: `justify-content` distributes extra main-axis space. `align-items` positions each item on the cross axis.

**Case: `flex: 1` fills the leftover**

```css
.toolbar { display: flex; gap: 0.75rem; }
.search { flex: 1 1 12rem; }
```

```html
<div class="toolbar">
  <input class="search">
  <button>Go</button>
</div>
```

Result: the input starts from a 12rem basis and takes the leftover width. The button stays at its content size.

Why: grow is 1, so extra space goes to the input. Shrink is 1, so it may get smaller than 12rem. The basis is the starting offer.

**Case: `margin-left: auto` pushes one item**

```css
.toolbar { display: flex; }
.toolbar a { margin-left: auto; }
```

```html
<div class="toolbar">
  <strong>Acme</strong>
  <a href="/search">Search</a>
</div>
```

Result: the link sits on the right. The name stays on the left.

Why: auto margin consumes free space on that side. This is the two-item version of "logo left, links right" without `space-between`. With three items, `space-between` spreads all of them. Auto margin pushes only the item that has it.

**Case: `order` does not change Tab**

```css
.toolbar { display: flex; }
.second { order: -1; }
```

```html
<div class="toolbar">
  <button>First</button>
  <button class="second">Second</button>
</div>
```

Result: `Second` paints on the left. Tab still lands on `First` first.

Why: `order` is visual. Source order is what the keyboard and the screen reader follow.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `justify-content` expected to center vertically in a row | It centers horizontally | `align-items: center` |
| `flex: 1` read as "basis 1%" or ignored | The item may not grow the way you pictured | Remember `1 1 0` |
| `order` used to fix the reading order | Keyboard users still hear the HTML order | Move the HTML |
| Forgetting `flex-wrap` | Items shrink into an unreadable row | `flex-wrap: wrap` and a sensible basis |

### Tip

Say the axis before you pick a property. "Row, so main is horizontal." `justify-content` is that axis. `align-items` is the other one.

### Practice

Build a toolbar with the word `Acme` on the left and a `Search` link on the right, vertically centered.

### Solution

```css
.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

```html
<div class="toolbar">
  <strong>Acme</strong>
  <a href="/search">Search</a>
</div>
```

Result: the two items sit at opposite ends of the row and share a center line.

## 9. Grid

### Concept

Grid is for rows and columns together. You describe the tracks, then place items into them. `grid-column: 1 / 3` starts at line 1 and stops at line 3, which is two columns. `1 / -1` stretches across every column. Named areas must form a rectangle. A hole is written `.`.

Flex is one axis. Grid is both. A toolbar is flex. A page with a header, a side column, and a footer is grid.

### Terms

| Term | Meaning |
| --- | --- |
| Track | One column or one row |
| Line | The edge between tracks. Line numbers start at 1 |
| Area | A rectangle of cells with a name |
| `fr` | A share of the space left after fixed tracks |
| `minmax` | A track that will not shrink below the first size |

### Properties

| Property | Meaning |
| --- | --- |
| `grid-template-columns` | Track sizes. `12rem 1fr 1fr` is one fixed column and two shares |
| `repeat(3, 1fr)` | Three equal columns |
| `repeat(auto-fit, minmax(16rem, 1fr))` | As many columns as fit, each at least 16rem |
| `grid-column: 1 / 3` | From line 1 to line 3. Two columns |
| `grid-column: 1 / -1` | Every column |
| `grid-template-areas` | A picture of the names |
| `justify-items` / `align-items` | Align inside the cell |
| `gap` | Between tracks |

| Job | Flex | Grid |
| --- | --- | --- |
| Axis | One, set by `flex-direction` | Rows and columns |
| A child spans tracks | No tracks. Use basis or order | `grid-column` and `grid-row` |
| Pick it | A toolbar, a row of cards | A page skeleton |

### Examples

**Case: a span across all columns**

```css
.stats {
  display: grid;
  grid-template-columns: 12rem 1fr 1fr;
  gap: 1rem;
}
.stats h2 { grid-column: 1 / -1; }
```

```html
<section class="stats">
  <h2>Irvine</h2>
  <p>Sales</p>
  <p>42</p>
  <p>Pending</p>
</section>
```

Result: the heading occupies the full width. The three paragraphs sit in the three columns of the next row.

Why: `1 / -1` is "from the first line to the last line". Three paragraphs then auto-place into the three tracks.

**Case: named areas have to be rectangular**

```css
.page {
  display: grid;
  grid-template-columns: 12rem 1fr;
  grid-template-areas:
    "header header"
    "nav    main"
    "footer footer";
}
.page > header { grid-area: header; }
.page > nav { grid-area: nav; }
.page > main { grid-area: main; }
.page > footer { grid-area: footer; }
```

Result: a full-width header, a 12rem nav beside the main column, and a full-width footer.

Why: each name forms a rectangle. `"header nav" / "header main"` would also be a rectangle for `header` down the side. A name that does not form a rectangle is invalid and the areas do not place.

**Case: `auto-fit` and `minmax`**

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
  gap: 1rem;
}
```

Result: as many columns as fit. Each is at least 16rem. On a narrow screen there is one column. The columns share the leftover space.

Why: `minmax` stops a column from becoming unreadably thin. `auto-fit` collapses empty tracks. `auto-fill` would keep empty tracks.

**Case: line numbers count the edges**

```css
.item { grid-column: 1 / 3; }
```

On a three-column grid, result: the item covers columns 1 and 2. It stops at line 3. It does not cover three columns.

Why: the numbers are lines, not column indexes. Two lines bound one column. `1 / 3` bounds two columns.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `grid-column: 1 / 3` read as columns 1, 2, and 3 | You get two columns | Count the lines |
| A broken `grid-template-areas` picture | Items fall back to auto placement | Make every name a rectangle |
| Grid for a single toolbar | More syntax than the job needs | Flex |
| `1fr 1fr` expected to ignore a long word | The minimum size of content can overflow | `minmax(0, 1fr)` |

### Tip

Sketch the areas as a little ASCII picture before you write numbers. If you cannot draw the rectangle, the template is wrong.

### Practice

Build a page grid with a full-width header, a 12rem nav, a flexible main, and a full-width footer.

### Solution

The named-area rule in the second example of this section.

Result: header and footer span the page. Nav is 12rem. Main takes the rest.

## 10. Typography

### Concept

Font properties inherit. Set them on `body` and override headings. End `font-family` with a generic family so an unknown name still has a fallback. Prefer `rem` for `font-size` so a user's default font size scales the page. `line-height` should be unitless. `text-overflow: ellipsis` only works with `overflow: hidden` and `white-space: nowrap`.

Link states are selectors. `a { color: blue; }` hits every `a`, including ones you did not mean, and it does not by itself show keyboard focus.

### Terms

| Term | Meaning |
| --- | --- |
| Generic family | `serif`, `sans-serif`, `monospace`. The last name in the list |
| Weight | `400` is normal. `700` is bold. The face must contain that weight |
| Unitless line-height | A multiplier of the element's font size |
| `:focus-visible` | A ring for keyboard focus, not for every mouse click |

### Properties

| Property | Notes |
| --- | --- |
| `font-family` | A list. The first installed face wins |
| `font-size` | Prefer `rem` |
| `font-weight` | `400`, `700`, or a number the face actually has |
| `font-style` | `normal`, `italic` |
| `line-height` | Unitless |
| `letter-spacing` | Extra space between characters |
| `text-align` | `left`, `right`, `center`, `justify` |
| `text-decoration` | `none`, `underline` |
| `text-transform` | `uppercase`, `lowercase`, `capitalize` |
| `white-space` | `normal`, `nowrap`, `pre`, `pre-wrap` |
| `text-overflow` | `ellipsis`, with hidden overflow and nowrap |

### Examples

**Case: the family list**

```css
body {
  font-family: Georgia, "Times New Roman", serif;
  font-size: 1.125rem;
  line-height: 1.5;
  color: #1c1c1c;
}
```

Result: Georgia if it is installed, otherwise Times New Roman, otherwise any serif. The page text is 1.125 times the user's root size.

Why: a single name fails on a machine that does not have it. The generic family is the guaranteed last step. Quote a family that contains a space.

**Case: ellipsis needs three declarations**

```css
.title {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

Result: a long title stays on one line and ends in `…`. Without `nowrap`, the text wraps and there is nothing to ellipsize. Without `hidden`, the overflow stays visible.

Why: `text-overflow` only decorates the clipped edge. The other two create that edge.

**Case: link states**

```css
a:link { color: #0b57d0; }
a:visited { color: #5b2a86; }
a:hover { text-decoration: none; }
a:focus-visible { outline: 2px solid #0b57d0; outline-offset: 2px; }
```

Result: an unvisited link is blue. A visited one is purple. Hover removes the underline. Keyboard focus shows a ring. A mouse click does not keep that ring once `:focus-visible` is the selector.

Why: these are states of one element. One `a` rule cannot tell them apart.

**Case: a heading that is not an `h1` size hack only**

```css
h1 {
  font-size: 2rem;
  font-weight: 700;
  line-height: 1.2;
}
```

Result: the `h1` is twice the root size, bold, with tighter lines than the body. The outline level is still `h1` because the HTML said so.

Why: CSS changes the look. It does not change the heading level.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A font name with a space, unquoted | The family is parsed as two names | Quote it |
| `text-overflow: ellipsis` alone | No ellipsis | Add `nowrap` and `overflow: hidden` |
| `outline: none` on `a:focus` | No keyboard ring | `:focus-visible` with a visible outline |
| `line-height` in pixels | Nested sizes and user fonts fight the line box | A unitless multiplier |

### Tip

Set type on `body`. Headings only need the properties that differ.

### Practice

Set the body to a serif stack, `1.125rem`, and line-height `1.5`.

### Solution

The `body` rule in the first example of this section.

Result: body text uses the stack, the root-relative size, and a 1.5 multiplier.

## 11. Backgrounds, borders, and effects

### Concept

`background` is a shorthand that can set color, image, position, size, and repeat in one line. A border is invisible until it has a style. `solid`, `dashed`, and `dotted` are styles. `box-shadow` does not take up layout space. `opacity` fades the element and its children. An alpha background fades only the paint you put it on. `object-fit` controls how an image fills the box you gave it.

### Terms

| Term | Meaning |
| --- | --- |
| Shorthand | One property that sets several longhands. A later longhand can override one piece |
| `cover` | Fill the box, cropping the extra |
| `contain` | Fit the whole image, leaving bars |
| Inset shadow | Drawn inside the border |

### Properties

| Property | Values |
| --- | --- |
| `background-color` | Any color |
| `background-image` | `url(...)` or a gradient |
| `background-repeat` | `repeat`, `no-repeat` |
| `background-position` | `center`, `top left` |
| `background-size` | `cover`, `contain`, or a length |
| `border` | Width, style, color. Style is required |
| `border-radius` | One value, or four clockwise from the top left |
| `box-shadow` | Offset-x, offset-y, blur, spread, color |
| `opacity` | The whole element |
| `object-fit` | On `img` and `video`: `cover`, `contain`, `fill` |
| `cursor` | `pointer`, `text`, `not-allowed` |

### Examples

**Case: a border with no style**

```css
.panel { border: 4px #ddd; }
```

Result: no visible border. The width and color are stored, and the style is `none`.

Why: `border: 1px solid #ddd` includes the style. Width alone does not paint.

**Case: shadow does not push siblings**

```css
.panel { box-shadow: 0 8px 24px rgb(0 0 0 / 0.08); }
```

Result: a soft shadow under the panel. The next element does not move down to make room.

Why: shadow is paint, like outline. Margin is what creates a gap.

**Case: `opacity` versus an alpha background**

```css
.fade { opacity: 0.5; }
.wash { background: rgb(255 255 255 / 0.5); }
```

Result: `.fade` makes its text and its children half transparent. `.wash` tints the background and leaves the text fully opaque.

Why: `opacity` is the element. A color alpha is one layer.

**Case: `object-fit`**

```css
img { width: 12rem; height: 8rem; object-fit: cover; }
```

Result: the image fills the 12 by 8 box. The extra is cropped. `contain` would show the whole photo and leave empty bands.

Why: the box and the file have different ratios. `object-fit` chooses which one wins. Without it, the image stretches (`fill`).

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `border: 1px #ddd` | Nothing is drawn | Add `solid` |
| `opacity` to tint a card | The words fade | Alpha on `background` |
| Shadow used as spacing | The next box overlaps the shadow | Margin |
| `object-fit` without a width and height | There is no box to fit into | Set both sizes |

### Tip

If a border "does not work", the style is missing. Color and width are not enough.

### Practice

Give `.panel` a white background, a 1px solid `#ddd` border, a 0.5rem radius, and a soft shadow.

### Solution

```css
.panel {
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 0.5rem;
  box-shadow: 0 8px 24px rgb(0 0 0 / 0.08);
}
```

Result: a white rounded panel with a hairline border and a shadow that does not change the layout.

## 12. Pseudo-classes and pseudo-elements

### Concept

A pseudo-class matches a state or a position among siblings. A pseudo-element matches a piece the HTML did not write. `::before` and `::after` generate boxes, and they do nothing until you set `content`, even `content: ""`. `:is()` takes the specificity of its most specific argument. `:where()` matches the same elements and has zero specificity. `:nth-child` counts elements, not just the tag you think of.

### Terms

| Term | Meaning |
| --- | --- |
| Pseudo-class | One colon. A state or a position. `:hover` |
| Pseudo-element | Two colons. A generated piece. `::before` |
| `:nth-child(2n)` | Even positions among the siblings. `2n+1` is odd |
| `:focus-visible` | Focus that should show a ring |

### Properties

| Pseudo-class | Matches |
| --- | --- |
| `:hover` | The pointer is over it |
| `:focus` | It has focus |
| `:focus-visible` | Focus that should show a ring |
| `:first-child` / `:last-child` | First or last sibling |
| `:not(.hidden)` | Elements that do not match the inner selector |
| `:disabled`, `:required`, `:checked` | Form states |
| `:empty` | No children |
| `:is(h1, h2)` | Any in the list. Specificity is the most specific argument |
| `:where(h1, h2)` | The same match, specificity zero |

| Pseudo-element | Use |
| --- | --- |
| `::before` / `::after` | Generated boxes. They need `content` |
| `::placeholder` | The placeholder text |
| `::marker` | The list bullet or number |
| `::selection` | Highlighted text |

### Examples

**Case: `:first-child` is about siblings, not about being the first `p`**

```html
<section>
  <h2>Irvine</h2>
  <p>Active listings.</p>
</section>
```

```css
p:first-child { font-weight: 700; }
```

Result: the paragraph is not bold. It is the second child.

Why: `:first-child` means "this element is the first child of its parent", not "the first paragraph". `p:first-of-type` would match it.

**Case: even rows**

```css
tr:nth-child(2n) { background: #f6f6f6; }
```

Result: rows 2, 4, 6 are shaded. Row 1 is not. The count includes every `tr` sibling, and it starts at 1.

Why: `2n` is the even positions. `2n+1` is the odd ones. If a `thead` row is a sibling in a different parent, it is not in this count. `tbody tr` only counts inside the body.

**Case: `::before` without `content` does nothing**

```css
.card::before {
  display: block;
  height: 4px;
  background: #0b57d0;
}
```

Result: no blue bar.

```css
.card::before {
  content: "";
  display: block;
  height: 4px;
  background: #0b57d0;
}
```

Result: a 4px blue bar at the start of the card.

Why: generated boxes are not created until `content` is set. An empty string is enough when you do not want text.

**Case: `:where` loses on purpose**

```css
:where(h1, h2) { margin-top: 0; }
h2 { margin-top: 2rem; }
```

Result: the `h2` has a 2rem top margin. The `:where` rule does not fight it.

Why: `:where` has zero specificity, so a single element selector later, or anywhere of higher specificity, wins. `:is(h1, h2)` would have had the specificity of `h2` and the tie would depend on order.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `p:first-child` when a heading comes first | The rule never matches | `:first-of-type`, or change the tree |
| `::before` with no `content` | No box | `content: ""` |
| `:hover` as the only focus style | Keyboard users see nothing | `:focus-visible` |
| One colon on a pseudo-element, in new code | It may work for the old four, and it is easy to misread | `::before` |

### Tip

`:first-child` asks "am I the first child?", not "am I the first of my tag?"

### Practice

Add a 4px navy bar to the top of `.card` without adding an HTML element.

### Solution

```css
.card::before {
  content: "";
  display: block;
  height: 4px;
  background: #0b57d0;
}
```

Result: a blue strip at the start of every card. The HTML is unchanged.

## 13. Transitions and transforms

### Concept

`transform` moves, scales, or rotates an element without pushing its siblings. `transition` animates the change between two values of a property. Name the property. `all` animates everything, including colors you did not mean to fade. A transition runs when a value changes, usually because `:hover` or a class changed. `@keyframes` with `animation` can loop. Respect `prefers-reduced-motion`.

### Terms

| Term | Meaning |
| --- | --- |
| Transform | A paint-time move. Layout stays |
| Transition | One change from the old value to the new value |
| Animation | Keyframes that can repeat |
| Duration | How long the change takes |
| Reduced motion | The reader asked the OS for less animation |

### Properties

| `transform` function | Effect |
| --- | --- |
| `translate(x, y)` | Move. Does not push siblings |
| `translateY(-2px)` | Move up |
| `scale(1.05)` | Resize from the transform origin |
| `rotate(8deg)` | Spin |

| Transition piece | Example |
| --- | --- |
| Property | `transform`. Prefer a name over `all` |
| Duration | `200ms` |
| Timing | `ease`, `linear`, `ease-in-out` |
| Delay | A fourth time value |

### Examples

**Case: hover lifts the card and the text below does not move**

```css
.card { transition: transform 200ms ease; }
.card:hover { transform: translateY(-2px); }
```

Result: on hover the card paints 2px higher. The next card stays put. Leaving the card animates it back.

Why: transform does not change the layout gap. A `margin-top: -2px` would pull the next card up.

**Case: a property that was not named does not animate**

```css
.card { transition: transform 200ms ease; }
.card:hover {
  transform: translateY(-2px);
  background: #f6f6f6;
}
```

Result: the move eases. The background snaps.

Why: only `transform` is in the transition list. Add `background-color 200ms ease` if the color should ease too.

**Case: reduced motion**

```css
@media (prefers-reduced-motion: reduce) {
  .card { transition: none; }
}
```

Result: a reader who asked the operating system for less motion gets the new state immediately.

Why: the hover change can still happen. The animation does not.

**Case: `scale` grows from the center**

```css
.card:hover { transform: scale(1.05); }
```

Result: the card paints 5% larger around its center and overlaps its neighbors. Their layout boxes stay the same.

Why: the default transform origin is the center. Overlap is paint, not a flex grow.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `transition: all` | Every property change animates, including ones you set from script | Name the properties |
| Animating `top` on an absolute element for a hover | It works and it is more expensive than `transform` | `translateY` |
| No reduced-motion query | Some readers get motion they asked not to have | `prefers-reduced-motion` |
| Expecting transform to reflow text | The gap stays | Margin or flex if you need reflow |

### Tip

If neighbors should move, use layout (`margin`, flex, grid). If only the paint should move, use `transform`.

### Practice

On hover, move `.card` up 2px over 200ms, and turn that motion off when the reader prefers reduced motion.

### Solution

```css
.card { transition: transform 200ms ease; }
.card:hover { transform: translateY(-2px); }
@media (prefers-reduced-motion: reduce) {
  .card { transition: none; }
}
```

Result: a short lift for most readers. No transition for readers who opted out.

## 14. Variables and media queries

### Concept

A custom property is a named value that inherits, like `color`. Define tokens on `:root`. `var(--name, fallback)` uses the fallback only when `--name` is not set. An invalid value at the point of use makes the declaration invalid. It does not automatically use the fallback if the variable exists and holds a bad value.

`min-width` queries are mobile-first: the base CSS is the narrow layout, and the query adds columns when there is room. `max-width` is the opposite. A media query does not replace flex wrap. Start with one column.

### Terms

| Term | Meaning |
| --- | --- |
| Custom property | `--name` |
| `:root` | The `html` element, with higher specificity than `html` |
| Fallback | The second argument of `var()` |
| Mobile-first | Base styles, then `min-width` |
| `prefers-color-scheme` | The reader's light or dark choice |

### Properties

| Idea | Detail |
| --- | --- |
| `var(--space)` | The inherited value of `--space` |
| `var(--panel, #fff)` | `#fff` when `--panel` was never set |
| `@media (min-width: 48rem)` | Applies at 48rem and wider |
| `@media (max-width: 40rem)` | Applies at 40rem and narrower |
| `@media (prefers-color-scheme: dark)` | A dark UI was requested |
| `@media (prefers-reduced-motion: reduce)` | Less motion was requested |

### Examples

**Case: a token inherited by a card**

```css
:root {
  --space: 1rem;
  --ink: #1c1c1c;
}
.card {
  color: var(--ink);
  padding: var(--space);
}
```

Result: the card's text is `#1c1c1c` and its padding is 1rem. A nested element can read `--ink` too, because custom properties inherit.

Why: you change `--ink` in one place. Every `var(--ink)` follows.

**Case: the fallback runs only when the variable is missing**

```css
.card { background: var(--panel, #fff); }
```

Result: the background is white, because `--panel` was not defined.

```css
:root { --panel: 12; }
.card { background: var(--panel, #fff); }
```

Result: the background declaration is invalid. The fallback is not used. `12` is not a color.

Why: the fallback substitutes for a missing variable. It does not repair a bad value.

**Case: mobile-first columns**

```css
.stats { display: grid; grid-template-columns: 1fr; gap: 1rem; }
@media (min-width: 40rem) {
  .stats { grid-template-columns: 1fr 1fr; }
}
```

Result: one column on a narrow viewport. Two columns once the viewport is at least 40rem wide.

Why: the base rule is the small screen. The query adds capability. You do not undo a desktop layout inside `max-width`.

**Case: dark scheme**

```css
:root { --ink: #1c1c1c; --panel: #fff; }
@media (prefers-color-scheme: dark) {
  :root { --ink: #f2f2f2; --panel: #1c1c1c; }
}
body { color: var(--ink); background: var(--panel); }
```

Result: readers who asked for dark get light text on a dark page. Everyone else gets the first pair.

Why: the components use the variables. The query only changes the tokens.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A bad variable value, expecting the fallback | The property is dropped | Validate the value. The fallback is for "unset" |
| Desktop CSS first, then a pile of `max-width` overrides | Every new phone size undoes the desktop | `min-width` additions |
| Variables used like JavaScript | They do not compute `2 + 2` unless you use a math function | `calc(var(--space) * 2)` |
| Forgetting that custom properties inherit | A card set `--ink` and the children change too | Set the variable on the element that should own it |

### Tip

Put the tokens on `:root`. Put the queries next to the component they change, not in one giant block at the bottom, once the file is long.

### Practice

Define `--space: 1rem` on `:root` and use it as `.card` padding. At `40rem` and wider, make `.stats` two equal columns.

### Solution

```css
:root { --space: 1rem; }
.card { padding: var(--space); }
.stats { display: grid; grid-template-columns: 1fr; }
@media (min-width: 40rem) {
  .stats { grid-template-columns: 1fr 1fr; }
}
```

Result: cards pad by 1rem. The stats grid is one column, then two.

## 15. A layout you can rebuild

### Concept

This is the HTML practice page from the HTML course, plus a stylesheet. Narrow screens stay one column. At `48rem` the main area becomes two columns. `nav` is a row. The form is a grid stack. Predict the narrow result before you resize.

### Terms

| Term | Meaning |
| --- | --- |
| Reset of box sizing | `border-box` on every element before any widths |
| One-column default | No media query required for the phone |
| Breakpoint | The `min-width` where a second column appears |

### Properties

The properties are the ones from earlier sections: `box-sizing`, `margin`, `font-family`, `line-height`, `padding`, `display`, `gap`, `color`, `grid-template-columns`, and `max-width`.

### Examples

**Case: the stylesheet**

```css
*, *::before, *::after { box-sizing: border-box; }
body {
  margin: 0;
  font-family: Georgia, serif;
  line-height: 1.5;
  color: #1c1c1c;
}
header, main, footer { padding: 1rem; }
nav { display: flex; gap: 1rem; }
nav a { color: inherit; }
form { display: grid; gap: 0.5rem; max-width: 24rem; }
@media (min-width: 48rem) {
  main {
    display: grid;
    grid-template-columns: 1fr 16rem;
    gap: 1.5rem;
  }
}
```

Result: below 48rem, header, main, and footer stack. The nav links sit in a row with 1rem between them. The form is a vertical grid no wider than 24rem. At 48rem and above, `main`'s children sit in two columns. The second column is 16rem.

Why: nothing in the base CSS creates a second column. The query adds it. `color: inherit` on the links uses the body color instead of the browser's blue, and it still inherits if a parent sets a color.

**Case: which selector wins**

```css
a { color: blue; }
nav a { color: inherit; }
```

Result: links in the nav use the inherited color. Links outside the nav stay blue.

Why: `nav a` is one element plus another element. `a` is one element. `0, 0, 2` beats `0, 0, 1`.

**Case: padding and `border-box`**

`header { padding: 1rem; }` does not make the header wider than the viewport.

Why: `border-box` is already on every element, and the header's width is the default `auto`, which fills the parent. The padding comes out of that width.

**Case: the form grid**

```css
form { display: grid; gap: 0.5rem; max-width: 24rem; }
```

Result: the label, the input, and the button stack. There is 0.5rem between them. The form never grows past 24rem even when `main` is wide.

Why: a grid with one implicit column stacks the children. `gap` replaces margins between them. `max-width` caps the line length.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| The two-column grid written outside the media query | Phones get a crushed 16rem column | Keep it inside `min-width` |
| `nav a { color: inherit; }` missing | Links stay the default blue | Inherit, or set a color on purpose |
| Box sizing added after widths were calculated by eye | Padding starts overflowing | Put `border-box` first |
| Expecting `header` to become two columns | The grid is on `main` only | Move the rule, or add one for `header` |

### Tip

Resize from narrow to wide. If the narrow page already needs a horizontal scrollbar, fix that before you add the breakpoint.

### Practice

Link this stylesheet to the HTML search page. At a narrow width, confirm the form stacks. Widen the window past 48rem and confirm `main` has two columns.

### Solution

Use the stylesheet in the first example and the page from HTML section 15.

Result: one column when the window is under 48rem. At 48rem and wider, the form and the glossary sit side by side if they are the two children of `main`. If the glossary is inside the form's sibling structure as written, the form is column one and the glossary section is the 16rem column.

## Drill

1. Write a class rule and a descendant rule, and say which wins against `p`.
2. With the selector tree from section 2, match only the paragraph immediately after the heading.
3. Set `box-sizing` so a 20rem card with padding stays 20rem on the outside.
4. Center the items of a row on the cross axis and push the last item to the end with an auto margin.
5. Add columns only at `40rem` and wider.

### Solution 1

`.card p` beats `p`. Specificity `0, 1, 1` beats `0, 0, 1`.

### Solution 2

```css
h2 + p { font-weight: 700; }
```

That is `Active listings.` only.

### Solution 3

```css
.card {
  box-sizing: border-box;
  width: 20rem;
  padding: 1rem;
}
```

### Solution 4

```css
.toolbar { display: flex; align-items: center; }
.toolbar a { margin-left: auto; }
```

### Solution 5

```css
.stats { display: grid; grid-template-columns: 1fr; }
@media (min-width: 40rem) {
  .stats { grid-template-columns: 1fr 1fr; }
}
```
