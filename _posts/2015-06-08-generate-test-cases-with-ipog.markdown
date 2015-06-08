---
layout: post
title:  "Generate Test Cases with IPOG"
date:   2015-06-08 12:00:00
categories: testing
---

Creating test cases for black box testing is tedious for complex applications.
It is easy to bias some scenarios and completely ignore others.
At the same time it is unreasonable to create an exhaustive test suite due to the complexity of the application.

One solution to this problem is [pairwise](http://www.pairwise.org/) testing.
In pairwise testing one creates a test suite that covers all combinations of two parameters.
This allows one to create a small test suite that is effective at finding faults.

Unfortunately, I did not find many open source libraries when I looked into applying this technique.
My requirements for a good t-way testing are as follows:

* support t-way testing
* support constraints
* support existing tests
* open source
* should be able to run the tool on a headless linux machine

Both [jenny](burtleburtle.net/bob/math/jenny.html) and [AllPairs](https://pypi.python.org/pypi/AllPairs/2.0.1) fit these constraints.
Jenny is a good command line application in C that is referenced a lot in academic articles.  AllPairs is a python library.
AllPairs did not pass my smoke tests for t-way testing where t > 2.  While I like Jenny I wanted something in ruby.
There is [pairwise](https://github.com/josephwilk/pairwise) but this gem does not support t > 2, constraints, and has not been active for 3 years.

Hence, I created my own implementation of IPOG (In-Parameter-Order-General), a well known deterministic algorithm in pairwise testing.  There is a [ruby](https://github.com/jesg/dither) implementation and a [java](https://github.com/jesg/dither-java) implementation. The library supports t-way testing, and constraints.  Support for existing tests will be added in the near future after I figure out how to publish the java version on maven central.

Basic Concepts/Usage
-----------

Create a test set with all two way comparisons.

```ruby
require 'dither'

test_set = Dither.ipog([[0, 1, 2, 3],
                        [true, false],
                        [:big, :small],
                        [:cat, :dog, :mouse]])
```

Create a test set with all three way comparisons.

```ruby
require 'dither'

test_set = Dither.ipog([[0, 1, 2, 3],
                        [true, false],
                        [:big, :small],
						[:cat, :dog, :mouse]],
						:t => 3)
```

Create a test set with all three way comparisons.  Exclude :big/:mouse combination.

```ruby
require 'dither'

test_set = Dither.ipog([[0, 1, 2, 3],
                        [true, false],
                        [:big, :small],
                        [:cat, :dog, :mouse]],
                       :t => 3,
                       :constraints => [
                         { 2 => 0, 3 => 2}
                       ])
```


Additional Resources
--------------------
[IPOG](http://csrc.nist.gov/acts/ecbs-cr-final.pdf)

[MIPOG](http://ijcte.org/papers/337-G452.pdf)

[t-way testing strategies](http://ijcte.org/papers/337-G452.pdf)
