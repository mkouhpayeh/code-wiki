---
tags:
  - Markdown
  - Tips
  - Documentation
---

## Code Annotation Examples

### Highlight inner text
`
Simply add **``** at the beginning and end of the text.
`

### Content Tabs
`
Just add **=== "TabName"**
`

### Unordered List
`
Just add "* Text"
`

### Ordered List
`
Just add "1. Text"
`

### Codeblocks

#### Plain codeblock
`
Simply add __```__ at the beginning and end of the code block.
`

#### Codeblock for a specific language
`
Write the name of the code language after **```** like **py**, **cshapr**, **shell**, ...
`

#### Codeblock with a title
`
Write **title="bubble_sort.py"**
`

#### With line numbers

``` py linenums="1"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

#### Highlighting lines

``` py hl_lines="2 3"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

## Icons and Emojs

:smile: 

:fontawesome-regular-face-laugh-wink:

:fontawesome-brands-twitter:{ .twitter }

:octicons-heart-fill-24:{ .heart }
