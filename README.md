# time complexity and space complexity: what each one measures, how to calculate both from real code, and the trade-off interviewers want you to name

Most people first meet these two terms in the same sentence, which is exactly why they get blended together. "What's the time and space complexity of your solution?" sounds like one question. It's two, and they measure different resources.

Time complexity describes how the number of operations grows as the input grows. Space complexity describes how much extra memory the algorithm allocates on top of the input it was handed. Both are written in the same Big O notation, both ignore constants, and neither one tells you how many milliseconds anything takes on your laptop.

That last point trips up a lot of beginners, so it's worth unpacking first.

## Time complexity is a growth rate, not a stopwatch reading

You could time a function, run it on bigger inputs, and plot the curve. The problem is that the result depends on your CPU, your language, your compiler flags, and whatever else is running on the machine. Run the same code on a phone and a server and you get two different numbers for the same algorithm.

So instead of counting seconds, complexity analysis counts operations as a function of input size, usually written as n. Iterate over an array once and you do roughly n units of work. Nested loops over the same array give you roughly n² units. Binary search on a sorted array gives you roughly log n, because each comparison throws away half of what's left.

The constants get dropped on purpose. An algorithm that does 3n operations and one that does n/2 operations are both O(n), even though the first is six times slower in practice. That sounds sloppy until you try one of them with n = 10,000,000. At that size the difference between 3n and n/2 is noise compared to the difference between n and n². Big O is about which term dominates when n gets large, not about who wins at n = 10.

## Space complexity counts everything you allocate on top of the input

The input is given to you, so it isn't charged to your algorithm. What counts is the auxiliary space: variables, arrays you build, hash maps you fill, and — the part people forget — the recursion call stack.

Three quick examples make the distinction concrete.

A function that sums an array with a single accumulator uses O(1) space. One integer, regardless of array length.

A function that builds a new array of the same length uses O(n) space, because memory grows with the input.

A recursive function that calls itself n levels deep uses O(n) space even if it declares almost nothing, because every frame on the call stack holds parameters, locals, and a return address.

python
def rsum(nums):          # time O(n)
    if not nums:         # space O(n) — call stack, not variables
        return 0
    return nums[0] + rsum(nums[1:])

def isum(nums):          # time O(n)
    total = 0            # space O(1)
    for x in nums:
        total += x
    return total


Same time complexity, very different memory profiles. On a list of ten million numbers, the recursive version doesn't just use more memory — it crashes with a stack overflow, because the default recursion limit in most languages is nowhere near ten million frames.

That's the practical reason space complexity matters: it's usually the constraint that fails first in production.

## Big O, Big Omega, Big Theta — and which one interviews actually want

These three get mixed up constantly, so here's the short version.

| Notation | Bound | Plain meaning |
| --- | --- | --- |
| O(f(n)) | Upper | Grows no faster than f(n) for large n |
| Ω(f(n)) | Lower | Grows at least as fast as f(n) |
| Θ(f(n)) | Tight | Both bounds match; f(n) is the exact growth order |

In interviews, when someone asks for "the complexity", they almost always want the tight worst-case bound, which is why people write O(n log n) for merge sort rather than Θ(n log n). If you want to be precise, naming Θ shows you know the difference. If you want to be pragmatic, O is what everyone else is using.

Worth knowing: worst case, best case, and average case are separate from the notation. Quick sort has a Θ(n log n) average case and a Θ(n²) worst case. Saying "quick sort is O(n log n)" is the common shorthand and it's technically incomplete. If an interviewer asks about worst case, the honest answer involves the pivot strategy.

## The complexity classes you'll actually see

This table, in one form or another, shows up in every interview prep resource. The useful column is the last one, because it tells you how bad things get.

| Class | Name | Everyday example | Double n and you... |
| --- | --- | --- | --- |
| O(1) | Constant | Array index, hash map lookup (average) | Do the same work |
| O(log n) | Logarithmic | Binary search, balanced BST lookup | Add one step |
| O(n) | Linear | Single scan, BFS, DFS | Do twice the work |
| O(n log n) | Linearithmic | Merge sort, heap operations on n items | Slightly more than double |
| O(n²) | Quadratic | Nested loops, bubble sort | Do four times the work |
| O(2ⁿ) | Exponential | Naive recursive Fibonacci, all subsets | Square the work |
| O(n!) | Factorial | Generating all permutations | Catastrophic |

The gap between the middle rows is where most interview optimization happens. Moving a solution from O(n²) to O(n log n) is often just a sort. Moving from O(n log n) to O(n) usually requires a hash map or a two-pointer sweep. Moving below O(n) requires that you not look at every element, which is only possible on sorted or otherwise structured data.

## The space patterns worth memorizing

Space complexity is less varied than time complexity, which is good news. Almost everything lands in one of five buckets.

| Pattern | Space | Where it appears |
| --- | --- | --- |
| A fixed number of variables | O(1) | Two pointers, sliding window, in-place swaps |
| New collection proportional to n | O(n) | Hash map, frequency count, prefix sums, sorted copy |
| Call stack of depth h | O(h) | DFS on a tree, backtracking, recursive divide and conquer |
| Table indexed by two variables | O(n·m) | 2D DP grids, edit distance |
| Everything at once | O(2ⁿ) or worse | Storing all subsets, memoizing an exponential state space |

Two clarifications that come up in interviews. First, hash map operations are O(1) average and O(n) worst case, because adversarially chosen keys can collide. Second, "in-place" means O(1) auxiliary space, not "uses no memory" — swapping two elements still needs a temporary variable.

## How to read complexity off code, step by step

The mechanical procedure is less mysterious than it looks.

1. Identify the input variable, and name it. If two inputs matter, keep both — a function over two strings is O(n·m), not O(n²).
2. Nested loops multiply. A loop over n inside a loop over n gives n².
3. Sequential loops add, and you keep the larger term. Two separate loops over n give 2n, which is O(n).
4. Halving gives a log. A loop that cuts the remaining range in half each iteration contributes log n.
5. Drop constants and lower-order terms. n² + 3n + 40 becomes O(n²).
6. Recursion needs a recurrence relation, not a loop count.

Here's a worked example on a single function:

python
def shortest_window(nums, target):
    nums.sort()                     # O(n log n)
    best = float("inf")             # O(1)
    for i in range(len(nums)):      # O(n)
        for j in range(i + 1, len(nums)):   # O(n) inside
            if nums[j] - nums[i] <= target:
                best = min(best, j - i)
    return best                     # total time: O(n log n + n²) → O(n²)


The sort looks expensive until you notice the nested loops. n² dominates n log n, so the sort is irrelevant to the complexity — though sorting first is often what lets you replace the inner loop with a pointer, which is the optimization an interviewer would be fishing for.

## Recursion: recurrence relations without the drama

For a recursive function, write down the cost as T(n) and solve it. Merge sort is the standard case: split the array in two, sort each half, merge in linear time.

T(n) = 2T(n/2) + O(n)

Each level of the recursion does O(n) merging work, and the input halves until you hit size one, which takes log n levels. Total: O(n log n) time and O(n) space, because merging needs a temporary array proportional to the input. That O(n) is why merge sort isn't used for in-memory sorting in most standard libraries — quick sort and heapsort sort in place, and the memory copies cost more in practice than the asymptotic gap suggests.

Naive recursive Fibonacci is the cautionary example. T(n) = T(n-1) + T(n-2) + O(1) solves to O(2ⁿ), because the same subproblems get recomputed over and over. Adding a memo dictionary drops the time to O(n) and raises the space to O(n) — a trade you'll see again below.

## Amortized complexity: why appending to a list is O(1)

Appending to a Python list, a Java ArrayList, or a Go slice is described as O(1) even though resizing occasionally copies every element, which is O(n).

The reason is amortization. Doubling the capacity means a copy happens after 1, 2, 4, 8, 16... insertions, so n insertions trigger at most n/2 + n/4 + n/8... total copying work, which is bounded by n. Spread over n operations, each append costs a constant amount on average. Worst case for a single call is still O(n); the amortized cost is O(1).

Interviewers ask this one specifically to see whether you distinguish "average" from "amortized". They aren't the same thing. Average case is about the distribution of inputs; amortized cost is a guarantee over a sequence of operations, and it holds regardless of what the input looks like.

## The time–space trade-off, with the three examples that matter

Every optimization you'll discuss in an interview is some version of spending memory to buy time, or the reverse.

**Duplicate detection.** Checking every pair is O(n²) time and O(1) space. A hash set makes it O(n) time and O(n) space. Which is right depends on whether the array is small enough for the quadratic version and whether memory is scarce. On a 10,000-element array the brute force does about 50 million comparisons — probably fine. On 10 million elements it isn't.

**Two sum.** Brute force is O(n²) with no extra memory. The hash map version is O(n) time and O(n) space: store each value's complement as you go, and check the map before inserting. There's a third option if the array is already sorted — two pointers, O(n) time, O(1) space — which is the answer people forget when they jump straight to hashing.

**Prefix sums.** Precomputing a cumulative sum array costs O(n) space and answers any range-sum query in O(1), instead of O(n) per query. If you're answering a million queries, that's the difference between 10¹² operations and 10⁶.

Memoized Fibonacci is the same pattern in miniature: cache the answers, pay O(n) memory, replace an exponential recursion with a linear loop.

## Turning complexity into a feasibility answer

Big O becomes concrete when you attach a rough constant. On typical hardware, one second of runtime is roughly 10⁸ simple operations. That gives you a fast sanity check:

| Input size | O(n) | O(n log n) | O(n²) |
| --- | --- | --- | --- |
| n = 10³ | Trivial | Trivial | ~10⁶ ops, fine |
| n = 10⁵ | Fine | Fine | ~10¹⁰ ops, too slow |
| n = 10⁶ | Fine | Fine | Unusable |

When a problem statement says "n up to 100,000", an O(n²) solution is dead on arrival and you should be looking for O(n log n) or better. When n is 20, exponential brute force is probably intended, because 2²⁰ is a million. Reading the constraints before writing code saves more interview time than any clever trick.

## Data structure operations, in one table

| Structure | Access | Search | Insert | Delete | Space |
| --- | --- | --- | --- | --- | --- |
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Linked list | O(n) | O(n) | O(1) | O(1)* | O(n) |
| Stack / queue | O(n) | O(n) | O(1) | O(1) | O(n) |
| Hash map | — | O(1) avg | O(1) avg | O(1) avg | O(n) |
| Balanced BST | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| Binary heap | O(1) for min | O(n) | O(log n) | O(log n) | O(n) |

\*O(1) only when you already hold a reference to the node; finding it is O(n).

## The mistakes that cost points

Saying "O(n²) because two loops" when the loops are sequential rather than nested.

Reporting O(n) for a recursive solution without counting the call stack.

Giving the average case when the interviewer asked for the worst case, or the reverse. Quick sort is the classic trap.

Forgetting that a string isn't free. Building a result by concatenating in a loop is O(n²) in languages with immutable strings, even though it looks linear.

Using amortized bounds to describe a single operation. "Append is O(1)" is a statement about a sequence, not about the one call that triggers a resize.

Not mentioning the auxiliary space at all. Interviewers ask for "time and space" and hear only half the answer more often than you'd expect.

## Where this skill is actually built

Definitions are easy to read and hard to internalize. The part that takes practice is looking at unfamiliar code and producing "O(n log n) time, O(n) space" without counting on your fingers — and knowing which of the two the interviewer cares about when they push back.

AlgoMonster is a coding interview prep platform built around that skill. It's put together by engineers who worked at Google along with competitive programmers, and the curriculum is organized around reusable patterns rather than a wall of individual problems. The delivery format is text, illustrations, and editorials — no video walkthroughs — with an interactive AI assistant for hints and explanations, a Speedrun mode for timed review, and company-specific question sets that get refreshed monthly. Current content runs to 500+ lessons across six courses, 1,000+ illustrations, and 3,000+ editorials, plus Foundation and Build an LLM courses bundled in. Complexity analysis shows up throughout, because you can't describe a sliding window or a DP transition without stating its cost.

If you want to see how the course actually presents complexity analysis before paying anything, the free tier gives limited access — 👉 [browse the free lessons and current plan pricing](https://algo.monster?fpr=li69).

## Plans and pricing

AlgoMonster doesn't sell a monthly subscription. The options are a free tier with limited content, an annual Pro plan, and a one-time lifetime purchase.

| Plan | What you get | Price | Billing | Get access |
| --- | --- | --- | --- | --- |
| Free | Limited access to sample lessons and problems | $0 | — | [Start on the free tier](https://algo.monster?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI assistant and Speedrun; 500+ lessons across 6 courses; company question sets updated monthly; member discount on live bootcamps; 12 months of access | $99/year (about $8.25/month) | Billed annually, cancel or upgrade anytime | [Check the yearly plan](https://algo.monster?fpr=li69) |
| Pro – Lifetime | Everything in the annual plan plus lifetime updates, paid once | $189 | One-time payment | [See the lifetime price](https://algo.monster?fpr=li69) |

Two things worth knowing before you buy. AlgoMonster runs sales frequently and the listed prices move with them, so the number you see at checkout may be lower than the table above — the link always shows the live price. And the annual versus lifetime math is simple arithmetic: if you expect to interview more than once in the next few years, the lifetime plan is cheaper than two annual renewals. If this is a single three-month push, buy the year.

One caveat about expectations: some specialized courses developed with outside partners, like Quant Interview Prep, are sold separately and aren't included in either Pro plan.

## A two-week way to actually get good at this

Reading this article once won't make complexity analysis automatic. What works is deliberate reps.

Spend the first few days on the classes and the mechanical rules — the loop rules from earlier, the recurrence pattern, the difference between amortized and average. Then take code you've already written and annotate it: write the time and space complexity for every function in a project you own, including the ones you think are obviously O(n). You'll find at least one that's quietly quadratic. After that, work through problems where the constraint forces the answer — if n is 10⁵, the intended solution is O(n log n) or O(n), and working backwards from constraints to complexity is the fastest version of this skill to acquire.

By the end of two weeks you should be able to look at a nested loop, a recursive call, and a hash map insertion and state both complexities in the same sentence, which is exactly what the interview asks for.

## FAQ

**Is space complexity always less important than time complexity?**
No, and embedded systems, mobile clients, and streaming pipelines are where it bites. An algorithm that needs O(n) memory to process a 10 GB file won't run at all, regardless of how fast it is. That's why single-pass streaming algorithms exist.

**Does space complexity include the input array?**
Only if you're talking about total space. Auxiliary space — the memory your algorithm allocates beyond the input — is the standard measure, and it's what interviewers mean unless they say otherwise.

**Can an algorithm have better space complexity than time complexity?**
Yes, and they're independent. Bubble sort is O(n²) time and O(1) space. Binary search is O(log n) time and O(1) space. There's no rule linking them.

**Is O(1) space always better?**
Not always in practice. In-place sorting algorithms can be slower than ones that copy data, because copying into contiguous memory is cache-friendly. Complexity analysis describes growth rates, not absolute speed.

**Do I need the master theorem for interviews?**
For most coding interviews, recognizing the shape of common recurrences is enough. Merge sort's 2T(n/2) + O(n) → O(n log n) and binary search's T(n/2) + O(1) → O(log n) cover the majority of what gets asked.

**What should I say when I don't know the complexity?**
Describe the work instead of guessing. "The outer loop runs n times, and for each iteration the inner loop scans the remaining elements, so it's quadratic" is a better answer than a wrong class name, and it usually gets you partial credit for the method.
