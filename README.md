Libvisio2svg
============

Library/Utilities to convert Microsoft (MS) Visio Documents and Stencils (VSS and VSD) to SVG.

[![Build Status](https://travis-ci.org/kakwa/libvisio2svg.svg?branch=master)](https://travis-ci.org/kakwa/libvisio2svg)

Motivation
==========

There are tons of publicly available stencils, for example 
[the Cisco ones](http://www.cisco.com/c/en/us/products/visio-stencil-listing.html) 
or the stencils from [VisioCafe](http://www.visiocafe.com/).

This library and utilities were created to be able to convert these stencils to SVG
and reuse them outside of Microsoft Visio, in programs like 
[yEd](https://www.yworks.com/products/yed), [Inkscape](https://inkscape.org), 
[Dia](http://dia-installer.de/), [Calligra Flow](https://www.calligra.org/flow/)...

This library is mainly the glue between [librevenge](http://sourceforge.net/p/libwpd/wiki/librevenge/)/[libvisio](https://github.com/LibreOffice/libvisio) and [libemf2svg](https://github.com/kakwa/libemf2svg).

About libemf2svg
----------------

[libemf2svg](https://github.com/kakwa/libemf2svg) is another library of mine.
It was developed to handle MS EMF (Enhanced Metafile) blobs which constitute most of Visio VSS shapes.

Librevenge/Libvisio would otherwise only bump the EMF blob in an \<image\> SVG tag, which most viewer/editor/browser
would be unable to display.

License
=======

Libvisio2svg is licensed under GPLv2.

Dependencies
============

* [librevenge](http://sourceforge.net/p/libwpd/wiki/librevenge/)
* [libvisio](https://github.com/LibreOffice/libvisio)
* [libemf2svg](https://github.com/kakwa/libemf2svg)
* [libxml2](http://www.xmlsoft.org/)

Building
========

Commands to build this project:

```bash
# CMAKE_INSTALL_PREFIX is optional, default is /usr/local/
$ cmake . -DCMAKE_INSTALL_PREFIX=/usr/

# compilation
$ make

# installation
$ make install
```

Usage
=====

Command Line
------------

Convert VSS:

```bash
# conversion
$ ./vss2svg-conv -i ./tests/resources/vss/2960CX.vss -o ./tests/out/

# help
$ ./vss2svg-conv --help
```

Convert VSD:

```bash
# conversion
$ ./vsd2svg-conv -i ./tests/resources/vss/2960CX.vss -o ./tests/out/

# help
$ ./vsd2svg-conv --help
```

Library
-------

Convert Visio Documents:
```cpp
#include <string>
#include <stdlib.h>
#include <sys/types.h>
#include <visio2svg/Visio2Svg.h>
#include <unordered_map>

// Put visio file in an std::string (not detailed here)
std::string visio_in;

// Initialize converter and output map
visio2svg::Visio2Svg converter;
std::unordered_map<std::string, std::string> out;

// Pick one of the following

// Convert vsd documents
converter.vsd2svg(visio_in, out);

// Convert vss (Stencils) documents
converter.vss2svg(visio_in, out);

// Do something with the output
for (const auto &rule_pair : out) {
    cout << "Sheet Title: " << rule_pair.first <<std::endl;
    cout << rule_pair.second << std::endl;
}
```

This library also comes with a RVNG generator to recover sheets titles:

```cpp
#include <librevenge-stream/librevenge-stream.h>
#include <librevenge-generators/librevenge-generators.h>
#include <librevenge/librevenge.h>
#include <libvisio/libvisio.h>
#include <visio2svg/TitleGenerator.h>

// put the Visio document in an RVNGStream (not detailed here)
librevenge::RVNGStringStream input;
/* or from file
librevenge::RVNGFileStream input;
*/

// Recover Titles of each sheets
librevenge::RVNGStringVector output_names;
visio2svg::TitleGenerator generator_names(output_names);

ret = libvisio::VisioDocument::parseStencils(&input, &generator_names);
/* or this for vsd
ret = libvisio::VisioDocument::parse(&input, &generator_names);
*/

if (!ret || output_names.empty()) {
    std::cerr << "ERROR: Failed to recover sheets titles failed!"
              << std::endl;
    return 1;
}

// do something with the names
for (unsigned k = 0; k < output_names.size(); ++k) {
    std::cout << output_names[k].cstr() << std::endl;
}
```
