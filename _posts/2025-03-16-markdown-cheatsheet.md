---
title: Markdown Cheat Sheet
date: 2025-03-16
layout: post
categories: [text]
---
This should come handy as I start using markdown to create posts. Based on excelent reference here: <https://www.markdown-cheatsheet.com/>

<br>
#### Headings

---

## Heading level 2 
### Heading level 3 
#### Heading level 4 
##### Heading level 5 
###### Heading level 6

<br>
#### Basic / Emphasis

---

**bold text**  
__bold text__  
*italic text*  
_italic text_  
***bold + italic text***  
___bold + italic text___  
~~strikethrough text~~  
> Blockquote
>
>> Nested blockquote

<br>
#### Lists

---

1. First item
2. Second item
    1. Indented item
    2. Indented item
3. Third item

- First item
- Second item
    - Indented item
    - Indented item
- Third item

- [ ] Checkbox 1
- [x] Checkbox 2
- [ ] Checkbox 3

<br>
#### Line breaks & paragraphs

---

To insert a line break (new line), place two or more trailing spaces at the end of the line.  
Separate paragraphs by a single blank line.  
Add HTML tag \<br> to introduce a blank line

This is the first line.  
And this is the second one.

This is the first paragraph.

And this is the second one.

<br>
#### Horizontal rules

---

To create a horizontal rule, use three or more dashes (---), underscores (___), or asterisks (***).  
Always put blank lines before and after the horizontal rule.

---

___

***

<br>
#### Tables

---


To create a table, use three or more hyphens (---) under each column’s header, and use pipes (\|) to separate each column. We recommend using [Markdown table converter](https://tableconvert.com/csv-to-markdown).


| ID | Title |
|---|-------|
|#1 | Hello |
|#2 | Markdown |

<br>
#### Escaping characters

---

To escape any markdown-sensitive character, place a backslash (\\) before the character.

Without *escaping*  
With \*escaping\*

<br>
#### Code

---

Here is some `inline code`

```
// Multiline code
const message = 'Hi!';
```

``Escaping `backticks` in the inline code``

<br>
#### Links & images

---


[Link](https://google.com)

[Link with title](https://google.com "Here the title goes")

<https://google.com>

![Markdown image](https://www.markdown-cheatsheet.com/images/markdown.png)

[![Markdown clickable image](https://www.markdown-cheatsheet.com/images/markdown.png "Click me!")](https://google.com)
