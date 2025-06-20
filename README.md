[![Build status badge][]][build status]



intervaltree
============

A mutable, self-balancing interval tree for Python 2 and 3. Queries may be by point, by range overlap, or by range envelopment.

This library was designed to allow tagging text and time intervals, where the intervals include the lower bound but not the upper bound.

**Version 3 changes!**

* The `search(begin, end, strict)` method no longer exists. Instead, use one of these:
    * `at(point)`
    * `overlap(begin, end)`
    * `envelop(begin, end)`
* The `extend(items)` method no longer exists. Instead, use `update(items)`.
* Methods like `merge_overlaps()` which took a `strict` argument consistently default to `strict=True`. Before, some methods defaulted to `True` and others to `False`.

Installing (UV)
---------------

Add inside uv `pyproject.toml`
```
dependencies = [
    "intervaltree",
]

[tool.uv.sources]
intervaltree = { git = "https://github.com/DREU007/intervaltree-uv.git" }
```

```sh
uv sync
```

Use imports as usual
```python
from intervaltree import Interval, IntervalTree
```

Features
--------

* Supports Python 2.7 and Python 3.6+ (Tested under 2.7, and 3.6 thru 3.11)
* Initializing
    * blank `tree = IntervalTree()`
    * from an iterable of `Interval` objects (`tree = IntervalTree(intervals)`)
    * from an iterable of tuples (`tree = IntervalTree.from_tuples(interval_tuples)`)

* Insertions
    * `tree[begin:end] = data`
    * `tree.add(interval)`
    * `tree.addi(begin, end, data)`

* Deletions
    * `tree.remove(interval)`             (raises `ValueError` if not present)
    * `tree.discard(interval)`            (quiet if not present)
    * `tree.removei(begin, end, data)`    (short for `tree.remove(Interval(begin, end, data))`)
    * `tree.discardi(begin, end, data)`   (short for `tree.discard(Interval(begin, end, data))`)
    * `tree.remove_overlap(point)`
    * `tree.remove_overlap(begin, end)`   (removes all overlapping the range)
    * `tree.remove_envelop(begin, end)`   (removes all enveloped in the range)

* Point queries
    * `tree[point]`
    * `tree.at(point)`                    (same as previous)

* Overlap queries
    * `tree[begin:end]`
    * `tree.overlap(begin, end)`          (same as previous)

* Envelop queries
    * `tree.envelop(begin, end)`

* Membership queries
    * `interval_obj in tree`              (this is fastest, O(1))
    * `tree.containsi(begin, end, data)`
    * `tree.overlaps(point)`
    * `tree.overlaps(begin, end)`

* Iterable
    * `for interval_obj in tree:`
    * `tree.items()`

* Sizing
    * `len(tree)`
    * `tree.is_empty()`
    * `not tree`
    * `tree.begin()`          (the `begin` coordinate of the leftmost interval)
    * `tree.end()`            (the `end` coordinate of the rightmost interval)

* Set-like operations
    * union
        * `result_tree = tree.union(iterable)`
        * `result_tree = tree1 | tree2`
        * `tree.update(iterable)`
        * `tree |= other_tree`

    * difference
        * `result_tree = tree.difference(iterable)`
        * `result_tree = tree1 - tree2`
        * `tree.difference_update(iterable)`
        * `tree -= other_tree`

    * intersection
        * `result_tree = tree.intersection(iterable)`
        * `result_tree = tree1 & tree2`
        * `tree.intersection_update(iterable)`
        * `tree &= other_tree`

    * symmetric difference
        * `result_tree = tree.symmetric_difference(iterable)`
        * `result_tree = tree1 ^ tree2`
        * `tree.symmetric_difference_update(iterable)`
        * `tree ^= other_tree`

    * comparison
        * `tree1.issubset(tree2)` or `tree1 <= tree2`
        * `tree1 <= tree2`
        * `tree1.issuperset(tree2)` or `tree1 > tree2`
        * `tree1 >= tree2`
        * `tree1 == tree2`

* Restructuring
    * `chop(begin, end)`      (slice intervals and remove everything between `begin` and `end`, optionally modifying the data fields of the chopped-up intervals)
    * `slice(point)`          (slice intervals at `point`)
    * `split_overlaps()`      (slice at all interval boundaries, optionally modifying the data field)
    * `merge_overlaps()` (joins overlapping intervals into a single interval, optionally merging the data fields)
    * `merge_equals()` (joins intervals with matching ranges into a single interval, optionally merging the data fields)
    * `merge_neighbors()` (joins adjacent intervals into a single interval if the distance between their range terminals is less than or equal to a given distance. Optionally merges overlapping intervals. Can also merge the data fields.)

* Copying and typecasting
    * `IntervalTree(tree)`    (`Interval` objects are same as those in tree)
    * `tree.copy()`           (`Interval` objects are shallow copies of those in tree)
    * `set(tree)`             (can later be fed into `IntervalTree()`)
    * `list(tree)`            (ditto)

* Pickle-friendly
* Automatic AVL balancing

Examples
--------

* Getting started

    ``` python
    >>> from intervaltree import Interval, IntervalTree
    >>> t = IntervalTree()
    >>> t
    IntervalTree()

    ```

* Adding intervals - any object works!

    ``` python
    >>> t[1:2] = "1-2"
    >>> t[4:7] = (4, 7)
    >>> t[5:9] = {5: 9}

    ```

* Query by point

    The result of a query is a `set` object, so if ordering is important,
    you must sort it first.

    ``` python
    >>> sorted(t[6])
    [Interval(4, 7, (4, 7)), Interval(5, 9, {5: 9})]
    >>> sorted(t[6])[0]
    Interval(4, 7, (4, 7))

    ```

* Query by range

    Note that ranges are inclusive of the lower limit, but non-inclusive of the upper limit. So:

    ``` python
    >>> sorted(t[2:4])
    []

    ```

    Since our search was over `2 ≤ x < 4`, neither `Interval(1, 2)` nor `Interval(4, 7)`
    was included. The first interval, `1 ≤ x < 2` does not include `x = 2`. The second
    interval, `4 ≤ x < 7`, does include `x = 4`, but our search interval excludes it. So,
    there were no overlapping intervals. However:

    ``` python
    >>> sorted(t[1:5])
    [Interval(1, 2, '1-2'), Interval(4, 7, (4, 7))]

    ```

    To only return intervals that are completely enveloped by the search range:

    ``` python
    >>> sorted(t.envelop(1, 5))
    [Interval(1, 2, '1-2')]

    ```

* Accessing an `Interval` object

    ``` python
    >>> iv = Interval(4, 7, (4, 7))
    >>> iv.begin
    4
    >>> iv.end
    7
    >>> iv.data
    (4, 7)

    >>> begin, end, data = iv
    >>> begin
    4
    >>> end
    7
    >>> data
    (4, 7)

    ```

* Constructing from lists of intervals

    We could have made a similar tree this way:

    ``` python
    >>> ivs = [(1, 2), (4, 7), (5, 9)]
    >>> t = IntervalTree(
    ...    Interval(begin, end, "%d-%d" % (begin, end)) for begin, end in ivs
    ... )

    ```

    Or, if we don't need the data fields:

    ``` python
    >>> t2 = IntervalTree(Interval(*iv) for iv in ivs)

    ```

    Or even:

    ``` python
    >>> t2 = IntervalTree.from_tuples(ivs)

    ```

* Removing intervals

    ``` python
    >>> t.remove(Interval(1, 2, "1-2"))
    >>> sorted(t)
    [Interval(4, 7, '4-7'), Interval(5, 9, '5-9')]

    >>> t.remove(Interval(500, 1000, "Doesn't exist"))  # raises ValueError
    Traceback (most recent call last):
    ValueError

    >>> t.discard(Interval(500, 1000, "Doesn't exist"))  # quietly does nothing

    >>> del t[5]  # same as t.remove_overlap(5)
    >>> t
    IntervalTree()

    ```

    We could also empty a tree entirely:

    ``` python
    >>> t2.clear()
    >>> t2
    IntervalTree()

    ```

    Or remove intervals that overlap a range:

    ``` python
    >>> t = IntervalTree([
    ...     Interval(0, 10),
    ...     Interval(10, 20),
    ...     Interval(20, 30),
    ...     Interval(30, 40)])
    >>> t.remove_overlap(25, 35)
    >>> sorted(t)
    [Interval(0, 10), Interval(10, 20)]

    ```

    We can also remove only those intervals completely enveloped in a range:

    ``` python
    >>> t.remove_envelop(5, 20)
    >>> sorted(t)
    [Interval(0, 10)]

    ```

* Chopping

    We could also chop out parts of the tree:

    ``` python
    >>> t = IntervalTree([Interval(0, 10)])
    >>> t.chop(3, 7)
    >>> sorted(t)
    [Interval(0, 3), Interval(7, 10)]

    ```

    To modify the new intervals' data fields based on which side of the interval is being chopped:

    ``` python
    >>> def datafunc(iv, islower):
    ...     oldlimit = iv[islower]
    ...     return "oldlimit: {0}, islower: {1}".format(oldlimit, islower)
    >>> t = IntervalTree([Interval(0, 10)])
    >>> t.chop(3, 7, datafunc)
    >>> sorted(t)[0]
    Interval(0, 3, 'oldlimit: 10, islower: True')
    >>> sorted(t)[1]
    Interval(7, 10, 'oldlimit: 0, islower: False')

    ```

* Slicing

    You can also slice intervals in the tree without removing them:

    ``` python
    >>> t = IntervalTree([Interval(0, 10), Interval(5, 15)])
    >>> t.slice(3)
    >>> sorted(t)
    [Interval(0, 3), Interval(3, 10), Interval(5, 15)]

    ```

    You can also set the data fields, for example, re-using `datafunc()` from above:

    ``` python
    >>> t = IntervalTree([Interval(5, 15)])
    >>> t.slice(10, datafunc)
    >>> sorted(t)[0]
    Interval(5, 10, 'oldlimit: 15, islower: True')
    >>> sorted(t)[1]
    Interval(10, 15, 'oldlimit: 5, islower: False')

    ```

Future improvements
-------------------

See the [issue tracker][] on GitHub.

Based on
--------

* Eternally Confuzzled's [AVL tree][Confuzzled AVL tree]
* Wikipedia's [Interval Tree][Wiki intervaltree]
* Heavily modified from Tyler Kahn's [Interval Tree implementation in Python][Kahn intervaltree] ([GitHub project][Kahn intervaltree GH])
* Incorporates contributions from:
    * [konstantint/Konstantin Tretyakov][Konstantin intervaltree] of the University of Tartu (Estonia)
    * [siniG/Avi Gabay][siniG intervaltree]
    * [lmcarril/Luis M. Carril][lmcarril intervaltree] of the Karlsruhe Institute for Technology (Germany)
    * [depristo/MarkDePristo][depristo intervaltree]

Copyright
---------

* [Chaim Leib Halbert][GH], 2013-2023
* Modifications, [Konstantin Tretyakov][Konstantin intervaltree], 2014

Licensed under the [Apache License, version 2.0][Apache].

The source code for this project is at https://github.com/chaimleib/intervaltree


[build status badge]: https://github.com/chaimleib/intervaltree/workflows/ci/badge.svg
[build status]: https://github.com/chaimleib/intervaltree/actions
[GH]: https://github.com/chaimleib/intervaltree
[issue tracker]: https://github.com/chaimleib/intervaltree/issues
[Konstantin intervaltree]: https://github.com/konstantint/PyIntervalTree
[siniG intervaltree]: https://github.com/siniG/intervaltree
[lmcarril intervaltree]: https://github.com/lmcarril/intervaltree
[depristo intervaltree]: https://github.com/depristo/intervaltree
[Confuzzled AVL tree]: http://www.eternallyconfuzzled.com/tuts/datastructures/jsw_tut_avl.aspx
[Wiki intervaltree]: http://en.wikipedia.org/wiki/Interval_tree
[Kahn intervaltree]: http://zurb.com/forrst/posts/Interval_Tree_implementation_in_python-e0K
[Kahn intervaltree GH]: https://github.com/tylerkahn/intervaltree-python
[Apache]: http://www.apache.org/licenses/LICENSE-2.0
