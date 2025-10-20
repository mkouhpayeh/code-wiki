---
tags:
  - Markdown
  - Tips
  - Documentation
---

You can find complete [MKDocs Material Documentation](https://squidfunk.github.io/mkdocs-material/reference/) here.

## Text Annotations

### Tables
| Name    | Role       | Years |
|---------|------------|------:|
| Mahi     | Developer |     10|
| Ali      | Designer  |     11|

```
| Name    | Role       | Years |
|---------|------------|------:|
| Mahi     | Developer |     10|
| Ali      | Designer  |     11|

```

### Nesting/Indentation
- Put a blank line before the list block.

- Use consistent spaces (avoid tabs) and indent nested list items by 4 spaces (safe and compatible).

### Highlight inner text
Simply write text between ` `` `


### Content Tabs
Just add ` === "TabName" `


### Unordered List
Just write ` * Text `


### Ordered List
Just write ` 1. Text `

### Admonition/callout with title
write ` !!! note/info/abstract/tip/question/success/ warning/failure "title" ` with an **indent** with **four spaces** 

for Collapsible just write ` ??? ` instead of ` !!! ` 

### Codeblocks

#### Plain codeblock
Simply write code block between ` ``` `

#### Codeblock for a specific language
Write the name of the code language after ` ``` ` like **py**, **cshapr**, **shell**, ...


#### Codeblock with a title
Write after ` ``` `, ` title="bubble_sort.py ` 


#### With line numbers
Write after ` ``` `, ` linenums="1" ` 


#### Highlighting lines
Write after ` ``` `, ` hl_lines="2 3" ` or hl_lines="1-6"

## Icons and Emojs
write ` :smile: ` for :smile:   
write ` :fontawesome-regular-face-laugh-wink: ` for :fontawesome-regular-face-laugh-wink:  
write ` :fontawesome-brands-twitter:{ .twitter } ` for :fontawesome-brands-twitter:{ .twitter }  
write ` :octicons-heart-fill-24:{ .heart } ` for :octicons-heart-fill-24:{ .heart }  




