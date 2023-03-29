# chklnks.rb
Website link crawler and checker written in Ruby.

* Around 250 lines of Ruby
* Requires no gems - uses only Ruby core/std libraries
* Recursively crawls "internal" links
* Checks status of "external" links
* Follows (one level of) redirection
* Tracks all page origin(s) of each unique link
* Generates HTML report

Each line of the report is a unique link in your site in queue order:

![output report screenshot](screenshot1.png?raw=true)

You can click to expand full details and headers from each link check:

![output detail expanded screenshot](screenshot2.png?raw=true)

## Usage

`chklnks` writes an HTML report to STDOUT and progress messages to STDERR.
You'll almost certainly want to redirect the report to a file:

    ./chklnks 'http://example.com/' > report.html

While it runs, you'll be able to see queue progress. It may take a while, but
at least it's fun to watch the queue grow and then shrink.

When it's done, open the report in a browser and enjoy.

## Limitations

chklnks assumes that all links with just a path (e.g. `foo/bar/baz.html`)
are "internal" to the site and all links with a protocol and host 
(e.g. `http://example.com/...`) are external.  This is true of my website, but
may not be true of yours.

Redirects (HTTP 301 and HTTP 302) are only followed to one level.  Anything
other than an HTTP 200 OK after the first redirect is marked as an "error" even
if it's fine.  The rationale is that if you're redirected more than once, you
should probably update your link, right?

_All_ link checks begin with a HEAD request.  "External" links are _only_
checked with a HEAD request. I'm starting to see servers which respond to HEAD
requests with a HTTP 405 Method Not Allowed. I personally think those servers
aren't being good citizens.

HTML `<base>` tags are not checked.  The base href defines a URI base for
relative links on the page.  I don't use this tag on my websites, so I didn't
bother to implement this.  I'll happily accept a pull request with the
addition.

## How it works

Starting with the first page, chklnks scans the page for anchor tags (`<a href="...">...</a>`).

Each link is parsed as a normalized Ruby `URI` and merged with the page's URI.

The string form of the normalized URI is used as a key to store in a new `Link`
(created as a `Struct`) in a `Hash` to ensure exactly *one* copy of each unique
link is stored.

Unique Links are put in a `Queue` to guarantee that each link is visited
exactly one time.

An HTTP HEAD request is performed on each link and the resulting status code
and headers are stored for the link.  If the result is a redirect, the
redirected URI is attempted immediately (with another HEAD request) and the
redirected result is also stored.

If the link is considered "internal" to the site (and appears to be `text/html`
content), it is scanned for more links and the circle of life continues.

## Contributing

Every website is a unique snowflake and you may wish to simply fork this repo
as a simple base for _your_ link checker.  On the other hand, suggestions will
be considered and improvements are very welcome.
