# valid parentheses: solve LeetCode 20 with one stack, the edge cases that break most answers, and the follow-ups interviewers ask next

Most people land on this problem within their first week of interview prep, solve it in ten minutes, and then quietly get marked down in a real screen for the same three reasons everyone else does. The question is short enough to read in fifteen seconds, which is exactly why the details matter more than they look.

## What "valid parentheses" is actually asking

You get a string made only of the six bracket characters: `(`, `)`, `{`, `}`, `[`, `]`. It's valid when all three conditions hold:

1. Every opening bracket is closed by the same type of bracket.
2. Brackets close in the correct order.
3. Every closing bracket has a matching opening bracket before it.

The `[]` half of that list is where the difficulty lives. Order, not just count.

| Input | Output | Why |
| --- | --- | --- |
| `()` | true | One matched pair |
| `()[]{}` | true | Three pairs, side by side |
| `(]` | false | Wrong type |
| `([)]` | false | Right types, interleaved wrongly |
| `{[]}` | true | Properly nested |

Constraints are tame. LeetCode's version caps `s.length` at 10⁴ and allows only the six bracket characters. The AlgoMonster version of the same problem lists a ceiling of 100,000. Either way you're writing a linear solution — no cleverness required beyond picking the right data structure.

One historical footnote worth knowing since it shows up in a lot of older write-ups: the empty string used to be explicitly labeled valid. Current constraints start at length 1, so `""` rarely comes up, but if an interviewer drags it out, "vacuously valid" is the safe answer.

## Why a stack, and not a counter

The instant instinct for a lot of beginners is to count. One counter for opens, one for closes, check they match. That works for the single-bracket version of the problem and fails immediately here:

- `)(` has one `(` and one `)`. Counts are equal. The string is still invalid, because the close came first.
- `([)]` has equal counts per type. Still invalid, because `[` was closed by `)` in the wrong order.

If you ever want to look clever in an interview, say out loud: the last bracket opened is the first one that must be closed. That's the LIFO property, and LIFO means stack.

Nesting is the reason. A valid expression has recursive structure — `{[]}` is a valid sequence containing `[]`, which is itself valid — and you can't process that structure from the inside out, because you don't know how deep it goes until you're already inside it. A stack lets you process it outside-in, one character at a time.

## The stack solution, character by character

Take `{[]}`:

1. `{` — opening bracket, push. Stack: `{`
2. `[` — opening bracket, push. Stack: `{ [`
3. `]` — closing bracket. Top of stack is `[`. Match. Pop. Stack: `{`
4. `}` — closing bracket. Top of stack is `{`. Match. Pop. Stack is empty.
5. End of string, stack is empty → valid.

Here's the version I'd write, in Python:

python
def isValid(s: str) -> bool:
    pairs = {")": "(", "]": "[", "}": "{"}
    stack = []

    for ch in s:
        if ch in pairs:
            if not stack or stack[-1] != pairs[ch]:
                return False
            stack.pop()
        else:
            stack.append(ch)

    return not stack


Two details in there are doing real work. The lookup `if ch in pairs` treats closing brackets as the interesting case and everything else as an opening bracket, which quietly handles the fact that the input only contains those six characters. And `not stack or stack[-1] != pairs[ch]` collapses two separate failure modes — closing with nothing open, and closing with the wrong type — into one guard.

Plenty of people write it with explicit branches instead:

python
if ch == '(':
    stack.append(')')
elif ch == '[':
    stack.append(']')
elif ch == '{':
    stack.append('}')
else:
    if not stack or stack.pop() != ch:
        return False


That's the "push what you expect to see" variant. It's not shorter, but it does mean the matching logic lives in one place instead of two, and the pop happens before the comparison so an empty stack short-circuits correctly. Either is fine; the version you can write from memory under pressure is the correct one.

## The four edge cases that actually get people rejected

The happy path takes three minutes. These take longer to remember, and interviewers ask about all of them.

**Closing bracket on an empty stack.** Input `")"`. If your code calls `stack.pop()` or `stack[-1]` before checking whether the stack has anything in it, you either crash or throw an out-of-range error. In a real screen that reads as "doesn't handle invalid input," which is a worse signal than a wrong final answer.

**Wrong type, right timing.** Input `"(]"`. A stack that only checks whether *something* is on top, without comparing bracket types, will happily pop and return true. This is the single most common bug in solutions posted online, and it usually comes from writing three parallel `if` branches and getting one of the comparisons backwards.

**Leftover opening brackets.** Input `"(()"`. Every closing bracket matches fine. The stack still contains an unmatched `(` when you hit the end. The final `return not stack` is not decoration — it's a third of the answer.

**Length parity.** A string with an odd number of characters can't possibly be valid. You don't need to check it — the main loop handles it — but adding `if len(s) % 2: return False` at the top is free and reads as someone who thinks about invariants.

## Complexity, and why it stops there

Time is O(n): one pass, and each character is pushed or popped at most once, so no character is touched more than twice. Space is O(n) in the worst case — a string like `"((((("` pushes everything and pops nothing.

The O(n²) alternative is the elimination approach: keep deleting adjacent matched pairs like `()` or `[]` until nothing can be deleted, then check whether the string is empty. It's a legitimate way to reason about the problem (each deletion shortens the string, and the shrinking preserves validity), and it's a decent sanity check on paper. It's just slower, and in an interview it's the answer you offer as "here's how I'd think about it before reaching for the stack."

Recursion also works, since the structure is recursive by definition, but it's the same O(n) time with call-stack overhead and none of the readability.

## Where this problem sits in the wider pattern set

Almost nobody gets asked valid parentheses in isolation. It's a warm-up that leads somewhere:

- **Generate all valid parentheses** — the same nesting structure, but you're building strings with backtracking, and the count of results for `n` pairs is the nth Catalan number. For `n = 3`, that's five strings: `((()))`, `(()())`, `(())()`, `()(())`, `()()()`.
- **Longest valid parentheses** — now the stack stores indices instead of characters, and the DP version is worth knowing too.
- **Minimum remove to make valid parentheses** — the stack tracks unmatched positions so you can delete them.
- **Minimum add to make parentheses valid** — how many characters you'd need to insert.
- **Valid parenthesis string** — asterisks that can be `(`, `)`, or nothing, which is the greedy-rule version of the same family.

If you only memorize the stack solution to problem 20 and never see those five, you've learned one answer rather than one pattern. That distinction is the whole pitch behind pattern-based prep, and it's a fair one: recognizing "this is a stack-and-matching problem" in the first thirty seconds of a screen is worth more than having solved forty random questions.

AlgoMonster is built around exactly that idea. It's a text-and-diagram platform from ex-Google engineers that organizes practice into 48 named patterns instead of a flat problem list, and valid parentheses lives in its foundation material with a step-by-step breakdown: identify the pattern, then work the test cases, with study notes attached to each problem. You can look at 👉 [how they structure the valid parentheses lesson](https://algo.monster?fpr=li69) before deciding whether it's worth paying for anything. The free tier is genuinely limited — enough to judge the teaching format, not enough to run a full prep cycle on.

What it isn't: there are no video walkthroughs at all, and the problem count is in the low hundreds rather than thousands. If you learn by watching someone else code, that's a real mismatch. If you learn by reading and typing, the diagram-heavy format is faster than scrubbing through a 40-minute video.

## Plans and current pricing

AlgoMonster runs three tiers, no monthly option.

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited sample of patterns and problems | $0 | — | [Open the free tier](https://algo.monster?fpr=li69) |
| Pro — Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; company questions updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials; member discount on live bootcamps | $5.84/mo, billed annually ($69.99/year) | Recurring, 12 months of access, cancel or upgrade anytime | [Start the yearly plan](https://algo.monster?fpr=li69) |
| Pro — Lifetime | Everything in the yearly plan, plus lifetime updates to anything added later | $89 | One-time payment | [Buy lifetime access](https://algo.monster?fpr=li69) |

Two honest caveats about those numbers. First, AlgoMonster's pricing moves with whatever promotion is running — 2026 reviews from different sites quote anywhere from $99 to $299 for the yearly plan and $189 upward for lifetime, because they caught different sales. Treat the checkout page as the live number, not any review, including this one. Second, the plan structure hasn't changed: everything except the free tier unlocks all patterns and all problems, so the choice is really about how long you want access, not about content tiers.

For the break-even math, the current yearly price is $69.99 and lifetime is $89. If you're preparing for one interview loop over the next three months and then closing the tab for two years, yearly wins. If you'd renew even once, lifetime already costs less than two years of annual billing.

## Questions people actually ask about this problem

**Can I solve valid parentheses without a stack?** Yes, with the repeated-elimination approach, at O(n²). It's a reasonable thing to mention in an interview as a first pass, then improve. Counting alone only works when there's a single bracket type.

**Does the solution handle very long inputs?** The stack approach handles 10⁴ and 10⁵ character strings without any changes. Python's recursion version would hit the default recursion limit around 1,000 nested frames, which is one more reason the iterative stack is the right default.

**Is solving valid parentheses enough to pass a coding screen?** It's a warm-up, not a screen. Expect a variant or a follow-up within the same session.

**Should I memorize the solution?** Memorize the invariant instead — the most recent unmatched opening bracket must be matched first — and the code writes itself. That's also what transfers when the follow-up changes the rules on you.

If you want a structured route through the pattern rather than a problem to tick off, 👉 [practice valid parentheses with the guided breakdown](https://algo.monster?fpr=li69) and check whether the written format suits how you study before you commit to a paid plan.
