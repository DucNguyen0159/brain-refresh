# Tailwind CSS

Course 12 of 15. Previous: [11-firebase-firestore.md](11-firebase-firestore.md). Next: [13-tanstack-query.md](13-tanstack-query.md).

Tailwind is a set of CSS classes. Each class sets one or a few properties. You put the classes on `className`. The CSS course is the meaning of those properties. This course is the class names and the variants.

The build step scans your files and keeps only the classes it finds. A class name assembled only by string concatenation may be missing from the output. Write the full class in the source.

## Contents

1. [How a class is built](#1-how-a-class-is-built)
2. [Spacing, sizing, and the box](#2-spacing-sizing-and-the-box)
3. [Typography and color](#3-typography-and-color)
4. [Layout](#4-layout)
5. [Borders, effects, and display](#5-borders-effects-and-display)
6. [Variants](#6-variants)
7. [A small component](#7-a-small-component)

## 1. How a class is built

| Piece | Example | CSS idea |
| --- | --- | --- |
| Utility | `p-4` | `padding` |
| Scale | `4` | A step on the spacing scale, not 4 pixels. `4` is `1rem` |
| Arbitrary value | `p-[13px]` | An exact value when the scale is wrong |
| Variant | `md:p-4` | Apply `p-4` from the `md` breakpoint up |
| State | `hover:underline` | Apply while hovering |
| Important | `p-4!` in v4, or `!p-4` in v3 | Last resort |
| Stack | `class="flex gap-2 p-4"` | All of them apply |

Default breakpoints:

| Prefix | Min width |
| --- | --- |
| (none) | All widths |
| `sm:` | 40rem |
| `md:` | 48rem |
| `lg:` | 64rem |
| `xl:` | 80rem |
| `2xl:` | 96rem |

Unprefixed utilities apply everywhere. A prefixed one overrides them from that width up. `p-2 md:p-6` is small padding, then larger padding from `md` up.

## 2. Spacing, sizing, and the box

The spacing scale is shared by padding, margin, gap, width, height, and several position offsets. The same number means the same distance on each of them.

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

`p-4`, `m-4`, `gap-4`, and `w-16` all read from this scale. `w-16` is the spacing token `16`, which is `4rem`, not 16 pixels.

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

`space-x-4` and `space-y-4` put margin between children. `gap-4` is the flex and grid version and is the one to prefer.

## 3. Typography and color

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

Colors use a palette name and a step. `500` is the middle. Higher numbers are darker in the default palette.

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

`text-inherit` and `bg-transparent` reset a color.

## 4. Layout

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

Flex and grid classes for the same job:

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

A centered row with space between the ends:

```html
<div class="flex items-center justify-between gap-4">
  <p>Logo</p>
  <nav class="flex gap-4">
    <a href="/search">Search</a>
  </nav>
</div>
```

A responsive grid:

```html
<ul class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
  <li>One</li>
  <li>Two</li>
</ul>
```

## 5. Borders, effects, and display

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

`hidden` is `display: none`. `invisible` keeps the gap. `sr-only` is for a label you want announced and not shown.

## 6. Variants

Variants stack: `md:hover:bg-slate-100`.

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

```html
<button class="rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 disabled:opacity-50" type="button">
  Save
</button>
```

`@apply` copies utilities into a CSS rule. It is an escape hatch. The normal style is to put the utilities in the markup.

## 7. A small component

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

| Class in that card | CSS property it stands for |
| --- | --- |
| `max-w-sm` | `max-width` |
| `rounded-lg` | `border-radius` |
| `aspect-video` | `aspect-ratio` |
| `object-cover` | `object-fit` |
| `flex flex-col gap-2` | Column flex with a gap |
| `p-4` | Padding |
| `text-slate-600` | A gray text color |

## Drill

1. Center a row of two links with a gap.
2. Make a grid that is one column on a phone and three columns from `lg` up.
3. Name the CSS property behind `p-4`, `hidden`, and `items-center`.
