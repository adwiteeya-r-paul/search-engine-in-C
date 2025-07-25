## Design Spec

In this document, the [Requirements Specification](REQUIREMENTS.md) is referenced to later focus on the implementation-independent design decisions.
Here we focus on the core subset:

- User interface
- Inputs and outputs
- Functional decomposition into modules
- Pseudo code (plain English-like language) for logic/algorithmic flow
- Major data structures
- Testing plan

## User interface

As described in the [Requirements Spec](REQUIREMENTS.md), the crawler's only interface with the user is on the command-line; it must always have three arguments.

```bash
$ crawler seedURL pageDirectory maxDepth
```

For example, to crawl one of the CS50 test sites, one needs to store the pages found in a subdirectory `testdirectory1` in the current directory, and to search only depths 0, 1, and 2, use this command line:

``` bash
$ mkdir ../data/letters
$ ./crawler http://cs50tse.cs.dartmouth.edu/tse/letters/index.html ../testdirectory1/letters 2
```

## Inputs and outputs

*Input:* there are no file inputs; there are command-line parameters described above.

*Output:* Per the requirements spec, Crawler will save each explored webpage to a file, one webpage per file, using a unique `documentID` as the file name.  For example,
the top file of the website would have `documentID` 1, the next webpage access from a link on that top page would be `documentID` 2, and so on.
Within each of these files, crawler writes:

 * the full page URL on the first line,
 * the depth of the page (where the `seedURL` is considered to be depth 0) on the second line,
 * the page contents (i.e., the HTML code), beginning on the third line.

## Functional decomposition into modules

There are 5 functions:

 1. *main*, which parses arguments and initializes other modules
 2. *crawler*, which loops over pages to explore, until the list is exhausted
 3. *pagefetcher*, which fetches a page from a URL
 4. *pagescanner*, which extracts URLs from a page and processes each one
 4. *pagesaver*, which outputs a page to the the appropriate file

And some helper modules that provide data structures:

  1. *bag* of pages we have yet to explore
  2. *hashtable* of URLs we've seen so far

## Pseudo code for logic/algorithmic flow

The crawler will run as follows:

    parse the command line, validate parameters, initialize other modules
    add seedURL to the bag of webpages to crawl, marked with depth=0
    add seedURL to the hashtable of URLs seen so far
    while there are more webpages in the bag:
        extract a webpage (URL,depth) item from the bag
        pause for one second
        use pagefetcher to retrieve a webpage for that URL
        use pagesaver to write the webpage to the pageDirectory with a unique document ID
        if the webpage depth is < maxDepth, explore the webpage to find the links it contains:
          use pagescanner to parse the webpage to extract all its embedded URLs
          for each extracted URL:
            normalize the URL (per requirements spec)
            if that URL is internal (per requirements spec):
              try to insert that URL into the *hashtable* of URLs seen;
                if it was already in the table, do nothing;
                if it was added to the table:
                   create a new webpage for that URL, marked with depth+1
                   add that new webpage to the bag of webpages to be crawled


The result may or may not be a Breadth-First Search, but for the crawler we don't care about the order as long as we explore everything within the `maxDepth` neighborhood.

The crawler completes and exits when it has nothing left in its *bag* - no more pages to be crawled.
The maxDepth parameter indirectly determines the number of pages that the crawler will retrieve.


## Major data structures

Helper modules provide all the data structures we need:

- *bag* of webpage (URL, depth) structures
- *hashtable* of URLs
- *webpage* contains all the data read for a given webpage, plus the URL and the depth at which it was fetched

## Testing plan

A sampling of tests that should be run:

1. Testing the program with various forms of incorrect command-line arguments to ensure that its command-line parsing, and validation of those parameters, works correctly.

1. Crawling a simple, closed set of cross-linked web pages like [letters](http://cs50tse.cs.dartmouth.edu/tse/letters/), at depths 0, 1, 2, or more.
Verifying that the files created match expectations.

3. Repeating with a different seed page in that same site.
If the site is indeed a graph, with cycles, there should be several interesting starting points.
