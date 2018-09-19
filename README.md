libxmlb
=======

Introduction
------------

XML is slow to parse and strings inside the document cannot be memory mapped as
they do not have a trailing NUL char. The libxmlb library takes XML source, and
converts it to a structured binary representation with a deduplicated string
table -- where the strings have the NULs included.

This allows an application to mmap the binary XML file, do an XPath query and
return some strings without actually parsing the entire document. This is all
done using (almost) zero allocations and no actual copying of the binary data.

As each node in the binary XML file encodes the 'next' node at the same level
it makes skipping whole subtrees trivial. A 10Mb binary XML file can be loaded
from disk **and** queried in less than a few milliseconds.

The binary XML is not supposed to be small. It's usually about half the size of
the text XML data where a lot of the tag content is duplicated, but can actually
be larger than the original XML file. This isn't important; the fast query speed
and the ability to mmap strings without copies more than makes up for the larger
on-disk size. If you want to compress your XML, this library probably isn't for
you -- just use gzip -- its gives you an almost a perfect compression ratio for
data like this.

XPath
-----

This library only implements a tiny subset of XPath. See the examples for the
full list, but it's basically restricted to element_name, attributes and text.
Additionally, the only thing that the query can return are nodes.

For example:

    $ xb-tool compile fedora.xmlb fedora.xml.gz

    $ du -h fedora.xml*
    12M         fedora.xmlb
    3.6M        fedora.xml.gz

    $ xb-tool query fedora.xmlb "components/component[@type=desktop]/id[firefox.desktop]"
    RESULT: firefox.desktop
    real        0m0.011s
    user        0m0.010s
    sys         0m0.001s

TODO
----

* Supporting more of the XPath specification

* Adding XbNode objects to a XbBuilder

* Only store the best language -- not any found in langs