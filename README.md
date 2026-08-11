# casefy

A casing detective and conversion lab. Type text, it names the casing style
you're using, then converts your words into any case you like. Pure HTML, CSS,
and Vanilla JavaScript in a single file.

## How to run

Open `index.html` in a browser. No server, no build, no dependencies.

## Step-by-step reasoning

### 1. What is it?

The name suggests "case" — the obvious reading is text casing. I went with a
casing detective: it doesn't just convert, it names the case you're already
writing in. That gives the app one core loop: words in → detect style → show
it → convert on demand.

### 2. Why one file?

Ponytail principle: fewest files possible. Everything here is a dozen case
transforms over a `words()` tokenizer. No reason to split markup, style, and
script across three files when the whole thing fits in one readable document.

### 3. The hard part: tokenizing words

Casing is about boundaries, and boundaries hide in plain sight. `words()`
uses a single regex that catches every common boundary without needing to know
the separator first:

```
/[A-Z]+(?![a-z])|[A-Z][a-z]*|[a-z]+|\d+/g
```

- `helloWorld` → `["hello", "World"]` (lower→upper transition)
- `SCREAMING_SNAKE` → `["SCREAMING", "SNAKE"]` (all-caps runs)
- `kebab-case`, `snake_case`, `hello world` (separators simply aren't matched)

Every conversion is then a one-liner over that word list. `camelCase` is
"first word + rest capitalized, joined", `SCREAMING_SNAKE` is "all caps,
joined by `_`", and so on. The list of cases is just data, so the button grid
renders itself from it.

### 4. Detection without an if-ladder

Instead of hand-writing rules ("has `_` → snake_case"), detection works by
reconstruction: rebuild the input in every known case and return the name of
the first one that matches exactly. It's self-consistent by construction — any
case you can convert *to* is a case you can detect *as* — and it naturally
returns "unclassified" for anything that isn't a clean case.

### 5. Honest failure

The case regex is deliberately tiny and imperfect — it can't reliably split a
phrase that's both multi-separator and mixed, and `detect()` says "single
word" for inputs with fewer than two tokens. That's a labeled ceiling, not a
bug. `ponytail:` comment lives in the self-test; the upgrade path is a real
parser.

### 6. A runnable check

Non-trivial logic (a tokenizer + a matrix of transforms) leaves one check
behind: a `self-test` button that asserts the tokenizer and a handful of
round-trip conversions, failing loudly in the console and flipping the style
badge to "broken" if anything regresses. No framework, no test runner.

### 7. Beauty

Typographic obsession instead of decoration: a near-black ink-blue canvas,
two blurred drifting color orbs (pure CSS keyframes), a glass card with
`backdrop-filter`, and the app name set in wide-tracked lowercase so the
letterforms themselves echo the casing theme. Every interaction is 1-2
properties: buttons lift on hover, output flashes on convert, the badge tints
accent. Click the output to copy. Nothing blinks, nothing autoplays.

## What was deliberately skipped

- A real case parser (the regex is a named ceiling; detection is
  reconstruction-based so it's always consistent).
- Copy feedback beyond a border flash — `navigator.clipboard` + a flash is
  enough; a toast is nice-to-have.
- Persistence/URL state — the app resets on reload, and that's fine for a
  novelty tool.
