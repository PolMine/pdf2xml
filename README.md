
<!-- README.md is generated from README.Rmd. Please edit that file -->

[![License: GPL
v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Travis-CI Build
Status](https://api.travis-ci.org/PolMine/trickypdf.svg?branch=master)](https://travis-ci.org/PolMine/trickypdf)
[![codecov](https://codecov.io/gh/PolMine/trickypdf/branch/dev/graph/badge.svg)](https://codecov.io/gh/PolMine/trickypdf/branch/master)

# trickypdf

## Purpose

At a basic level, R users have efficient tools to extract text from pdf
documents. The [pdftools](https://CRAN.R-project.org/package=pdftools),
and the [Rpoppler](https://CRAN.R-project.org/package=Rpoppler) packages
are extremely useful and handy.

There are couple of issues that may arise when processing pdf that
remain tricky. The *trickypdf*-package offers a class *PDF* to handle
problems that reoccurringly cause headaches when processing pdf
documents. The class offers methods to…

  - remove stuff outside the main text region (page headers, page
    numbers etc) as a preprocessing step.
  - handle multi-column layouts;
  - reconstruct lines of text, if the (OCRed) document has been scanned
    in tilted fashion;
  - reconstruct paragraphs.

The output will be a valid XML document, with optional document
metadata. XML is generated to serve as the input to a Natural Language
Processing (NLP) pipeline. A method to create browsable html from the
xmlified pdf document is meant to assist quality checking in corpus
preparation. The package thus makes handling tricky problems with
processing pdf documents much easier\!

## Installation

The package relies on the pdftohtml command line utility that is part of
the poppler library. On Linux, installation is easy:

``` sh
sudo apt-get install -y poppler-utils
```

On MacOS, it is recommended to use Homebrew as a package manager. To
install brew, open a terminal and install
Homebrew.

``` sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

The snippet is taken from the [Homebrew website](https://brew.sh). You
can then install poppler as follows,

``` sh
brew install poppler
```

To check that poppler is installed, check the availability of the
pdftohtml command line utility.

``` sh
pdftohtml -v
```

The trickypdf-package imports from a set of standard R packages that are
available from CRAN using `install.packages`.

``` r
install.packages(
  pkgs = c("methods", "xml2", "pbapply", "Rpoppler", "htmltools", "markdown", "stringi", "plyr")
  )
```

Finally, trickypdf can be installed from GitHub using devtools.

``` r
install.packages("devtools") # if you have not yet installed devtools
devtools::install_github("PolMine/trickypdf")
```

## Usage

``` r
library(trickypdf)
```

### Scenario 1: PDF with column titles, page numbers, footnotes

As a sample document, the package includes the United Nations Millenium
Declaration (pdf version). We create an instance of the PDF class by
supplying the
filename.

``` r
doc <- system.file(package = "trickypdf", "extdata", "pdf", "UN_Millenium_Declaration.pdf")
Decl <- PDF$new(filename_pdf = doc)
```

Let us have a look at the document. The *show\_pdf*-method will open a
browser window with the document.

``` r
Decl$show_pdf()
```

For the pages of the document, we define boxes that define the area of
the text that we want to
retain.

``` r
Decl$add_box(page = 1, box = c(left = 110, top = 290, width = 400, height = 415))
Decl$add_box(page = 2, box = c(left = 110, top = 80, width = 400, height = 644))
Decl$add_box(page = 3, box = c(left = 110, top = 80, width = 400, height = 580))
Decl$add_box(page = 4, box = c(left = 110, top = 80, width = 400, height = 570))
Decl$add_box(page = 5, box = c(left = 110, top = 80, width = 400, height = 550))
Decl$add_box(page = 6, box = c(left = 110, top = 80, width = 400, height = 515))
Decl$add_box(page = 7, box = c(left = 110, top = 80, width = 400, height = 550))
Decl$add_box(page = 8, box = c(left = 110, top = 80, width = 400, height = 580))
Decl$add_box(page = 9, box = c(left = 110, top = 80, width = 400, height = 290))
```

We now drop anything outside the boxes (i.e page numbers, headers and
column titles, footnotes). We get the remaining text, do some
postprocessing and xmlify things. By generating a html document, we
perform a quality check.

``` r
Decl$remove_unboxed_text_from_all_pages()
Decl$get_text_from_pages()
Decl$purge()
Decl$xmlify()
Decl$xml2html()
Decl$browse()
```

Looks ok? Let is save the file to disk.

``` r
output <- tempfile(fileext = ".xml")
Decl$write(filename = output)
```

### Scenario 2: PDF with two columns.

The second scenario may appear somewhat more difficult: It is a protocol
of a meeting of the UN General Assembly, a document with a two column
layout.

``` r
doc <- system.file(package = "trickypdf", "extdata", "pdf", "UN_GeneralAssembly_2016.pdf")
UN <- PDF$new(filename_pdf = doc)
UN$show_pdf()
```

As in the previous example, we define boxes. But this time, we define a
second box for each page that does not replace the previously defined
box.

``` r
UN$add_box(page = 1, box = c(top = 380, height = 250, left = 52, width = 255))
UN$add_box(page = 1, box = c(top = 232, height = 400, left = 303, width = 255), replace = FALSE)
UN$add_box(page = 2, box = c(top = 80, height = 595, left = 52, width = 255))
UN$add_box(page = 2, box = c(top = 80, height = 595, left = 303, width = 255), replace = FALSE)
```

In the first scenario, we dropped anything outside the boxes and got
everything that was still on the page. Now we explicitly grab the text
in the boxes.

``` r
UN$get_text_from_boxes(paragraphs = TRUE)
UN$xmlify()
```

Let us inspect the html of the resulting xml.

``` r
UN$xml2html()
UN$browse()
```

Does it look ok? Let’s save it to disk.

``` r
output <- tempfile(fileext = ".xml")
UN$write(filename = output)
```

## Contributing to package development

Please note that this project is released with a [Contributor Code of
Conduct](CONDUCT.md). By participating in this project you agree to
abide by its terms.
