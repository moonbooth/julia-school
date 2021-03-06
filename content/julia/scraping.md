---
title: Scraping web pages with Julia and the HTTP and Gumbo packages
date: 2020-08-05 12:31:32+11:00
seoTitle: "Scraping web pages with Julia HTTP & Gumbo: Tutorial"
description: Julia can be used for fast web scraping, not just data analysis.
authors: ["Ron Erdos"]
tableOfContents: true
version: 1.4.2
---

Do you want to crawl and scrape web pages with the Julia language? This tutorial will show you how.

## Ethical and legal web scraping

Before we begin, it goes without saying that you should only scrape web pages where you are not infringing any laws or rules by doing so. See the box below for a few websites you can scrape legally and ethically.

<aside>

### Which websites can I legally practise web scraping on?

You can practise web scraping legally on `example.com` (or any of its clones, `example.net`, `example.org` and `example.edu`). These simple websites are published by the Internet Corporation for Assigned Names and Numbers (ICANN) especially for documentation and testing purposes.

If you want **more complex websites to scrape legally**, try `toscrape.com`, which provides a list of quotes at `quotes.toscrape.com` and a dummy bookstore at `books.toscrape.com`. These websites are published by Scraping Hub for scraping practice.

</aside>

## Introducing two Julia web scraping packages: HTTP.jl and Gumbo.jl

We're going to use two packages. Here's what they each do:

- `HTTP.jl`: Scrapes web pages
- `Gumbo.jl`: Parses these web pages after we've scraped them with `HTTP.jl`, so that we can more easily retrieve specific elements (such as an `<h1>` heading).

There is a third Julia package named `Cascadia.jl` which allows you to scrape HTML elements by CSS class or id, but that's out of scope for this article (for now).

### Downloading and installing the packages

First, if you don't have Julia itself installed, here's [how to install Julia on a Mac](../setup/).)

OK, once you have Julia installed, fire up your terminal of choice, and enter the Julia REPL by typing `julia` at your command prompt.

You should see the green Julia command prompt:

`julia`

Now it's time to add the packages.

My favourite way to do this is to type the `]` (right square bracket, located above the Return / Enter key on your keyboard) into the Julia terminal.

This changes the green prompt we saw above into a purple one that says:

<code class="julia-pkg">pkg></code>

The three letters above stand for "package". We're now in Julia's package manager.

We can now easily add the three web scraping and parsing packages we need, using the `add` command:

`add HTTP Gumbo`

Note that we separate the package names with spaces, rather than commas.

Hit Enter and let Julia do its thing.

Once the packages have been installed, you can exit out of Julia's package manager by pressing Delete / Backspace. You'll then see the green prompt again:

`julia`

### "Using" the packages

Before we do any actual web scraping, we need to tell Julia we intend to use all three of our newly installed packages:

`using HTTP, Gumbo`

Unlike when we `add`ed the packages, we need to use commas to separate the package names in a `using` command.

## Scraping example.com

We're going to scrape the homepage of `example.com` in this tutorial.

We'll store it in a new variable we'll create, called `r`.

We'll just use the `HTTP` package for now---we'll use the others later.

We enter our command into the Julia REPL:

`r = HTTP.get("https://example.com/")`

<aside>

### We need to use the full protocol in urls

Note that we need to include the full protocol (e.g. `https://`) at the start of our target url, as shown just above this box. If we just do this:

`r = HTTP.get("example.com")`

... we get this error:

`ERROR: DNSError: , unknown node or service (EAI_NONAME)`

</aside>

### The output

Here's the output. We'll walk through it below.

```
HTTP.Messages.Response:
"""
HTTP/1.1 200 OK
Age: 433583
Cache-Control: max-age=604800
Content-Type: text/html; charset=UTF-8
Date: Sun, 02 Aug 2020 02:37:09 GMT
Etag: "3147526947+ident"
Expires: Sun, 09 Aug 2020 02:37:09 GMT
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Server: ECS (sjc/4E8D)
Vary: Accept-Encoding
X-Cache: HIT
Content-Length: 1256

<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;

    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domai
⋮
1256-byte body
"""
```

### Output walkthrough

Julia truncates the output, but we can see, in descending order:

1) the **protocol** and **status code** (`HTTP/1.1 200 OK`). `HTTP/1.1` is the protocol and `200` is the status code. A code of `200` means the page has loaded correctly, which is why we also see the word `OK`.
2) the **headers**, which in this case consists of eleven key-value pairs from `Age: 433583` through to `Content-Length: 1256`
3) the first 1000 or so characters of the **source code** of the actual HTML document. In this case, we see `<!doctype html>` through to just after `<h1>Example Domain</h1>`. Note that the full web page will be stored in our variable `r`; it's just the output in the Julia REPL that's truncated.

Later on in this tutorial, we'll explore techniques for laser-targeting _just_ the status code or _just_ the headers (see the table of contents at the top for links to these), but for now, let's continue exploring how we can target individual HTML elements in the source code.

## Enter Gumbo.jl

`Gumbo.jl` is a Julia package that enables us to transform the relatively amorphous blob of HTML we crawled with `HTTP.jl` into a parseable HTML tree. This means we can fairly easily zero in on particular HTML elements, such as the `<h1>` heading. We'll do just that below.

<aside>

### "Using" Gumbo.jl

Earlier in this tutorial, we told Julia that we want to be `using Gumbo` (as part of a multi-package `using HTTP, Gumbo` command), so we don't need to do that again here.

If you skipped entering that command into your Julia REPL earlier, you should tell Julia that you want to be `using Gumbo`.

</aside>

### Turning our blob of HTML into a parseable HTML tree with Gumbo

Earlier, we stored our crawl of the homepage of `example.com` into a variable we called `r`.

Now we'll make a "Gumbo" version---which will be easily traversable---of that `r` variable, and we'll call it `r_parsed`.

The way to do that is below:

`r_parsed = parsehtml(String(r.body))`

### The output from the Gumbo command

```
HTML Document:
<!DOCTYPE html>
HTMLElement{:HTML}:<HTML>
  <head>
    <title>
      Example Domain
    </title>
    <meta charset="utf-8"/>
    <meta content="text/html; charset=utf-8" http-equiv="Content-type"/>
    <meta content="width=device-width, initial-scale=1" name="viewport"/>
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;

    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
...
```

This output is also truncated, but we're now ready to start digging down into our parsed HTML tree. We'll do that right now.

### Digging down into Gumbo's parsed HTML tree

What if you want to see _just_ the `<head>` section of this source code, the source code of `example.com`?

Well, we need to add `.root` to `r_parsed` to start working with it. So we'll be using `r_parsed.root`. And inside that, there are two main section, the `<head>` (available at `r_parsed.root[1]`), and the `<body>` (at `r_parsed.root[2]`).

Since we want the `<head>` section to start with, we'll use this command:

`head = r_parsed.root[1]`:

```
HTMLElement{:head}:<head>
  <title>
    Example Domain
  </title>
  <meta charset="utf-8"/>
  <meta content="text/html; charset=utf-8" http-equiv="Content-type"/>
  <meta content="width=device-width, initial-scale=1" name="viewport"/>
  <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;

    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
...
```

Now, if we want _just_ the `<body>` section, we'll use _this_ command:

`body = r_parsed.root[2]`

... and we see this:

```
HTMLElement{:body}:<body>
  <div>
    <h1>
      Example Domain
    </h1>
    <p>
      This domain is for use in illustrative examples in documents. You may use this
      domain in literature without prior coordination or asking for permission.
    </p>
    <p>
      <a href="https://www.iana.org/domains/example">
        More information...
      </a>
    </p>
  </div>
</body>
```

### Getting the text of the h1 heading

OK, so how do we get the text of the `<h1>` heading?

Well, immediately inside the `<body>`, we have a `<div>`, and then immediately inside that, we have our `<h1>`. So that's the first element inside the first element inside the `<body>`.

So we'll do this:

`h1 = body[1][1]`

... and we get this:

```
HTMLElement{:h1}:<h1>
  Example Domain
</h1>
```

OK, so how do we get just the text of this `<h1>`?

Well, if we enter this command:

`h1[1].text`

... we get this:

`"Example Domain"`

Booyah!

## Getting just the status code of web pages

Let's do this with an example.

By the way, if you're not sure what a status code is, read the box below.

<aside>

### What is a web page status code?

When you request a web page in your browser, the server will send your browser a three-digit code to tell it how that request went. For example, a code of `200` means the page loaded as requested. You beauty!

A code of `404` means the page couldn't be found.

And a code of `301` means the page has been permanently redirected to another page.

There are many other status codes; these are just three examples.

</aside>

<aside>

### "Using" the HTTP package

If you haven't already done so, you need to tell Julia that you'll be `using` the `HTTP` package (you won't need the `Gumbo` package to get the status codes).

To do so, at the green `julia` prompt, enter the following command:

`using HTTP`

Julia will take a second or two to action this.

</aside>

### Getting the status code of a working page (status code 200)

Let's say we want to get the status code of `https://example.com/`. This is a working page, which therefore has the status code of `200`.

To do this, we'll have Julia visit the url above and assign it to the variable `example_com`:

`example_com = HTTP.head("https://example.com/")`

<aside>

### HTTP.get() vs HTTP.head() in the Julia HTTP.jl package

Note that we aren't using `HTTP.get()` to pull the status code, although we could. We're using `HTTP.head()` because it only pulls the headers (which include the status code). We're saving bandwidth (ours and that of the web server) by not pulling the whole web page when we only need the headers.

</aside>

Now we can trivially get the status code:

`example_com.status`

You should see the following output:

`200`

As explained in the box above, `200` is the status code that indicates the page has loaded correctly.

### Getting the status code of a 404 page

If we want to get the status code of a 404 page, then we need to add a bit more to our code.

Let's say we want the status code of `https://example.com/404`, which has the status code of `404`, which means the page could not be found.

If we just try this:

`example_com_404 = HTTP.head("https://example.com/404")`

... then we get this error:

`ERROR: HTTP.ExceptionRequest.StatusError`

The way to avoid this error is by adding the argument `status_exception=false` to our command:

`example_com_404 = HTTP.head("https://example.com/404", status_exception=false)`

Now we can get the status code:

`example_com_404.status`

... and we see the correct result in our terminal:

`404`

### Getting the status code of a 500 page

When pages cannot be loaded due to a server error, there's a good chance it has the status code of `500`.

For example, this page has a (deliberate) `500` error:

`http://getstatuscode.com/500`

Similarly to the `404` example above, we need to use the `status_exception=false` argument in our code:

`getstatuscode_500 = HTTP.head("http://getstatuscode.com/500", status_exception=false)`

... so that we can ask for the status code:

`getstatuscode_500.status`

... and get the right answer:

`500`

By contrast, if we leave out the `status_exception=false` bit:

`getstatuscode_500 = HTTP.head("http://getstatuscode.com/500")`

... then we get the same sort of error we saw earlier in our 404 example:

`ERROR: HTTP.ExceptionRequest.StatusError`

### Getting the status code of a 301 redirect

If you want to know which pages on a website have been 301 redirected, this also requires a _different_ bit of additional code.

For example, let's say you want the status code for `http://balloon.com/`. That page 301 redirects to `https://LABalloons.com/`, which appears to be a Los Angeles-based balloon supply company.

However, if we simply run something like this:

`balloon_com = HTTP.head("http://balloon.com/")`

... and then ask for the status code:

`balloon_com.status`

... we end up with this:

`200`

... which may not be what we want. So what's going on here? The reason we see a status code of `200` is because Julia is showing us the _eventual_ status code, i.e. the status code of the final page, `https://LABalloons.com/`.

However, if we want to see that a 301 redirect occurred, we need to add the argument `redirect=false`:

`balloon_com = HTTP.head("http://balloon.com/", redirect=false)`

Now we can ask to see the status:

`balloon_com.status`

... and we see that a 301 redirect has occurred:

`301`

### Getting the status code of a 302 redirect

The process is very similar for a 302 redirect (a temporary redirect).

Let's take a look at an example.

Did you know that Jeff Bezos owns `relentless.com` and 302 redirects it to Amazon?

So if we have Julia crawl it:

`relentless_com = HTTP.head("http://relentless.com/")`

... and then ask for the status code:

`relentless_com.status`

... we get:

`200`

... because that's the status code of the _eventual_ page you end up on, `https://www.amazon.com/`.

However, if we add our `redirect=false` argument:

`relentless_com = HTTP.head("http://relentless.com/", redirect=false)`

... we get:

`302`

## Crawling just the headers

If you're not sure what headers are, check out the box below.

<aside>

### What are HTTP headers?

HTTP headers are a form of "meta information" sent between the server and the browser (and vice versa). Examples of HTTP headers sent by a server to a browser are the date the web page was last modified, and the length of time for which the page may be cached. By contrast, headers sent the other way---from browser to server---may include cookies or the referring page.

</aside>

### Worked example

Let's fetch the headers from `example.com`. As mentioned earlier, because we aren't fetching the whole page, we don't need to use `HTTP.get()` (which fetches the headers _and_ all the HTML); we can use `HTTP.head()` (which only fetches the headers) instead. We're saving bandwidth (ours and that of the web server) this way.

First, we visit the page:

`example_com = HTTP.head("https://example.com/")`

... and then we request the headers:

`headers = example_com.headers`

This is the output I got:

```
12-element Array{Pair{SubString{String},SubString{String}},1}:
  "Accept-Ranges" => "bytes"
            "Age" => "390198"
  "Cache-Control" => "max-age=604800"
   "Content-Type" => "text/html; charset=UTF-8"
           "Date" => "Sat, 01 Aug 2020 03:25:23 GMT"
           "Etag" => "\"3147526947\""
        "Expires" => "Sat, 08 Aug 2020 03:25:23 GMT"
  "Last-Modified" => "Thu, 17 Oct 2019 07:18:26 GMT"
         "Server" => "ECS (sjc/4E74)"
           "Vary" => "Accept-Encoding"
        "X-Cache" => "HIT"
 "Content-Length" => "1256"
 ```

Note that some or all of the dates in your output may be different.

### Extracting the relevant headers from the Julia array by index

You can then pull the relevant key-value pair(s) from this array as needed. For instance, if you want the `Last-Modified` header (which is the eighth element in the `headers` array we created), you could do this:

`last_modified = headers[8]`

... and you'd get this:

`"Last-Modified" => "Thu, 17 Oct 2019 07:18:26 GMT"`

Or if you want just the _value_, you could do this:

`last_modified = headers[8][2]`

... and you'd get this:

`"Thu, 17 Oct 2019 07:18:26 GMT"`

NB: The `[2]` at the end of the last command above signifies that we want only the _second_ item in the key-value pair, namely, the value, which in this case is the actual date and time the page was last modified. If we'd used `[1]` instead of `[2]`, we'd get just the key, which in this case is simply the words `"Last-Modified"`.

### Extracting the relevant headers from the Julia array by key name

It's probably a better idea to extract the value of a given header by its _name_ (e.g. `"Last-Modified"` rather than its position in the array.

For example, if you're scraping multiple web pages, some might have 11-element header arrays, and some might have 9, 10, 12, and so on.

Here's how to get the value of a given header (we'll work with `"Last-Modified"`) by name.

Using our same `headers` array we created above:

```
for header in headers
  if header[1] == "Last-Modified"
    println(header[2])
  end
end
```

Above, we iterate over each header in the `headers` array (`for header in headers`).

Then we say that if the header key (name) is "Last-Modified" (`if header[1] == "Last-Modified"`), then print the value of that header (`println(header[2])`). Of course, you could save it to another array or a dataframe; I've just provided you with a simple example above.
