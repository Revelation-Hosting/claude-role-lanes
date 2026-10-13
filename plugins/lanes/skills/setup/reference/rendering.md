# Template rendering rules

The templates in `templates/` use a small Handlebars-style tag set. These rules are the whole language: a construct not
listed here is not supported. Render by hand, or with a throwaway script run from stdin or the scratchpad. Never write a
renderer into the TARGET. Either way, these rules are the specification.

## Values

- **Config** = the parsed `.claude/lanes.yml`. A **path** `a.b.c` names a nested key.
- `{{a.b}}` is replaced by the value at that path, **literally**: no escaping, no added quotes, no added backticks.
- A multi-line value (a YAML `|` block, used by `roles.<role>.extra` and `process.extra`) is inserted with its own
  line breaks, minus its final line break. If the tag is indented, indent every non-empty line of the value to the
  tag's column. Empty lines stay empty.
- A value tag (`{{a.b}}`) that names a **required** key (see `config.md`) with no value is an error. Stop, name the
  key, and ask for the value. Never guess one. Only value tags that are rendered count: a tag inside a dropped block is
  never checked, and a condition (`{{#if a.b}}`) is never an error.
- A tag that names an optional key with no value renders as empty text.

## Truthiness

`false`, `null`, a missing key, `""` and `[]` are false. Everything else is true.

## Block tags

| Tag | Renders |
|---|---|
| `{{#if X}} A {{/if}}` | A when X is true. |
| `{{#if X}} A {{else}} B {{/if}}` | A when X is true, else B. |
| `{{#if (eq X "v")}} A {{/if}}` | A when X is exactly the string `v`. `{{else}}` works here too. |
| `{{#unless X}} A {{/unless}}` | A when X is false. `{{#unless (eq X "v")}}` is A when X is not exactly `v`. |
| `{{#each L}} A {{/each}}` | A once per item of list L, in order. Nothing when L is empty or missing. |

- Inside `{{#each}}`, `{{this}}` is the current item (a scalar), and `{{this.k}}` is field `k` of the current item.
  `{{#if this.k}}` tests a field of the current item.
- Blocks nest. A close tag closes the nearest open block of its kind.

## Comments

`{{!-- text --}}` is removed. A line that holds only a comment is removed, with its line break.

## Whitespace

1. **Standalone tag lines are removed.** A line that holds exactly one block tag (`{{#if ...}}`, `{{else}}`,
   `{{/if}}`, `{{#unless ...}}`, `{{/unless}}`, `{{#each ...}}`, `{{/each}}`) and nothing else but spaces is removed
   entirely, with its line break. This applies inside YAML frontmatter too.
2. **Inline tags are replaced in place.** A tag that shares its line with other text is replaced where it stands. The
   text around it is unchanged. An inline block may span several lines; its whole content is kept or dropped as one
   unit.
3. **Blank-line runs collapse.** After rendering, outside fenced code blocks, replace every run of two or more blank
   lines with one empty line. A line that holds only spaces is blank. Remove blank lines at the very start of a file.
4. **Fences.** A line whose first non-space characters are three backticks opens or closes a fenced code block, at any
   indentation. Tags inside fenced blocks are rendered like any other tags.

## Literal text

Everything that is not a tag is copied **byte for byte**: punctuation, backticks, `$`, `\(`, line breaks, line
lengths, ASCII hyphens. Do not fix grammar, re-wrap lines, shorten, reorder, translate or "improve" template text. If
template text is wrong for a project, the fix is a config value or `roles.<role>.extra`, never an edit during render.

## Self-check after each file

- Search the rendered text for `{{`. Every match is a render bug. Fix it before writing the file.
- In an agent file, the frontmatter must still open with `---` on line 1 and close with `---`. Every line of the
  `initialPrompt: |` block must start with 2 spaces.
