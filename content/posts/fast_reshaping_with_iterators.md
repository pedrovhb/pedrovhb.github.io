---
title: "Fast reshaping with iterators"
date: 2021-07-05T15:04:32-03:00 
draft: false
---

Lately I've been reading up a bit more on functional programming. I feel that
after years of knowing about the existence of the paradigm and even trying to
dip my toes in a couple of times without success, now it finally "clicked".

I'm still playing around with concepts and languages, and trying to look at some
programming exercises in a more functional light. It was surprising to me,
though, how easily and efficiently I was able to
solve [today's LeetCode challenge](https://leetcode.com/explore/challenge/card/july-leetcoding-challenge-2021/608/week-1-july-1st-july-7th/3803/)
. The problem was fairly simple - reshape an MxN matrix (presented as a list of
lists) into another matrix that is RxC (an arbitrary number of rows and columns)
while keeping the original matrix elements.

Here's the solution in its entirety:

```python
from itertools import chain, islice
from typing import List


class Solution:
    def matrixReshape(self, mat: List[List[int]], r: int, c: int) -> List[
        List[int]]:

        # If the proposed RxC matrix would have a different 
        # number of elements, return the matrix itself
        if len(mat) * len(mat[0]) != r * c:
            yield from mat
        else:
            # Create an iterable over all the values of `mat`
            it = chain.from_iterable(mat)

            # Yield from an iterator of r c-sized iterators   
            yield from (islice(it, c) for _ in range(r))
```

Interestingly, LeetCode allows us to return iterators where lists are expected,
so we don't have to litter everything with `list` or explicit comprehensions.

My first thought is that this is a bit terse, and it's certainly less obvious
than busting out the for-loops. I'm not convinced that's necessarily a bad
thing, though. Of course, overcomplicating things is bad and code should be as
simple as it can be, but this is essentially a higher-level way of doing things
- instead of writing loops, we think about data streams and dimensions. It's
natural that using a different way of thinking requires learning new idioms.
While this may be alien for someone who isn't familiar with the concepts, it's
not really intractable. It's quite worth taking the time to understand have it
in your toolbelt.

On the other side, we have the hints given by LeetCode on how to solve this.
They actually made me chuckle thinking about how much more inelegant other
solutions dealing with indexes must be (spoiler alert):

> Do you know how 2d matrix is stored in 1d memory? Try to map 2-dimensions into one.

> M[i][j]=M[n*i+j] , where n is the number of cols. This is the one way of converting 2-d indices into one 1-d index. Now, how will you convert 1-d index into 2-d indices?

> Try to use division and modulus to convert 1-d index into 2-d indices.

> M[i] => M[i/n][n%i] Will it result in right mapping? Take some example and check this formula.

Those all look terrible. We didn't have to deal with a single index in our
iterator solution!

What's more, it's *fast*, beating 98.89% of other submitted Python solutions:

![Pedro](/img/2021/fast_reshaping/performance_reshape.png)

I actually thought that it could've been a bug with how LeetCode analyzes
submissions - maybe it'd evaluate and measure the function execution time, and
then check the results. In that case, we'd get a wrong assessment of the
solution because the iterators we used are lazy, i.e. they're only evaluated
when needed (which would've been after profiling if the above were correct). But
it seems that's legit, actually. At the very least, iterators are consumed, as
the absolute difference to other submissions wasn't as massive as to imply there
was no iteration whatsoever going on.

So there we go! An elegant, fast solution to an otherwise hairy problem in (
effectively) two lines of functional-style Python :)
