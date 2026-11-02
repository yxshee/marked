<!-- claude-projects-attribution: {"github_login":"yxshee"} -->
_Requested by **Sir** · [project thread](https://claude.ai/code/project/chan_01FGhb5Fba7ZosnqFBVKeY7t?thread=cmsg_01FGhb5Fba7ZosnqFBVKeY7tB5HtdF6YvT4RNDg9dYg6Fi)_

**Marked version:** 18.1.2 (also master at 8babc79)

**Markdown flavor:** CommonMark

## Description

A line with less indentation than a list item is only a lazy continuation when it continues a paragraph. When the line before it ends a fenced code block, or is a heading or a thematic break inside a nested list item or block quote, there is no open paragraph, so the list should end there. marked keeps the line in the outer list item instead.

## Expectation

~~~md
- Setup
  - Install it:
    ```
    npm install
    ```
Then run it.
~~~

`Then run it.` is a paragraph after the list (commonmark.js 0.31 and the CommonMark dingus agree).

## Result

marked 18.1.2 renders `<p>Then run it.</p>` inside the outer `<li>`. The same happens for `- - # heading` and `- > ***` followed by an unindented line; with `- > ***` the text even ends up as bare text in the item.

## Cause

In the "not enough indentation" branch of the list tokenizer, `line` is the previous line already relative to the item content, but it was tested with `fencesBeginRegex(indent)`, `headingBeginRegex(indent)` and `hrRegex(indent)`, which only allow `indent - 1` leading spaces. A closing fence in a nested item (three backticks after two spaces, relative to the outer item) has more than that. Those checks also never looked past a nested list marker or `>` on the line.

The fix tests the previous line with the block rules (0 to 3 spaces), after stripping nested list and block quote markers, and only when that line was not itself a lazy line (a lazy line is paragraph text, so `- a\nb\n# c` behaves as before). Pedantic mode is unchanged.

## Contributor

- [x] Test(s) exist to ensure functionality and minimize regression: `test/specs/new/list_lazy_after_nested_block.md` covers the fence, heading, hr and block quote cases, plus a nested lazy line that must stay in the nested item.
- [ ] no tests required for this PR.
- [ ] If submitting new feature, it has been documented in the appropriate places.

## Committer

In most cases, this should be a different person than the contributor.

- [ ] CI is green (no forced merge required).
- [ ] Squash and Merge PR following [conventional commit guidelines](https://www.conventionalcommits.org/).
