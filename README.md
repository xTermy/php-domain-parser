PHP Domain Parser 
=================

**PHP Domain Parser** is a [Public Suffix List](http://publicsuffix.org/) based 
domain parser implemented in PHP.

Motivation
----------

While there are plenty of excellent URL parsers and builders available, there
are very few projects that can accurately parse a url into its component 
subdomain, registerable domain, and public suffix parts. 

Consider the domain www.pref.okinawa.jp.  In this domain, the
*public suffix* portion is **okinawa.jp**, the *registerable domain* is
**pref.okinawa.jp**, and the *subdomain* is **www**. You can't regex that.

Other similar libraries focus primarily on URL building, parsing, and manipulation and 
additionally include public suffix domain parsing.  PHP Domain Parser was built around 
accurate Public Suffix List based parsing from the very beginning, adding a URL 
object simply for the sake of completeness.

Installation
------------

The only (currently) supported method of installation is via
[Composer](http://getcomposer.org).

Create a `composer.json` file in the root of your project:

``` json
{
    "require": {
        "stormcode/php-domain-parser": "~1.0"
    }
}
```

And then run: `composer install`

Add the autoloader to your project:

``` php
<?php

require_once 'vendor/autoload.php'
```

You're now ready to begin using the PHP Domain Parser.

Usage
-----

### Parsing URLs ###

Parsing URLs into their component parts is as simple as the example you see below.

``` php
<?php

require_once '../vendor/autoload.php';

$pslManager = new Pdp\PublicSuffixListManager();
$parser = new Pdp\Parser($pslManager->getList());
$host = 'http://user:pass@www.pref.okinawa.jp:8080/path/to/page.html?query=string#fragment';
$url = $parser->parseUrl($host);
var_dump($url);
```

The above will output:

```
class Pdp\Uri\Url#6 (8) {
    private $scheme =>
    string(4) "http"
    private $host =>
    class Pdp\Uri\Url\Host#5 (3) {
        private $subdomain =>
        string(3) "www"
        private $registerableDomain =>
        string(15) "pref.okinawa.jp"
        private $publicSuffix =>
        string(10) "okinawa.jp"
    }
    private $port =>
    int(8080)
    private $user =>
    string(4) "user"
    private $pass =>
    string(4) "pass"
    private $path =>
    string(18) "/path/to/page.html"
    private $query =>
    string(12) "query=string"
    private $fragment =>
    string(8) "fragment"
}
```

### Convenience Methods

A magic __get() method is provided to access the above object properties.  Obtaining the
public suffix for a parsed domain is as simple as:

``` php
<?php

$host = 'waxaudio.com.au';
$url = $parser->parseUrl($host);
$publicSuffix = $url->host->publicSuffix;

// $publicSuffix = 'com.au'
```

### IDNA Support

[IDN (Internationalized Domain Name)](http://en.wikipedia.org/wiki/Internationalized_domain_name) 
support was added in version `1.4.0`. Both unicode domains and their ASCII equivalents
are supported.

**IMPORTANT**: PHP's [intl](http://php.net/manual/en/book.intl.php) extension is
required for the [IDN functions](http://php.net/manual/en/ref.intl.idn.php).

#### Unicode

Parsing IDNA hosts is no different that parsing standard hosts. Setting `$host = '????????????.????';` (Russian-Cyrillic)
in the *Parsing URLs* example would return:

```
class Pdp\Uri\Url#6 (8) {
  private $scheme =>
  string(4) "http"
  private $host =>
  class Pdp\Uri\Url\Host#5 (4) {
    private $subdomain =>
    NULL
    private $registerableDomain =>
    string(17) "????????????.????"
    private $publicSuffix =>
    string(4) "????"
    private $host =>
    string(17) "????????????.????"
  }
  private $port =>
  NULL
  private $user =>
  NULL
  private $pass =>
  NULL
  private $path =>
  NULL
  private $query =>
  NULL
  private $fragment =>
  NULL
}
```

#### ASCII (Punycode)

If you choose to provide the ASCII equivalent of the unicode domain name
(`$host = 'http://xn--d1acpjx3f.xn--p1ai';` in the case of the *Parsing URLs* example),
the ASCII equivalent will be returned by the parser:

```
class Pdp\Uri\Url#6 (8) {
  private $scheme =>
  string(4) "http"
  private $host =>
  class Pdp\Uri\Url\Host#5 (4) {
    private $subdomain =>
    NULL
    private $registerableDomain =>
    string(22) "xn--d1acpjx3f.xn--p1ai"
    private $publicSuffix =>
    string(8) "xn--p1ai"
    private $host =>
    string(22) "xn--d1acpjx3f.xn--p1ai"
  }
  private $port =>
  NULL
  private $user =>
  NULL
  private $pass =>
  NULL
  private $path =>
  NULL
  private $query =>
  NULL
  private $fragment =>
  NULL
}
```

### IPv6 Support

Parsing IPv6 hosts is no different that parsing standard hosts. Setting `$host = 'http://[2001:db8:85a3:8d3:1319:8a2e:370:7348]:8080/';`
in the *Parsing URLs* example would return:

```
class Pdp\Uri\Url#6 (8) {
  private $scheme =>
  string(4) "http"
  private $host =>
  class Pdp\Uri\Url\Host#5 (4) {
    private $subdomain =>
    NULL
    private $registerableDomain =>
    NULL
    private $publicSuffix =>
    NULL
    private $host =>
    string(38) "[2001:db8:85a3:8d3:1319:8a2e:370:7348]"
  }
  private $port =>
  string(4) "8080"
  private $user =>
  NULL
  private $pass =>
  NULL
  private $path =>
  string(1) "/"
  private $query =>
  NULL
  private $fragment =>
  NULL
}
```

**IMPORTANT**: IPv6 url host names *must* be enclosed in square brackets. They
will not be parsed properly otherwise.

> Hat tip to [@geekwright](https://github.com/geekwright) for adding IPv6 support in a 
> [bugfix pull request](https://github.com/jeremykendall/php-domain-parser/pull/35).

### Parsing Domains ###

If you'd like to parse the domain (or host) portion only, you can use 
`Parser::parseHost()`.

```php
<?php

$host = $parser->parseHost('a.b.c.cy');
var_dump($host);
```

The above will output:

```
class Pdp\Uri\Url\Host#7 (3) {
    private $subdomain =>
    string(1) "a"
    private $registerableDomain =>
    string(6) "b.c.cy"
    private $publicSuffix =>
    string(4) "c.cy"
}
```

### Retrieving Domain Components Only ###

If you're only interested in a domain component, you can use the parser to
retrieve only the component you're interested in

```php
<?php

var_dump($parser->getSubdomain('www.scottwills.co.uk'));
var_dump($parser->getRegisterableDomain('www.scottwills.co.uk'));
var_dump($parser->getPublicSuffix('www.scottwills.co.uk'));
```

The above will output:

```
string(3) "www"
string(16) "scottwills.co.uk"
string(5) "co.uk"
```

### Sanity Check ###

You can quickly parse a url from the command line with the provided `parse` 
vendor binary.  From the root of your project, simply call:

``` bash
$ ./vendor/bin/parse <url>
```

If you pass a url to `parse`, that url will be parsed and the output printed
to screen.  

If you do not pass a url, `http://user:pass@www.pref.okinawa.jp:8080/path/to/page.html?query=string#fragment` will be parsed and the output printed to screen.

Example:

``` bash
$ ./vendor/bin/parse http://www.waxaudio.com.au/

Array
(
    [scheme] => http
    [user] =>
    [pass] =>
    [host] => www.waxaudio.com.au
    [subdomain] => www
    [registerableDomain] => waxaudio.com.au
    [publicSuffix] => com.au
    [port] =>
    [path] => /
    [query] =>
    [fragment] =>
)
Host: http://www.waxaudio.com.au/
```

### Example Script ###

For more information on using the PHP Domain Parser, please see the provided
[example script](https://github.com/jeremykendall/php-domain-parser/blob/master/example.php).

### Refreshing the Public Suffix List ###

While a cached PHP copy of the Public Suffix List is provided for you in the
`data` directory, that copy may or may not be up to date (Mozilla provides an
[Atom change feed](http://hg.mozilla.org/mozilla-central/atom-log/default/netwerk/dns/effective_tld_names.dat)
to keep up with changes to the list). Please use the provided vendor binary to
refresh your cached copy of the Public Suffix List.

From the root of your project, simply call:

``` bash
$ ./vendor/bin/pdp-psl
```

You may verify the update by checking the timestamp on the files located in the
`data` directory.

**Important**: The vendor binary `pdp-psl` depends on an internet connection to
update the cached Public Suffix List.

Contributing
------------

Pull requests are *always* welcome! Please review the CONTRIBUTING.md document before
submitting pull requests.

Attribution
-----------

The HTTP adapter interface and the cURL HTTP adapter were inspired by (er,
lifted from) Will Durand's excellent
[Geocoder](https://github.com/willdurand/Geocoder) project.  His MIT license and
copyright notice are below.

```
Copyright (c) 2011-2013 William Durand <william.durand1@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is furnished
to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```

Portions of the PublicSuffixListManager and the DomainParser are derivative
works of the PHP
[registered-domain-libs](https://github.com/usrflo/registered-domain-libs).
Those parts of this codebase are heavily commented, and I've included a copy of
the Apache Software Foundation License 2.0 in this project.
