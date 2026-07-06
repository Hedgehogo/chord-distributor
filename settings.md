> **Note on terminology:** A **base** is a sequence of characters that best represents a lemma, but is not a linguistic stem. For example, in words with a "fleeting" vowel, the base will likely omit that vowel. Words with alternating consonants will typically be split into multiple bases. Words sharing the same base may be merged. For example, the English word *choose* will likely receive the base `chose`, because those letters appear in that order across all its forms. If this concept feels abstract, you can mentally substitute *base* with *word* — the configuration remains broadly understandable.

The file contains the following fields:
- `top_count` (required) — the number of top bases (simplified: the most frequent) to include in the distribution.
- `distributor` (required) — settings related to chord allocation between bases.
- `generator` — settings related to chord generation and evaluation for each individual base.

## `distributor`
Chord distribution uses a **sliding window** approach. The window moves from the top of the base list to the bottom. It covers a certain number of bases at a time, solves their allocation optimally, then shifts down by a fixed step, fixes the chords that have exited the window, and repeats.

The `distributor` section contains the following fields:
- `window_size` (required) — the number of bases covered in one pass. This value should not exceed `top_count`, though it is not strictly forbidden. Larger values improve solution quality but increase processing time.
- `window_pitch` (required) — the step by which the window moves down. Must not exceed `window_size`. Smaller values improve solution quality but increase processing time.
- `fixed_chords` — a mapping from bases to manually assigned chords (e.g., "ли": "л\\d").
- `forbidden_chords` — a list of chords that are forbidden for any base assignment.

If you set `window_size` and `window_pitch` to one, then the bases will simply receive the first of the remaining best chords in order.

Example with all fields filled:
```json
{
	"top_count": 500,
	"distributor": {
		"window_size": 100,
		"window_pitch": 50,
		"fixed_chords": {
			"ли": "л\\d"
		},
		"forbidden_chords": [
			"от"
		]
	}
}
```

## `generator`
The `generator` section contains the following fields:
- `length_limit` (default: `8`) — maximum base length processed by the generator. If exceeded, extra letters are truncated. Larger values increase processing time.
- `half_penalty` (default: `1.5`) — the penalty threshold at which a chord becomes half as good as one that preserves a fully memorable and ergonomic letter set. In other words, how many ideal letters you are willing to lose before the chord quality is reduced by half.
- `mnemonic_preference` (default: `0.5`) — coefficient balancing mnemonics vs. ergonomics, where:
  - `0.0` — only ergonomics matters
  - `1.0` — only mnemonics matters