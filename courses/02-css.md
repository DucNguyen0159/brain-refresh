# CSS

Course 2 of 15. Previous: [01-html.md](01-html.md). Next: [03-javascript.md](03-javascript.md).

CSS describes how the HTML tree looks. A rule is a selector and a list of declarations. The cascade decides which declaration wins when several match the same element.

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

```css
p {
  color: #1a1a1a;
  line-height: 1.5;
}
```

| Piece | In the example |
| --- | --- |
| Selector | `p` |
| Declaration | `color: #1a1a1a;` |
| Property | `color` |
| Value | `#1a1a1a` |
| Comment | `/* note */` |

| Where | How | When |
| --- | --- | --- |
| External file | `<link rel="stylesheet" href="styles.css">` | Normal pages. Cached. One place to edit |
| `style` in the head | `<style> p { color: black; } </style>` | A single demo file |
| Inline | `<p style="color: black">` | One element, highest ordinary specificity. Hard to maintain |

A missing semicolon can swallow the next declaration. A misspelled property is ignored. The rest of the rule still applies.

## 2. Selectors

```css
h1 { }
.card { }
#hero { }
article p { }
.card > h2 { }
h2 + p { }
a[href^="https"] { }
h1, h2 { }
* { }
```

| Selector | Matches |
| --- | --- |
| `h1` | Every `h1` |
| `.card` | Every element with class `card` |
| `#hero` | The one element with that id |
| `*` | Everything |
| `article p` | A `p` anywhere inside an `article` (descendant) |
| `.card > h2` | An `h2` that is a direct child of `.card` |
| `h2 + p` | A `p` immediately after an `h2` |
| `h2 ~ p` | Every `p` after an `h2`, same parent |
| `a[href]` | An `a` that has `href` |
| `a[href="x"]` | Exact attribute value |
| `a[href^="https"]` | Value starts with `https` |
| `a[href$=".pdf"]` | Value ends with `.pdf` |
| `a[href*="report"]` | Value contains `report` |
| `input[type="email"]` | That attribute pair |
| `h1, h2` | Either. The declarations are shared |

Class and id selectors are the ones you will write most. Element selectors are for defaults, such as `body` and `a`.

## 3. The cascade, inheritance, and specificity

When two declarations set the same property on the same element, the winner is chosen in this order:

| Step | Rule |
| --- | --- |
| 1 | Importance. `!important` beats a normal declaration. Avoid it |
| 2 | Origin. Author styles beat browser defaults. A user `!important` can beat the author |
| 3 | Specificity. The more pointed selector wins |
| 4 | Order. If specificity ties, the later rule in the file wins |

Specificity is a triple: (ids, classes, elements).

| Selector | Specificity | Why |
| --- | --- | --- |
| `p` | 0, 0, 1 | One element |
| `.card` | 0, 1, 0 | One class |
| `#hero` | 1, 0, 0 | One id |
| `.card p` | 0, 1, 1 | One class and one element |
| `.card.featured` | 0, 2, 0 | Two classes on the same element |
| Inline `style` | beats selectors | Treated as heavier than an id |

`0, 1, 0` beats `0, 0, 12`. Twelve elements do not beat one class. An id beats a pile of classes. Do not stack ids to "win." Change the structure.

Some properties inherit from the parent. Some do not.

| Inherits | Does not inherit |
| --- | --- |
| `color`, `font`, `line-height`, `letter-spacing` | `margin`, `padding`, `border`, `background`, `width`, `display` |

```css
body { color: #222; }
p { color: inherit; } /* same as the default for color */
.card { border: 1px solid #ccc; } /* children do not grow a border */
```

`inherit` takes the parent's computed value. `initial` takes the property's default. `unset` inherits if the property inherits, otherwise it uses `initial`. `revert` goes back to the browser or user-agent style.

## 4. Colors and units

```css
.swatch {
  color: #1d4e89;
  background: rgb(255, 248, 240);
  border-color: hsl(210, 60%, 40%);
  outline: 2px solid color-mix(in srgb, blue, white 30%);
}
```

| Form | Example |
| --- | --- |
| Named | `navy`, `transparent`, `currentcolor` |
| Hex | `#fc0`, `#ffcc00`, `#ffcc0080` (last pair is alpha) |
| `rgb` / `rgba` | `rgb(255 204 0 / 0.5)` |
| `hsl` | `hsl(48 100% 50%)` |
| `currentcolor` | Whatever `color` computed to |

| Unit | Means |
| --- | --- |
| `px` | A CSS pixel. Stable. Good for borders and hairlines |
| `rem` | Relative to the root font size. Good for type and spacing |
| `em` | Relative to the element's font size. Compounds when nested |
| `%` | Relative to the parent, for the property in question |
| `vw` / `vh` | Percent of the viewport width or height |
| `ch` | Roughly the width of the `0` character |
| `fr` | A fraction of free space. Grid only |
| unitless `line-height` | A multiplier of the font size. Prefer this over `px` |
| unitless `0` | Zero does not need a unit |

`box-sizing: border-box` makes `width` include padding and border. Set it on every element at the start of a stylesheet. The content-box default makes a `width: 100%` element overflow as soon as you add padding.

```css
*, *::before, *::after { box-sizing: border-box; }
```

## 5. The box model

Every element is a content box, then padding, then border, then margin.

```css
.card {
  width: 20rem;
  padding: 1rem;
  border: 1px solid #ccc;
  margin: 0 0 1rem;
}
```

| Property | Shorthand order |
| --- | --- |
| `margin` / `padding` with 1 value | All four sides |
| 2 values | Vertical, horizontal |
| 3 values | Top, horizontal, bottom |
| 4 values | Top, right, bottom, left. Clockwise |
| `margin-block` / `padding-inline` | Logical sides. Follow the writing direction |

| Fact | Detail |
| --- | --- |
| Margin collapse | Vertical margins of adjacent blocks combine into the larger one. Horizontal margins do not collapse |
| Padding collapse | Never |
| Negative margin | Pulls neighbors closer. Allowed |
| `height` | Content height, unless `border-box` |
| `min-height` / `max-width` | Limits. `width: 100%; max-width: 40rem` is a common reading column |
| `overflow` | `visible`, `hidden`, `auto`, `scroll`. `auto` shows a scrollbar only when needed |

`outline` draws outside the border and does not take up space. `border` does. Do not remove a focus outline unless you replace it.

## 6. Display

`display` sets the formatting context.

| Value | The element |
| --- | --- |
| `block` | New line, fills the width |
| `inline` | In the line. Width and height are ignored |
| `inline-block` | In the line. Width, height, and vertical margins work |
| `none` | Removed from the layout. Not merely invisible |
| `flex` | A flex container. Children become flex items |
| `inline-flex` | Flex layout, sitting in a line |
| `grid` | A grid container |
| `contents` | The element's box disappears. Its children act as if they belong to the parent |

`visibility: hidden` keeps the gap and hides the paint. `opacity: 0` keeps the element hit-testable. `display: none` removes it from layout and from the accessibility tree.

```css
.row { display: flex; gap: 1rem; }
.stack { display: grid; gap: 0.5rem; }
```

`gap` is the space between children in flex and grid. It is not margin, so the first and last children do not grow an outer gap.

How the boxes leave, or stay in, the line:

| | In normal flow | Takes a gap | Offsets work | Typical use |
| --- | --- | --- | --- | --- |
| `display: block` | Yes | Yes, the full line | No, until you set `position` | A paragraph, a section |
| `display: inline` | Yes, in the line | The line box | No | A link inside a sentence |
| `display: none` | No | No | No | Hide it from layout and from assistive tech |
| `visibility: hidden` | Yes | Yes | No | Keep the gap, paint nothing |
| `position: relative` | Yes | Yes | Yes, from its own spot | A nudge, or an anchor for an absolute child |
| `position: absolute` | No | No | Yes, from the nearest positioned ancestor | A badge in the corner of a card |
| `position: fixed` | No | No | Yes, from the viewport | A bar that stays while the page scrolls |
| `position: sticky` | Yes, until it sticks | Yes | Yes, from the scrollport | A heading that pins inside its parent |

## 7. Position and stacking

| `position` | The offsets are relative to |
| --- | --- |
| `static` | Normal flow. `top` does nothing. This is the default |
| `relative` | Where the element would have been. The original gap stays |
| `absolute` | The nearest ancestor that is not `static`. Removed from flow |
| `fixed` | The viewport. Stays put while scrolling |
| `sticky` | Flow, until a scroll threshold, then fixed inside its parent |

```css
.badge {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
}
.card { position: relative; }
.toolbar {
  position: sticky;
  top: 0;
}
```

`z-index` only applies to positioned elements (and flex/grid items). A larger value paints later. The comparison happens inside a stacking context. A child cannot climb out of a parent that created a context with `opacity` less than 1, a `transform`, or its own `z-index`.

`float` pulls a box to the left or right and lets text wrap. `clear` stops an element from sitting beside a float. Flex and grid cover the jobs floats used to do. You will still meet floats in old code.

## 8. Flexbox

Use flex for one row or one column of items that should share space.

```css
.toolbar {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}
.toolbar .search { flex: 1 1 12rem; }
```

| Container property | Values you will use |
| --- | --- |
| `flex-direction` | `row`, `row-reverse`, `column`, `column-reverse` |
| `flex-wrap` | `nowrap`, `wrap` |
| `justify-content` | Main axis: `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-items` | Cross axis: `stretch`, `flex-start`, `flex-end`, `center`, `baseline` |
| `align-content` | Extra cross-axis space when there are multiple lines |
| `gap` | `row-gap` and `column-gap` |

The main axis follows `flex-direction`. In a row, main is horizontal and cross is vertical.

| Item property | Meaning |
| --- | --- |
| `flex-grow` | Share of extra space. `0` means do not grow |
| `flex-shrink` | How willingly it shrinks. `0` means do not shrink |
| `flex-basis` | Starting size before grow and shrink. `auto` uses width or height |
| `flex` | Shorthand: grow, shrink, basis. `flex: 1` means `1 1 0` |
| `align-self` | Override `align-items` for one item |
| `order` | Visual order. The DOM order stays the same for keyboards and screen readers |

`margin-left: auto` on a flex item eats the free space on that side and pushes the item to the end. That is the usual "logo left, links right" trick when you do not want `space-between`.

## 9. Grid

Use grid when rows and columns both matter.

```css
.stats {
  display: grid;
  grid-template-columns: 12rem 1fr 1fr;
  grid-template-rows: auto auto;
  gap: 1rem;
}
.stats h2 {
  grid-column: 1 / -1;
}
```

| Property | Meaning |
| --- | --- |
| `grid-template-columns` | Track sizes. `200px 1fr 1fr` is one fixed column and two equal shares |
| `repeat(3, 1fr)` | Three equal columns |
| `repeat(auto-fit, minmax(16rem, 1fr))` | As many columns as fit, each at least 16rem |
| `grid-column: 1 / 3` | Start at line 1, end at line 3. That is two columns |
| `grid-column: 1 / -1` | Stretch across every column |
| `grid-row` | The same idea for rows |
| `grid-area` | A named area, after `grid-template-areas` |
| `justify-items` / `align-items` | Align inside the cell |
| `justify-content` / `align-content` | Align the whole grid inside the container when there is extra space |

```css
.page {
  display: grid;
  grid-template-areas:
    "header header"
    "nav    main"
    "footer footer";
  grid-template-columns: 12rem 1fr;
}
.page > header { grid-area: header; }
.page > nav { grid-area: nav; }
.page > main { grid-area: main; }
.page > footer { grid-area: footer; }
```

Named areas must form a rectangle. A hole is written `.`.

Flex and grid divide the same job differently.

| Job | Flex | Grid |
| --- | --- | --- |
| Axis | One main axis, set by `flex-direction` | Rows and columns together |
| Track sizes | Each item's `flex-basis`, then grow and shrink | `grid-template-columns` and `grid-template-rows` |
| A child spans tracks | No tracks to span. Use `flex-basis` or `order` | `grid-column` and `grid-row` |
| Both axes aligned | `justify-content` on the main axis, `align-items` on the cross axis | `justify-items` and `align-items` inside cells, `justify-content` and `align-content` for the whole grid |
| Gap | `gap` | `gap` |
| Pick it | A toolbar, a row of cards, a stack | A page with a header, a side column, and a footer |

## 10. Typography

```css
body {
  font-family: Georgia, "Times New Roman", serif;
  font-size: 1.125rem;
  line-height: 1.5;
  color: #1c1c1c;
}
h1 {
  font-size: 2rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  line-height: 1.2;
}
```

| Property | Notes |
| --- | --- |
| `font-family` | A list. The browser uses the first one it has. End with a generic: `serif`, `sans-serif`, `monospace` |
| `font-size` | Prefer `rem` for the page, so a user font setting scales the layout |
| `font-weight` | `400` normal, `700` bold. Not every face has every weight |
| `font-style` | `normal`, `italic` |
| `line-height` | Unitless multiplier |
| `letter-spacing` | Extra space between characters |
| `text-align` | `left`, `right`, `center`, `justify` |
| `text-decoration` | `none`, `underline`. How you restyle links |
| `text-transform` | `uppercase`, `lowercase`, `capitalize` |
| `white-space` | `normal`, `nowrap`, `pre`, `pre-wrap` |
| `text-overflow` | `ellipsis` with `overflow: hidden` and `white-space: nowrap` |
| `word-break` / `overflow-wrap` | Where a long token may break |

A link's states need selectors, not only `a { }`:

```css
a:link { color: #0b57d0; }
a:visited { color: #5b2a86; }
a:hover { text-decoration: none; }
a:focus-visible { outline: 2px solid #0b57d0; outline-offset: 2px; }
```

## 11. Backgrounds, borders, and effects

```css
.panel {
  background: #fff url("grain.png") center / cover no-repeat;
  border: 1px solid #ddd;
  border-radius: 0.5rem;
  box-shadow: 0 8px 24px rgb(0 0 0 / 0.08);
}
```

| Property | Values |
| --- | --- |
| `background-color` | Any color |
| `background-image` | `url(...)`, or a gradient |
| `background-repeat` | `repeat`, `no-repeat`, `repeat-x` |
| `background-position` | `center`, `top left`, `50% 0` |
| `background-size` | `cover`, `contain`, or a length |
| `border` | width, style, color. Style is required or the border is invisible |
| `border-radius` | One value for all corners, or four clockwise from top left |
| `box-shadow` | offset-x, offset-y, blur, spread, color. `inset` draws inside |
| `opacity` | The whole element, including children. Prefer an alpha color when only the background should fade |
| `linear-gradient` | `linear-gradient(#fff, #e8eef5)` |
| `object-fit` | On `img` and `video`: `cover`, `contain`, `fill` |
| `cursor` | `pointer`, `text`, `not-allowed` |

`border-style` keywords: `solid`, `dashed`, `dotted`, `none`.

## 12. Pseudo-classes and pseudo-elements

A pseudo-class matches a state. A pseudo-element matches a piece the HTML did not write.

| Pseudo-class | Matches |
| --- | --- |
| `:hover` | Pointer over it |
| `:focus` | Keyboard or click focus |
| `:focus-visible` | Focus that should show a ring, usually keyboard |
| `:active` | Being pressed |
| `:first-child` / `:last-child` | First or last among siblings |
| `:nth-child(2n)` | Even positions. `2n+1` is odd |
| `:not(.hidden)` | Elements that do not match the inner selector |
| `:disabled`, `:required`, `:checked` | Form states |
| `:empty` | No children |
| `:is(h1, h2)` | Any in the list. Specificity is the most specific argument |
| `:where(h1, h2)` | Same match, specificity zero |

| Pseudo-element | Use |
| --- | --- |
| `::before` / `::after` | Generated boxes. They need `content`, even `content: ""` |
| `::placeholder` | The placeholder text |
| `::marker` | The list bullet or number |
| `::selection` | Text the user highlighted |

```css
.card::before {
  content: "";
  display: block;
  height: 4px;
  background: #0b57d0;
}
```

## 13. Transitions and transforms

```css
.card {
  transition: transform 200ms ease, box-shadow 200ms ease;
}
.card:hover {
  transform: translateY(-2px);
}
@media (prefers-reduced-motion: reduce) {
  .card { transition: none; }
}
```

| `transform` function | Effect |
| --- | --- |
| `translate(x, y)` | Move. Does not push siblings |
| `translateY(-2px)` | Move up |
| `scale(1.05)` | Resize from the transform origin |
| `rotate(8deg)` | Spin |
| `skew(10deg)` | Slant |

| Transition piece | Example |
| --- | --- |
| Property | `transform`, or `all` (prefer naming the property) |
| Duration | `200ms` |
| Timing | `ease`, `linear`, `ease-in-out` |
| Delay | A fourth time value |

`@keyframes` plus `animation` repeats a set of keyframes. A transition runs once between two states. An animation can loop. Respect `prefers-reduced-motion`.

## 14. Variables and media queries

```css
:root {
  --space: 1rem;
  --ink: #1c1c1c;
  --panel: #fff;
}
.card {
  color: var(--ink);
  padding: var(--space);
  background: var(--panel, #fff);
}
@media (max-width: 40rem) {
  .stats { grid-template-columns: 1fr; }
}
```

| Idea | Detail |
| --- | --- |
| Custom property | `--name`. Inherited like color |
| `var(--name, fallback)` | The fallback is used when `--name` is not set |
| `:root` | The `html` element, with higher specificity. The usual place to define tokens |
| `min-width` | Styles apply at this width and wider. Mobile-first |
| `max-width` | Styles apply at this width and narrower |
| `prefers-color-scheme: dark` | The reader asked for a dark UI |
| `prefers-reduced-motion` | The reader asked for less motion |

A media query does not replace flex wrap. Start with a layout that works in one column, then add columns when there is room.

## 15. A layout you can rebuild

Attach this to the HTML practice page. Predict the narrow-screen result before you resize.

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

| Question | Answer |
| --- | --- |
| Why `box-sizing: border-box` first? | Width then includes padding and border |
| Why is the grid inside `min-width`? | The page is one column until 48rem, then two |
| Which selector wins, `a` or `nav a`? | `nav a`, because it has one element plus another element |
