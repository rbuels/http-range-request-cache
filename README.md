# http-range-fetcher

[![NPM version](https://img.shields.io/npm/v/http-range-fetcher.svg?style=flat-square)](https://npmjs.org/package/http-range-fetcher)
[![Build Status](https://img.shields.io/travis/rbuels/http-range-fetcher/master.svg?style=flat-square)](https://travis-ci.org/rbuels/http-range-fetcher) 

Cache/manager for HTTP byte-range requests that merges requests together and caches results.
Designed for applications that request lots of small byte ranges over HTTP.

Uses `cross-fetch` in the backend, so it works either in node or webpack/browserify. Respects
HTTP caching semantics, with the exception of setting a default minimum TTL of 1 second on
requests that are not really supposed to be cached (e.g. `Cache-Control: no-cache`). You can
turn that behavior off by setting `minimumTTL` to 0 though.

## Install

    $ npm install --save http-range-fetcher

## Usage

```js
const { HttpRangeFetcher } = require('http-range-fetcher')

const cache = new HttpRangeFetcher({})
cache.getRange('http://foo.bar/baz.bam', 20, 10)
.then( response => {
  assert(response.buffer.length === 10)
  assert(response.headers['content-range'] === '20-29/23422')
  // response objects contain `headers` and `buffer`.  the `headers` object
  // contains the original headers that came from the server in response to the
  // aggregated call, except the Content-Range header has been overwritten
  // to match the requested range, and it adds a X-Resource-Length header that
  // conveniently gives the total length of the remote resource so you don't
  // have to parse the Content-Range header.
  assert(response.headers['x-resource-length'] === '23422')
})

// these will be aggregated behind the scenes
// as a single request for a big chunk of the remote file,
// which will be cached to satisfy subsequent requests
Promise.all([
    cache.getRange('http://foo.bar/baz.bam', 20, 10),
    cache.getRange('http://foo.bar/baz.bam', 30, 10),
    cache.getRange('http://foo.bar/baz.bam', 40, 10),
    cache.getRange('http://foo.bar/baz.bam', 50, 10),
    cache.getRange('http://foo.bar/baz.bam', 60, 10),
    cache.getRange('http://foo.bar/baz.bam', 70, 10),
])
.then(fetchResults => {
    fetchResults.forEach(res => assert(res.buffer.length === 10))
})
```

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [HttpRangeFetcher](#httprangefetcher)
    -   [getRange](#getrange)
    -   [stat](#stat)
    -   [reset](#reset)

### HttpRangeFetcher

smart cache that fetches chunks of remote files.
caches chunks in an LRU cache, and aggregates upstream fetches

**Parameters**

-   `$0` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `$0.fetch`   (optional, default `crossFetchBinaryRange`)
    -   `$0.size`   (optional, default `10000000`)
    -   `$0.chunkSize`   (optional, default `32768`)
    -   `$0.aggregationTime`   (optional, default `100`)
    -   `$0.minimumTTL`   (optional, default `1000`)
-   `args` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** the arguments object

#### getRange

Fetch a range of a remote resource.

**Parameters**

-   `key` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** the resource's unique identifier, this would usually be a URL.
    This is passed along to the fetch callback.
-   `position` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)?** offset in the file at which to start fetching (optional, default `0`)
-   `length` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)?** number of bytes to fetch, defaults to the remainder of the file

#### stat

Fetches the first few bytes of the remote file (if necessary) and uses
the returned headers to populate a `fs`-like stat object.

Currently, this attempts to set `size`, `mtime`, and `mtimeMs`, if
the information is available from HTTP headers.

**Parameters**

-   `key` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

Returns **[Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)** for a stats object

#### reset

Throw away all cached data, resetting the cache.

## Academic Use

This package was written with funding from the [NHGRI](http://genome.gov) as part of the [JBrowse](http://jbrowse.org) project. If you use it in an academic project that you publish, please cite the most recent JBrowse paper, which will be linked from [jbrowse.org](http://jbrowse.org).

## License

MIT © [Robert Buels](https://github.com/rbuels)
