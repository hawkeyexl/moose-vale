# moose-vale

One Vale package that gives every hawkeyexl repo the same voice. It carries the [Voices](https://github.com/jdkato/voices) core and the **Direct** voice, vendored with a few fixes, and adds one house rule. Nothing else.

Direct means no hedging, no preamble, and sentences under 25 words.

## Use it

Requires Vale 3.20.0 or later.

Put this in the repo's `.vale.ini`:

```ini
StylesPath = .vale/styles
MinAlertLevel = suggestion
Packages = https://github.com/hawkeyexl/moose-vale/releases/latest/download/Moose.zip
```

Then run:

```bash
vale sync
vale .
```

Add `.vale/styles/` to `.gitignore`. Sync fetches the styles; do not commit them.

The package ships its own config. It enables `Voices`, `Direct`, and `Moose` for every `.md`, `.mdx`, and `.txt` file. You do not write a `BasedOnStyles` line.

## If you write your own sections

Vale applies the last matching section and drops the rest. A section like `[docs/**] BasedOnStyles = Google` silently removes the house voice from `docs/`. Always list the three styles in any section you add:

```ini
[docs/**]
BasedOnStyles = Voices, Direct, Moose, Google
```

## Files Vale cannot parse

A file with broken frontmatter, or an MDX table the parser rejects, aborts the whole run. No level setting downgrades that. Keep such files out with `--glob`, and name each one so the exclusion stays narrow:

```bash
vale --glob='!{test/fixtures/broken.md,docs/generated/**}' .
```

In the GitHub Action, pass the same value through `vale_flags`.

## Dials

Local config wins over the package. Common overrides:

```ini
Direct.Length[max] = 30
Voices.ColonReveal = NO
Moose.EmDash = NO
```

## What is in the package

| Style | Source | What it checks |
|-------|--------|----------------|
| `Voices` | vendored from jdkato/voices | Banned words, puffery, weasel words, weak verbs, throat clearing, recaps |
| `Direct` | vendored from jdkato/voices | Hedging, preamble, sentence length |
| `Moose` | this repo | Em dashes. Rewrite the sentence instead of swapping punctuation |
| `Std` | vendored from vale-cli/Std | Installed, not enabled. `Direct.Length` extends its sentence-length rule, and you can enable any `Std.*` rule yourself |

The Voices, Direct, and Std rules are copied into this package, not fetched. The package has no nested dependency, so `vale sync` installs these four styles and nothing else.

## Fixes carried against upstream

`NOTICE` lists the upstream commit and every change. In short:

- `Voices.WeakVerbs` has one replacement per inflection, so the auto-fix is always grammatical.
- `Voices.InflatedWords` keeps the matched word's case when it replaces it.
- `Voices.ColonReveal` and `Voices.BinaryContrast` cannot match across a line break, and `BinaryContrast` accepts hyphenated phrases.
- `Std.Usage.GenderedTerms` no longer carries a stray swap entry that flagged every occurrence of the word "name".

## Refresh from upstream

Copy the rule files from `jdkato/voices` into `Moose/styles/Voices/` and `Moose/styles/Direct/`, and from the `vale-cli/Std` release into `Moose/styles/Std/`. Re-apply the fixes above, update the versions in `NOTICE`, run the tests, and cut a new release. Every consumer picks it up on its next `vale sync`.

## Release

Tag a version and push the tag. The release workflow zips `Moose/` and attaches `Moose.zip` to the GitHub release.

```bash
git tag v0.1.0
git push origin v0.1.0
```

## Test locally

```bash
zip -r Moose.zip Moose
mkdir -p /tmp/consumer && cp fixtures/*.md /tmp/consumer/
printf 'StylesPath = styles\nMinAlertLevel = suggestion\nPackages = %s\n' "$PWD/Moose.zip" > /tmp/consumer/.vale.ini
cd /tmp/consumer && vale sync && vale --no-exit bad.md && vale good.md
```

`bad.md` reports `Direct.Hedging`, `Direct.Preamble`, `Direct.Length`, `Moose.EmDash`, `Voices.InflatedWords`, `Voices.WeakVerbs`, and `Voices.BinaryContrast`. `good.md` reports nothing.
