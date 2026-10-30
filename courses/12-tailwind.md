# Tailwind CSS

Course 12 of 15. Previous: [11-firebase-firestore.md](11-firebase-firestore.md). Next: [13-tanstack-query.md](13-tanstack-query.md).

Tailwind is a set of CSS classes. Each class sets one or a few properties. You put the classes on `className`. The CSS course is the meaning of those properties. This course is the class names and the variants.

The build step scans your files and keeps only the classes it finds. A class name assembled only by string concatenation may be missing from the output. Write the full class in the source.

Every numbered section has the same parts: the concept, the terms, the properties, several cases with the result, common mistakes, a tip, and a practice with a solution.

## Contents

1. [How a class is built](#1-how-a-class-is-built)
2. [Spacing, sizing, and the box](#2-spacing-sizing-and-the-box)
3. [Typography and color](#3-typography-and-color)
4. [Layout](#4-layout)
5. [Borders, effects, and display](#5-borders-effects-and-display)
6. [Variants](#6-variants)
7. [A small component](#7-a-small-component)

## 1. How a class is built

### Concept

A utility is one class that sets one or a few CSS properties. The number in the class is a step on a scale, not a raw pixel count, unless you write an arbitrary value. A variant prefixes the utility and decides when it applies. Unprefixed utilities apply everywhere. A prefixed one overrides them from that width up. `p-2 md:p-6` is small padding, then larger padding from `md` up. The build keeps a class only when the full name appears in the source.

### Terms

| Term | Meaning |
| --- | --- |
| Utility | One class, such as `p-4`, that sets a CSS property |
| Scale | The step inside the name. `4` is a spacing step, not 4 pixels |
| Arbitrary value | An exact value in square brackets when the scale is wrong |
| Variant | A prefix such as `md:` or `hover:` that limits when the utility applies |
| Breakpoint | A minimum width. Styles from that prefix apply at that width and above |
| Stack | Several classes on one element. All of them apply |

### Properties

| Piece | Example | CSS idea |
| --- | --- | --- |
| Utility | `p-4` | `padding` |
| Scale | `4` | A step on the spacing scale, not 4 pixels. `4` is `1rem` |
| Arbitrary value | `p-[13px]` | An exact value when the scale is wrong |
| Variant | `md:p-4` | Apply `p-4` from the `md` breakpoint up |
| State | `hover:underline` | Apply while hovering |
| Important | `p-4!` in v4, or `!p-4` in v3 | Last resort |
| Stack | `class="flex gap-2 p-4"` | All of them apply |

| Prefix | Min width |
| --- | --- |
| (none) | All widths |
| `sm:` | 40rem |
| `md:` | 48rem |
| `lg:` | 64rem |
| `xl:` | 80rem |
| `2xl:` | 96rem |

### Examples

**Case: one utility sets one property**

```html
<p class="p-4">Irvine</p>
```

Result: the class sets padding on all sides to `1rem`.
Why: `p` is padding and `4` is the spacing step. That step is `1rem`, which is 16px when the root font size is 16px.

**Case: an arbitrary value skips the scale**

```html
<p class="p-[13px]">Irvine</p>
```

Result: the class sets padding to exactly 13px.
Why: the brackets hold a raw CSS value. Use them when no scale step matches.

**Case: a breakpoint overrides from that width up**

```html
<p class="p-2 md:p-6">Irvine</p>
```

Result: below `md` the padding is `0.5rem`. From `48rem` wide and up, the padding is `1.5rem`.
Why: the unprefixed class applies at every width. `md:p-6` overrides it from the `md` breakpoint up. `md` is `48rem`.

**Case: a joined string is missing from the build**

```html
<p class="p-4 text-slate-900">Irvine</p>
```

```javascript
const size = "p-" + "4";
```

Result: `p-4` in the markup is in the built CSS. A class that exists only as `"p-" + "4"` can be absent from the output, so the element has no padding from that string.
Why: the build scans source text for complete class names. Write the full class in the source.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Reading `p-4` as 4 pixels | The padding is `1rem`, about 16px | Treat the number as a scale step |
| Building a class with string concatenation | The scanner never sees `p-4`, so the CSS is missing | Write the full class in the source |
| Expecting `md:p-6` to apply only at exactly 48rem | It applies from `md` up, including `lg` and wider | Add a larger prefix only when that width needs a new value |
| Reaching for `p-4!` on the first conflict | The important flag fights the rest of the stylesheet | Put the winning utility later, or use a variant |

### Tip

If a class you can see in the file has no effect, check that the full name is written out and that a wider variant is not overriding it.

### Practice

Write a paragraph that has `0.5rem` of padding on a phone and `1.5rem` of padding from the `md` breakpoint up.

### Solution

```html
<p class="p-2 md:p-6">Irvine</p>
```

Result: padding is `0.5rem` below `48rem`, and `1.5rem` from `md` up.
Why: `2` is `0.5rem` and `6` is `1.5rem`. The `md:` prefix starts at `48rem`.

## 2. Spacing, sizing, and the box

### Concept

The spacing scale is shared by padding, margin, gap, width, height, and several position offsets. The same number means the same distance on each of them. `p-4`, `m-4`, `gap-4`, and `w-16` all read from this scale. `w-16` is the spacing token `16`, which is `4rem`, not 16 pixels. `space-x-4` and `space-y-4` put margin between children. `gap-4` is the flex and grid version and is the one to prefer.

### Terms

| Term | Meaning |
| --- | --- |
| Spacing token | A number, or `px`, that maps to one length |
| `p` / `m` | Padding or margin on all sides |
| `x` / `y` | The horizontal pair or the vertical pair |
| `t` `r` `b` `l` | Top, right, bottom, left |
| `s` / `e` | Inline start and end |
| Negative margin | A leading `-`, as in `-mt-4` |

### Properties

| Token | Length | About this many pixels when the root is 16px |
| --- | --- | --- |
| `0` | `0` | 0 |
| `px` | `1px` | 1 |
| `0.5` | `0.125rem` | 2 |
| `1` | `0.25rem` | 4 |
| `2` | `0.5rem` | 8 |
| `4` | `1rem` | 16 |
| `8` | `2rem` | 32 |
| `16` | `4rem` | 64 |
| `64` | `16rem` | 256 |

| Class | Property |
| --- | --- |
| `p-4` | Padding on all sides |
| `px-4` | Left and right |
| `py-4` | Top and bottom |
| `pt-4` `pr-4` `pb-4` `pl-4` | One side |
| `ps-4` `pe-4` | Padding-inline start and end |
| `m-4` `mx-auto` `mt-8` | Margin. `mx-auto` centers a block that has a width |
| `-mt-4` | Negative margin |
| `w-full` | Width 100% |
| `w-1/2` | Width 50% |
| `w-64` | A spacing-scale width |
| `max-w-xl` | A named max width, `36rem` |
| `max-w-prose` | A reading measure |
| `h-screen` | `100vh` |
| `min-h-screen` | At least the viewport |
| `size-10` | Width and height together |

### Examples

**Case: the same step on padding, margin, and gap**

```html
<div class="m-4 flex gap-4 p-4">
  <p>One</p>
  <p>Two</p>
</div>
```

Result: the box has `1rem` of margin, `1rem` of padding, and `1rem` between the children.
Why: `4` is `1rem` on margin, padding, and gap. The scale is shared.

**Case: `w-16` is not 16 pixels**

```html
<div class="w-16">Irvine</div>
```

Result: the class sets the width to `4rem`, about 64px when the root is 16px.
Why: `16` is the spacing token `16`. Token `4` is the one that is `1rem`.

**Case: axes and one side**

```html
<div class="px-4 pt-8">
  <p class="mx-auto w-64">Irvine</p>
</div>
```

Result: the outer box has horizontal padding of `1rem` and top padding of `2rem`. The paragraph is `16rem` wide and horizontally centered.
Why: `px` is left and right, `pt` is top, `w-64` is token `64` (`16rem`), and `mx-auto` centers a block that has a width.

**Case: gap instead of space between children**

```html
<div class="flex gap-4">
  <p>One</p>
  <p>Two</p>
</div>
```

Result: the flex row puts `1rem` of gap between the children.
Why: `gap-4` is the flex and grid way to separate children. `space-x-4` would put margin between them instead.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Reading `w-16` as 16px | The box is `4rem` wide | Use `w-4` for `1rem`, or `w-[16px]` for 16 pixels |
| `mx-auto` on an element with no width | There is no free space to split, so it does not look centered | Set a width, such as `w-64` or `max-w-xl` |
| Using `space-x-4` inside a flex row | Margins fight `gap` and wrapping | Prefer `gap-4` |
| Setting height with `h-screen` when you wanted a minimum | The box is locked to the viewport | `min-h-screen` is at least the viewport |

### Tip

When two spacing classes disagree, look at the suffix. `p-4` and `px-8` can both apply: the axis class overrides that pair of sides.

### Practice

Make a block that is half the parent wide, centered, with `1rem` of padding and a top margin of `2rem`.

### Solution

```html
<div class="mx-auto mt-8 w-1/2 p-4">Irvine</div>
```

Result: the width is 50%, the auto horizontal margins center it, the top margin is `2rem`, and the padding is `1rem`.

## 3. Typography and color

### Concept

Type utilities set size, family, weight, line height, letter spacing, alignment, decoration, and transform. `truncate` is one line with hidden overflow and an ellipsis. Colors use a palette name and a step. `500` is the middle. Higher numbers are darker in the default palette. The slash form sets alpha: `bg-black/50` is black at 50% alpha. `text-inherit` and `bg-transparent` reset a color.

### Terms

| Term | Meaning |
| --- | --- |
| Palette | A color family name, such as `slate` or `blue` |
| Step | The number after the name. `500` is the middle. Higher numbers are darker in the default palette |
| Alpha | The slash value. `bg-black/50` is 50% opacity on that color |
| Truncate | One line, hidden overflow, ellipsis |
| Reset | `text-inherit` or `bg-transparent` puts the color back |

### Properties

| Class | Effect |
| --- | --- |
| `text-sm` `text-base` `text-lg` `text-xl` `text-2xl` | Font size |
| `font-sans` `font-serif` `font-mono` | Family |
| `font-normal` `font-medium` `font-semibold` `font-bold` | Weight |
| `italic` | Italic |
| `leading-tight` `leading-normal` `leading-relaxed` | Line height |
| `tracking-wide` | Letter spacing |
| `text-left` `text-center` `text-right` | Alignment |
| `underline` `no-underline` | Decoration |
| `uppercase` `lowercase` `capitalize` | Transform |
| `truncate` | One line, hidden overflow, ellipsis |
| `whitespace-nowrap` | Do not wrap |
| `text-balance` | Balance wrapping on headings |

| Class | Paints |
| --- | --- |
| `text-slate-900` | Text |
| `text-slate-500` | Muted text |
| `bg-white` `bg-slate-100` | Background |
| `bg-blue-600` | A solid action color |
| `text-white` | Text on that action |
| `decoration-blue-600` | Underline color |
| `accent-blue-600` | Form control accent |
| `opacity-50` | The whole element |
| `bg-black/50` | Black at 50% alpha. The slash is the alpha |

### Examples

**Case: size, weight, and a muted line**

```html
<h2 class="text-lg font-semibold text-slate-900">Irvine</h2>
<p class="text-sm text-slate-500">3 beds</p>
```

Result: the heading is a large semibold dark slate, and the line under it is small and muted.
Why: `text-lg` and `text-sm` set size. `text-slate-900` is the darker step. `text-slate-500` is the middle, used here as muted text.

**Case: one line with an ellipsis**

```html
<p class="w-32 truncate">A long address that does not fit</p>
```

Result: the class keeps a single line, hides the overflow, and shows an ellipsis.
Why: `truncate` is that trio. `whitespace-nowrap` alone would refuse to wrap and would not add the ellipsis.

**Case: an action color and a transparent scrim**

```html
<button class="bg-blue-600 text-white" type="button">Save</button>
<div class="bg-black/50">Shade</div>
```

Result: the button has a solid blue background and white text. The div paints black at 50% alpha.
Why: `600` is darker than the middle step `500`. The slash is the alpha. `opacity-50` would fade the whole element, children included.

**Case: reset a color**

```html
<p class="bg-transparent text-inherit">Irvine</p>
```

Result: the text color comes from the parent, and the background is transparent.
Why: `text-inherit` and `bg-transparent` reset a color that an earlier class or a parent rule had set.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Using `text-slate-500` for text on a blue button | The label looks muted on the action | `text-white` on `bg-blue-600` |
| Writing `bg-black-50` for a scrim | That class is not 50% alpha | `bg-black/50` |
| Using `opacity-50` when only the background should fade | Text and children fade too | Put the alpha on the color: `bg-black/50` |
| Expecting `truncate` to wrap to two lines | It stays one line | It is one line, hidden overflow, and an ellipsis |

### Tip

Pick one palette for text (`slate`) and one for actions (`blue`) so the steps stay comparable. `900` and `600` are darker than `500`.

### Practice

Style a heading as large, semibold, and `slate-900`, with a small muted subtitle that will ellipsize.

### Solution

```html
<h2 class="text-lg font-semibold text-slate-900">Irvine</h2>
<p class="truncate text-sm text-slate-500">3 beds · a long public note</p>
```

Result: the heading is large, semibold, and dark. The subtitle is small, muted, and one line with an ellipsis when it overflows.

## 4. Layout

### Concept

`flex` and `grid` turn a layout on. Flex direction is a row or a column. Grid direction is a count of columns or rows. `justify-*` packs the main axis. `items-*` aligns the cross axis. `flex-1` lets one child grow and shrink. `col-span-2` lets one grid item cover two columns. `order-*` changes visual order and leaves DOM order unchanged. A responsive grid is one set of columns, then another from a breakpoint: `grid-cols-1 md:grid-cols-3`.

### Terms

| Term | Meaning |
| --- | --- |
| Main axis | The direction flex lays children out. Row is horizontal. Column is vertical |
| Cross axis | The other direction. `items-*` aligns it |
| Track | One grid column or row |
| Span | How many tracks an item covers |
| DOM order | Source order. `order-1` does not change it |

### Properties

| Class | CSS |
| --- | --- |
| `block` `inline` `inline-block` `flex` `inline-flex` `grid` `hidden` | `display` |
| `flex-row` `flex-col` `flex-row-reverse` | `flex-direction` |
| `flex-wrap` | Wrap |
| `flex-1` | Grow and shrink |
| `flex-none` | Do not grow or shrink |
| `grow` `shrink-0` | One axis of flexibility |
| `justify-start` `justify-center` `justify-end` `justify-between` | Main axis |
| `items-start` `items-center` `items-end` `items-stretch` `items-baseline` | Cross axis |
| `self-center` | One item's cross alignment |
| `gap-4` `gap-x-4` `gap-y-2` | Gap |
| `grid-cols-3` | Three equal columns |
| `grid-cols-1 md:grid-cols-3` | One column, then three |
| `col-span-2` | An item covers two columns |
| `col-span-full` | An item covers every column |
| `grid-rows-2` | Two rows |
| `order-1` `order-2` | Visual order. DOM order is unchanged |
| `basis-1/2` | Flex basis |

| Job | Flex | Grid |
| --- | --- | --- |
| Turn the layout on | `flex` or `inline-flex` | `grid` or `inline-grid` |
| Direction | `flex-row`, `flex-col` | `grid-cols-3`, `grid-rows-2` |
| Space between children | `gap-4`, `gap-x-4`, `gap-y-2` | The same `gap-*` classes |
| Pack the line | `justify-between`, `justify-center` | `justify-items-center` for items, `justify-between` for the grid as a whole |
| Cross alignment | `items-center` | `items-center` |
| One child takes space | `flex-1`, `basis-1/2` | `col-span-2`, `row-span-2` |
| Wrap onto another line | `flex-wrap` | Extra items move to the next implicit row |
| Change at a breakpoint | `md:flex-row` | `md:grid-cols-3` |

### Examples

**Case: a centered row with space between the ends**

```html
<div class="flex items-center justify-between gap-4">
  <p>Logo</p>
  <nav class="flex gap-4">
    <a href="/search">Search</a>
  </nav>
</div>
```

Result: the outer class makes a row, centers the children on the cross axis, and pushes the logo and the nav to opposite ends with a `1rem` gap.
Why: `justify-between` packs the main axis. `items-center` aligns the cross axis. The nav is its own row with the same gap.

**Case: two links centered as a group**

```html
<div class="flex items-center justify-center gap-4">
  <a href="/search">Search</a>
  <a href="/saved">Saved</a>
</div>
```

Result: the links sit in a row, in the center of the parent, with `1rem` between them.
Why: `justify-center` packs the main axis to the center. `items-center` centers them on the cross axis. `gap-4` is the space between.

**Case: one column, then two, then three**

```html
<ul class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
  <li>One</li>
  <li>Two</li>
</ul>
```

Result: the list is one column by default, two columns from `sm` (`40rem`) up, and three columns from `lg` (`64rem`) up, with `1rem` of gap.
Why: each `grid-cols-*` replaces the column count from that breakpoint up. Extra items move to the next implicit row.

**Case: one item covers two columns**

```html
<div class="grid grid-cols-3 gap-4">
  <p class="col-span-2">Wide</p>
  <p>Side</p>
  <p class="col-span-full">Footer</p>
</div>
```

Result: the first item covers two of the three columns, the second covers one, and the footer covers every column.
Why: `col-span-2` and `col-span-full` set how many tracks an item covers. DOM order is still the source order.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `justify-center` when you wanted the cross axis | A row centers sideways and stays top-aligned | `items-center` for the cross axis |
| `grid-cols-3` with no breakpoint | Three columns even on a narrow screen | `grid-cols-1 lg:grid-cols-3` |
| `order-1` to fix heading order for assistive tech | Visual order changes. DOM order does not | Put the source in the order it should be read |
| `flex-1` on a grid item | The item does not span tracks | `col-span-2` or `row-span-2` |

### Tip

Say the job first, then pick the column in the flex-or-grid table. Gap classes are the same on both.

### Practice

Center a row of two links with a gap. Then make a grid that is one column on a phone and three columns from `lg` up.

### Solution

```html
<nav class="flex items-center justify-center gap-4">
  <a href="/search">Search</a>
  <a href="/saved">Saved</a>
</nav>
<ul class="grid grid-cols-1 gap-4 lg:grid-cols-3">
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
</ul>
```

Result: the links form a centered row with `1rem` between them. The list is one column until `lg`, then three.

## 5. Borders, effects, and display

### Concept

Border, radius, shadow, and ring utilities draw the edge of a box. `overflow-hidden` clips it. `object-cover` decides how an image fills its box. Position utilities (`relative`, `absolute`, `fixed`, `sticky`) plus offsets place it. `hidden` is `display: none`. `invisible` keeps the gap. `sr-only` is for a label you want announced and not shown.

### Terms

| Term | Meaning |
| --- | --- |
| `hidden` | `display: none`. The element takes no gap |
| `invisible` | The element is not painted and still occupies its gap |
| `sr-only` | Visually hidden, still read by a screen reader |
| Ring | An outline-like line from `ring-*` |
| Inset | `inset-0` sets all four offsets to zero |

### Properties

| Class | Effect |
| --- | --- |
| `border` | A 1px border |
| `border-2` | A 2px border |
| `border-slate-200` | Border color |
| `border-t` | Top only |
| `rounded` `rounded-md` `rounded-lg` `rounded-full` | Radius. `full` is a pill or circle |
| `shadow` `shadow-md` `shadow-lg` | Box shadow |
| `ring-2` `ring-blue-500` | An outline-like ring |
| `overflow-hidden` `overflow-auto` | Overflow |
| `object-cover` `object-contain` | How an image fills its box |
| `aspect-video` | 16:9 box |
| `cursor-pointer` | Pointer cursor |
| `select-none` | Text cannot be selected |
| `sr-only` | Visually hidden, still read by a screen reader |
| `absolute` `relative` `fixed` `sticky` | Position |
| `inset-0` | All four offsets zero |
| `top-0` `right-4` | Offsets |
| `z-10` | Stacking |
| `transition` | A default transition |
| `duration-200` | 200ms |
| `ease-out` | Timing |

### Examples

**Case: a bordered card edge**

```html
<article class="overflow-hidden rounded-lg border border-slate-200 shadow-sm">
  Photo
</article>
```

Result: the class draws a 1px slate border, a large radius, a small shadow, and clips anything that sticks out of the radius.
Why: `border` is the 1px width, `border-slate-200` is the color, `rounded-lg` is the radius, and `overflow-hidden` clips the children.

**Case: an image fills a 16:9 box**

```html
<img class="aspect-video w-full object-cover" src="house.jpg" alt="Front of a one-story house">
```

Result: the image box is 16:9 and full width. The picture covers that box and the extra is cropped.
Why: `aspect-video` sets the ratio. `object-cover` fills the box. `object-contain` would letterbox instead.

**Case: `hidden`, `invisible`, and `sr-only`**

```html
<p class="hidden">Gone</p>
<p class="invisible">Gap</p>
<p class="sr-only">Save this home</p>
```

Result: the first paragraph takes no space. The second is invisible and still occupies its line. The third is not shown and is still read by a screen reader.
Why: `hidden` is `display: none`. `invisible` keeps the gap. `sr-only` is the visually hidden label.

**Case: a badge pinned to the corner**

```html
<div class="relative">
  <img class="aspect-video w-full object-cover" src="house.jpg" alt="Front of a one-story house">
  <span class="absolute right-4 top-0 rounded-full bg-white px-2">New</span>
</div>
```

Result: the span sits at the top of the image, `1rem` in from the right, above the image in the stack only if a `z-*` class is added later.
Why: `relative` is the anchor. `absolute` with `top-0` and `right-4` sets the offsets. `inset-0` would stretch the span to all four edges.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `invisible` to remove a block from the layout | The gap remains | `hidden` |
| `hidden` on a label the screen reader must hear | The name is gone for everyone | `sr-only` |
| `object-cover` without a sized box | The image has no frame to crop into | `aspect-video` or a height, plus `w-full` |
| `absolute` with no positioned ancestor | The offsets use a farther ancestor, or the page | Put `relative` on the box that should own the offsets |

### Tip

Use `overflow-hidden` on the same element as `rounded-lg` when a child, such as an image, would paint square corners over the radius.

### Practice

Build a 16:9 image that covers its box, inside a relative frame, with a pill label at the top right.

### Solution

```html
<div class="relative overflow-hidden rounded-lg">
  <img class="aspect-video w-full object-cover" src="house.jpg" alt="Front of a one-story house">
  <span class="absolute right-4 top-0 rounded-full bg-white px-2">New</span>
</div>
```

Result: the image fills a 16:9 frame and is cropped to the radius. The label sits at the top, `1rem` from the right.

## 6. Variants

### Concept

A variant is a prefix. Variants stack: `md:hover:bg-slate-100` applies that background when the pointer is over the element and the viewport is at least `md`. `focus-visible:` is the keyboard-focus variant. Prefer it for rings. `dark:` applies when the page is in dark mode, after you enable the dark variant. `before:` and `after:` need `content-['']` or another content class. `group-hover:` watches a parent with `class="group"`. `peer-invalid:` watches a sibling with `class="peer"`. `@apply` copies utilities into a CSS rule. It is an escape hatch. The normal style is to put the utilities in the markup.

### Terms

| Term | Meaning |
| --- | --- |
| State variant | `hover:`, `focus:`, `focus-visible:`, `active:`, `disabled:` |
| Structural variant | `first:`, `last:`, `odd:`, `even:` |
| Group | A parent marked `group`. Children use `group-hover:` |
| Peer | A sibling marked `peer`. Later siblings use `peer-invalid:` |
| `@apply` | Copies utilities into a CSS rule. An escape hatch |

### Properties

| Variant | Applies when |
| --- | --- |
| `hover:` | Pointer over the element |
| `focus:` | The element is focused |
| `focus-visible:` | Keyboard focus. Prefer this for rings |
| `active:` | Being pressed |
| `disabled:` | The control is disabled |
| `first:` `last:` `odd:` `even:` | Position among siblings |
| `dark:` | The page is in dark mode, after you enable the dark variant |
| `placeholder:` | Placeholder text |
| `before:` `after:` | Pseudo-elements. They need `content-['']` or another content class |
| `motion-reduce:` | The reader prefers less motion |
| `aria-expanded:` | An ARIA state |
| `group-hover:` | A parent with `class="group"` is hovered |
| `peer-invalid:` | A sibling with `class="peer"` is invalid |

### Examples

**Case: hover, keyboard focus, and disabled**

```html
<button class="rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 disabled:opacity-50" type="button">
  Save
</button>
```

Result: the button is blue, darker on hover, shows an offset outline for keyboard focus, and fades when disabled.
Why: each prefix is a separate condition. `focus-visible:` is the one to prefer for rings, because it follows keyboard focus.

**Case: stacked variants**

```html
<div class="bg-white md:hover:bg-slate-100">Row</div>
```

Result: from `md` up, hovering paints `slate-100`. Below `md`, hover does not apply that background.
Why: variants stack. `md:hover:bg-slate-100` needs both the breakpoint and the hover.

**Case: group and peer**

```html
<div class="group">
  <p class="group-hover:underline">Irvine</p>
</div>
<input class="peer" type="email">
<p class="peer-invalid:text-red-600">Enter an email</p>
```

Result: the paragraph underlines when the parent is hovered. The message turns red when the email input is invalid.
Why: `group-hover:` requires `class="group"` on the parent. `peer-invalid:` requires `class="peer"` on the earlier sibling.

**Case: `@apply` is the escape hatch**

```css
.btn {
  @apply rounded bg-blue-600 px-4 py-2 text-white;
}
```

```html
<button class="btn" type="button">Save</button>
```

Result: the button receives those utilities through the CSS rule.
Why: `@apply` copies utilities into a rule. The normal style is to put `rounded bg-blue-600 px-4 py-2 text-white` on the element itself.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `hover:md:bg-slate-100` when you meant "hover, from md up" | The variant order is easy to misread | Stack the width first: `md:hover:bg-slate-100` |
| `focus:` for a mouse-and-keyboard ring | The ring appears on click as well | `focus-visible:` |
| `group-hover:` with no `group` parent | The child never sees the hover | Put `class="group"` on the parent |
| `before:block` with no content class | The pseudo-element is empty and does not show | Add `content-['']` or another content class |
| Reaching for `@apply` for one button | A class in CSS hides the utilities from the markup | Put the utilities on `class` |

### Tip

Read a stacked class from the left. `md:hover:bg-slate-100` means "at md, when hovered, this background."

### Practice

Write a Save button that darkens on hover, shows a keyboard outline, and fades when disabled. Do it without `@apply`.

### Solution

```html
<button class="rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 disabled:opacity-50" type="button">
  Save
</button>
```

Result: hover darkens the button, keyboard focus draws the outline, and the disabled state sets half opacity. The utilities sit on the element.

## 7. A small component

### Concept

A component is a stack of utilities you already know: a max width, a border, a radius, a shadow, an image frame, and a column of type. Each class still sets one property. The card below is the whole pattern in one place.

### Terms

| Piece of the card | Job |
| --- | --- |
| Frame | `max-w-sm`, border, radius, background, shadow, overflow |
| Media | `aspect-video`, `w-full`, `object-cover` |
| Body | `flex flex-col gap-2 p-4` |
| Type | Size, weight, and slate colors |

### Properties

| Class in that card | CSS property it stands for |
| --- | --- |
| `max-w-sm` | `max-width` |
| `overflow-hidden` | `overflow` |
| `rounded-lg` | `border-radius` |
| `border` | `border-width` |
| `border-slate-200` | `border-color` |
| `bg-white` | `background-color` |
| `shadow-sm` | `box-shadow` |
| `aspect-video` | `aspect-ratio` |
| `object-cover` | `object-fit` |
| `w-full` | `width` |
| `flex flex-col gap-2` | Column flex with a gap |
| `p-4` | Padding |
| `text-lg` | `font-size` |
| `font-semibold` | `font-weight` |
| `text-slate-900` | A dark slate text color |
| `text-sm` | A smaller `font-size` |
| `text-slate-600` | A gray text color |
| `text-base font-medium` | Body size and a medium weight |

### Examples

**Case: the card**

```html
<article class="max-w-sm overflow-hidden rounded-lg border border-slate-200 bg-white shadow-sm">
  <img class="aspect-video w-full object-cover" src="house.jpg" alt="Front of a one-story house">
  <div class="flex flex-col gap-2 p-4">
    <h2 class="text-lg font-semibold text-slate-900">Irvine</h2>
    <p class="text-sm text-slate-600">3 beds · 2 baths</p>
    <p class="text-base font-medium">$999,000</p>
  </div>
</article>
```

Result: the article is at most the `max-w-sm` width, with a light border, a large radius, a white background, and a small shadow. The image covers a 16:9 frame. The text stacks in a column with `0.5rem` of gap and `1rem` of padding.
Why: every class sets the property in the table. `gap-2` is the `0.5rem` step. `p-4` is the `1rem` step.

**Case: the image box without `object-cover`**

```html
<img class="aspect-video w-full" src="house.jpg" alt="Front of a one-story house">
```

Result: the box is still 16:9 and full width. The image uses its default fit, so it may not crop to the frame.
Why: `aspect-video` sets `aspect-ratio`. `object-cover` is the class that sets `object-fit: cover`.

**Case: the body column**

```html
<div class="flex flex-col gap-2 p-4">
  <h2 class="text-lg font-semibold text-slate-900">Irvine</h2>
  <p class="text-sm text-slate-600">3 beds · 2 baths</p>
</div>
```

Result: the heading and the line stack vertically, with `0.5rem` between them and `1rem` of padding around the stack. The heading is large, semibold, and dark. The line is small and gray.
Why: `flex-col` is the column direction. `text-slate-600` is the gray. `text-slate-900` is the darker heading color.

**Case: the price line**

```html
<p class="text-base font-medium">$999,000</p>
```

Result: the price uses the body font size and a medium weight.
Why: `text-base` is `font-size`. `font-medium` is `font-weight`. No color class means the price keeps the inherited text color.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Dropping `overflow-hidden` on a rounded card | The image paints over the rounded corners | Keep `overflow-hidden` on the article |
| `max-w-sm` with no parent constraint and a huge image | The card stops growing, and the image still needs `w-full` inside it | `w-full object-cover` on the image |
| `gap-2` on a block that is not flex or grid | The gap does nothing | Add `flex flex-col` or `grid` |
| Putting the price color on `text-slate-600` by habit | The price looks like the muted line | Leave it `text-base font-medium`, or set a darker step on purpose |

### Tip

Read the card from the outside in: frame, media, body, type. If one layer looks wrong, the class for that layer is the one to change.

### Practice

Add a fourth line to the card body that says the city is in California, small and gray, without changing the price styles.

### Solution

```html
<article class="max-w-sm overflow-hidden rounded-lg border border-slate-200 bg-white shadow-sm">
  <img class="aspect-video w-full object-cover" src="house.jpg" alt="Front of a one-story house">
  <div class="flex flex-col gap-2 p-4">
    <h2 class="text-lg font-semibold text-slate-900">Irvine</h2>
    <p class="text-sm text-slate-600">3 beds · 2 baths</p>
    <p class="text-base font-medium">$999,000</p>
    <p class="text-sm text-slate-600">California</p>
  </div>
</article>
```

Result: the new line uses the same small gray type as the beds line. The price stays body size and medium weight.

## Drill

1. Center a row of two links with a gap.
2. Make a grid that is one column on a phone and three columns from `lg` up.
3. Name the CSS property behind `p-4`, `hidden`, and `items-center`.

### Solution 1

```html
<nav class="flex items-center justify-center gap-4">
  <a href="/search">Search</a>
  <a href="/saved">Saved</a>
</nav>
```

Result: the two links sit in a centered row with a `1rem` gap.
Why: `flex` turns the row on, `justify-center` centers the main axis, `items-center` centers the cross axis, and `gap-4` is the gap.

### Solution 2

```html
<ul class="grid grid-cols-1 gap-4 lg:grid-cols-3">
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
</ul>
```

Result: one column at every width, then three columns from `lg` (`64rem`) up.
Why: the unprefixed `grid-cols-1` is the phone. `lg:grid-cols-3` overrides it from that breakpoint up.

### Solution 3

| Class | CSS property |
| --- | --- |
| `p-4` | `padding` |
| `hidden` | `display` (`display: none`) |
| `items-center` | `align-items` |
