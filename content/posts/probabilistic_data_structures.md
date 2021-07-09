---
title: "Probabilistic data structures - bloom filters"
date: 2021-06-20T17:24:16-03:00 
draft: false
------------

Traditionally in programming we think about 0s and 1s as being infallible. They
either are, or aren't, and other than for cosmic rays flipping bits in RAM (it
happens) there's not much margin for uncertainty. It's one of the nice things
about computers: our memory is imperfect, but theirs isn't.

What if we could trade off a bit of this consistency for a lot of performance,
though? Well, it turns out that we sometimes can, by using **probabilistic data
structures**. These are very useful tools for a few specific use cases.

Let me tell you about two of them and how they may be used. This blog post will
cover bloom filters, and in a second part we'll go over HyperLogLog. I won't go
deep into how they work internally (even though that's fascinating) in order to
focus on giving a high-level overview of tradeoffs and practical applications.

## Bloom filters

A bloom filter is a probabilistic data structure that can quickly and
efficiently check whether an elements is included in a (possibly very large) set
of values. It offers only two operations: adding elements and verifying
membership, both of which are executed in constant time (in other words, they're
O(1) operations). That's it - you can't remove an element after adding it, and
you also can't retrieve previously inserted elements or iterate over them.
Another thing to know is that, as per the theme of the post, it's fallible,
meaning it doesn't always return the correct answer if you ask it whether an
element is contained or not.

So this sounds pretty bad. Why would anyone use that?

Well, the reason anyone uses probabilistic data structures is that they're
really damn efficient.

Parameters are customizable, and you can have as much or as little accuracy as
you want, but a ballpark figure is that **an optimized bloom filter set with
0.1% error rate requires only around 14.4 bits of memory per element**.

Those are bits, not bytes. If we're expecting our total set to contain about 10
thousand elements, then the size taken up by the bloom filter for that will be
only about 18 kB. This size also isn't affected by the size of the objects
contained; we could be keeping track of 10k distinct integers, or 10k
multi-gigabyte video files, and the final number would be the same 18 kB. Cool!

It's also not fair to just say "it doesn't always return the correct answer".
The defining property of bloom filters is that false positive matches are
possible, but false negatives are not. It tells you either "this element is
probably in the set" or "this element is definitely not in the set". If it says
something isn't contained in the set, you can be 100% sure it really isn't.

### But how?

For a *very* brief description of how it works - the bloom filter uses multiple
hash functions to process the input, and maps the results to a bit array, OR'ing
previous items that were there.

When a query comes in, the value to be checked is hashed the same way as values
added to the filter are, and the resulting set bits are compared to the bits set
in the filter. If all resulting bits are also set in the filter, it means either

- The value was previously added to the bloom filter and it's a positive
- By coincidence, the hashes of other values have set the all the same bits and
  it's a false positive

Hash functions are designed have sparse and uniformly distributed results so
false positives are as unlikely as possible, but it can happen.

If any of the resulting bits set in the value's hash are *not* set in the bloom
filter, then we know for sure that the value can't have been added to the filter
previously. Adding a value is simple as OR'ing the current bloom filter with the
result of its check.

### Where to use it

So what applications does this interesting set of properties lend itself to?

For a long time, Chrome shipped with a pre-built bloom filter of known malicious
URLs. When a user accessed a website, Chrome checked the bloom filter to see if
the URL was contained within it. If not, it'd let the request through as normal,
and the check was pretty fast as it happened locally. If the filter said the URL
may be contained in the bad website list (which isn't, hopefully, the most
frequent case) then it would make a request to Google's servers to ask if the
URL was dangerous for sure, and flash a scary red screen to warn the user if it
was.

The cost incurred was a tiny amount of storage for the end user (in the order of
kilobytes of disk/memory), and the upside was that a ton of requests were
avoided, saving Google lots of money in server costs and making the user's
experience instant in almost all cases. In fact, I'd say the entire feature was
made possible by the data structure, as it'd be impractically costly and slow to
check each website accessed by each user and including a several-megabyte list
of URLs in an installer is a big ask.

A similar idea is to keep a bloom filter as a sort of index to prevent
unnecessary I/O operations. In a file storage system, for instance, we may have
several hundred thousand different files. Disk operations are slow compared to
CPU cycles and RAM access, and if your program needs to check whether some of
these files exist (maybe you want to try to access them only if they do), it may
spend a lot of time waiting for the disk to spin up, look for the file you want,
and come back to you with an answer.

You could scan the disk once and store each existing filepath in an in-memory
hashlist for quickly checking, and that's a great idea, but we can avoid a big
chunk of the resulting memory cost by using a bloom filter instead. When you
need to check for a file's existence, query the bloom filter - if the file
doesn't exist, it'll tell you immediately. If the bloom filter says the file
*probably* exists (remember, false positives are possible), then you'll need to
hit the disk and check. The benefit is that you save a lot of time on negative
hits for only a negligible memory cost. The time saved is even larger if these
checks happen over a network instead of a disk.

Of course, we need to think about the details for our specific problem. This
works best when we expect most checked files to not be contained in the set.
Also, there's no way to remove elements from the filter, so we have to rebuild
it when files are deleted or moved. and it might not be worth it if that happens
often (though we can keep partial filters and rebuild only a deleted file's
directory's filter, for instance; combining them is as simple as OR).

Hopefully that adds a tool to your arsenal, and gets you interested in
probabilistic data structures. Stay tuned for the next part in which we'll have
a quick look over HyperLogLog!

*This was originally posted on 2019-09-06 as a one-part post on my old blog,
which I ended up losing. Recovered it from LinkedIn, made some changes and split
it in two for easier digestion. Second part coming soon :)*
