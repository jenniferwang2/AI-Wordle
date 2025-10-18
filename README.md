# Wordle AI

An information-theoretic Wordle solver that chooses each guess to **maximize expected information gain** (Shannon entropy) from the feedback pattern. It maintains a candidate list of possible answers, filters that list after every guess, and scores the next guess by how well it is expected to split (i.e., shrink) the remaining search space. On the standard word list, the solver averages **fewer than 4 guesses** per solution (measured across all answers with `evaluatePlayer()`).

## How It Works

1. **Candidate Set**
   Keep `C`, the set of all answers still consistent with the feedback seen so far.

2. **Feedback Encoding (0/1/2)**
   For a guess `g` against a secret `s`, compute a 5-tile vector:

   * `2` = green (right letter, right place)
   * `1` = yellow (right letter, wrong place; counts handled correctly)
   * `0` = gray (letter not present in the needed count)

3. **Filter**
   After each guess, remove any word in `C` that would not have produced the observed pattern.

4. **Score by Expected Entropy**
   For each candidate **guess** `w` (from `C` or the full dictionary):

   * Partition `C` by the feedback pattern you’d get if `w` were played against each `s ∈ C`.
   * Let `p_k = |bucket_k| / |C|`.
   * Compute entropy ( H(w) = -\sum_k p_k \log_2 p_k ).
   * Pick the `w` with **maximum** `H(w)`.

This explicitly prefers guesses that create **balanced branches** of outcomes, avoiding stalls where many rhyme-class words (e.g., `latch/catch/patch/hatch`) are equally valid but low-information. A deliberately “probing” word (e.g., `pilch`) can more efficiently cut the space.

## Why Entropy (vs. simple pattern scores)?

Simple “sum the 0/1/2s” or “best match” heuristics do not optimize information and often tie among many near-duplicate words, revealing little new information. Entropy directly measures how much the next guess is expected to **reduce uncertainty**.

## Key Methods

* **`feedback(guess, secret)`**
  Implements exact Wordle rules (mark greens first, then allocate yellows by remaining letter counts).

* **`getScore(guess, candidates)`**
  Simulates `feedback(guess, s)` for all `s ∈ candidates`, buckets patterns, and returns entropy in bits.

* **`chooseNextGuess(candidates, guessDomain)`**
  Evaluates `getScore` for each `guess` (answers-only or full dictionary) and returns the arg-max.

* **`evaluatePlayer()`**
  Plays the solver against every answer and reports the **mean number of guesses**.

Typical entry points:

* `play` – interactively solve a puzzle (you type in feedback after each guess).
* `evaluate` – run across all answers and print mean guesses.
* `suggest` – print the best next guess given a history of feedback.

> Note: Allowing guesses from the **full dictionary** (not just remaining answers) often improves expected information and lowers average guesses.

## Results

* Average guesses: **< 4.0** across the standard answer list (with entropy selection and correct feedback rules).
  Exact numbers vary with the word lists and whether full-dictionary guesses are enabled.

## Future Improvements

* **Optimal opener:** Compute the best first word for the current lists instead of hard-coding one.
* **Frequency-aware heuristics:** Combine position-specific letter frequencies with entropy for faster first-turn estimation.
* **Caching & speedups:** Memoize `(guess, secret) → pattern` and reuse buckets across runs.

## References

* Shannon, C. E. “A Mathematical Theory of Communication.” (for entropy ( H = -\sum p \log_2 p ))
* “Optimal Wordle” discussions/articles that analyze entropy-based strategy.
