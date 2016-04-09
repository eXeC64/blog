+++
date = "2016-04-09T10:01:00Z"
title = "Goodbye LaTeX, Hello HTML"
+++

## It's not me, it's you.

As a 3rd year Computer Science student, I'm no stranger to LaTeX and the joys
of typesetting documents. Most of the time I'm lazy and I just write my reports
in Markdown and compile them using the excellent [pandoc](http://pandoc.org).
Of course, the moment I need a little more control I have to revert back to
LaTeX, either embedded in the Markdown or by abandoning Markdown all together,
in order to take back control of the document's formatting. And of course, if
I ever need to deviate from LaTeX's oh-so-specific formatting, as I do from
time to time, I enter the seventh circle of hell, with cryptic backdoors and
hacks to get LaTeX to work anywhere close to how you want.

If you don't believe me, find a post-graduate student and hold up a sign that
reads `\renewcommand`. Chances are they'll have a PTSD induced panic attack.
Either that or they used Microsoft Word.

## Enter HTML

In one of my more coherent LaTeX compile failure panic attacks I had an epiphany.
I'd just done some work in HTML and CSS, and it'd been a great success. What if
I could use HTML and CSS to write documents in for print? Well, it turns out
not to have been such an original idea. Thomas Park built an interesting CSS
framework for academia called [PubCSS](http://thomaspark.co/2015/01/pubcss-formatting-academic-publications-in-html-css/).

Pretty cool work, and the examples are really polished. But in the CS tradition
of [NIH syndrome](https://en.wikipedia.org/wiki/Not_invented_here), I decided
I wanted something bespoke for my needs.

Now, I know what most of you might be thinking. HTML+CSS? What about page numbers,
tables of contents, citations?

Well, it turns out HTML+CSS can actually manage all of those, with the exception
of the table of contents. You can either built that manually and use CSS to
automatically display the correct page numbers for the relevant sections, or
you you can write some JavaScript to generate the table of contents automatically.
Being a programmer, I took the lazy route: I automated it.

So what new magic gives us page numbers, automatic citations, and more? Well,
[CSS Generated Content for Paged Media](https://www.w3.org/TR/css-gcpm-3/). A
deceptively dull title for a really cool feature.

Here's an example of its power:

```css
@page {
  size: a4;
  margin: 3.5cm;

  @bottom-right-corner {
    content: counter(page);
  }
}
```

That simple bit of CSS just let me select a paper size (you can specify exact
dimensions in metric or imperial if you like too), set a margin, and then
put some arbitrary content in the bottom right corner of every page. In this
case, the page number. Oh, and you can control the formatting of the page
number, set and reset the count at any time, or display the current chapter
title, the possibilites are almost endless.

So let's say you have a front cover, some preamble, and then the main content.
Your front cover doesn't want any page numbers, bur your preamble wants page
numbers in lower case roman numerals, and your main content wants page numbers
starting from 1 in our regular arabic numerals. Easy!

```html
<body>
<div>
  <h1>This is my front page! Isn't it cool.</h1>
  <p>I'd like to thank the academy...</p>
</div>

<div id=preamble>
  <h1>Abstract</h1>
  <p>In here I put all the content for my preamble.</p>
</div>

<div id=main>
  <h1>Introduction</h1>
  <p>This is the main content. Here I want normal page numbers.</p>
</div>
</body>
```

```css
#main {
  page: main; /* Set current page */
}

#preamble {
  page: preamble;
}

@page main { /* For "main" pages */
  reset-counter: page; /* Restart the page count */
  @bottom-right-corner {
    content: counter(page);
  }
}

@page preamble {
  @bottom-right-corner {
    content: counter(page, lower-roman); /* lower-roman format this time */
  }
}
```

### Page breaks

So what do we do when we want a page break? Well, there's a useful set of
css properties for influencing them:

* `page-break-before`
* `page-break-after`
* `page-break-during`

Which we can set to values such as `always` or `avoid`. For example, if we
want all `<h1>`s to start on a new page:

```css
h1 {
  page-break-before: always;
}
```

It's really that easy.

If you want to force a new page somewhere arbitrary you can always create a
class for that:

```css
.page-break {
  page-break-before: always;
}
```

```html
<p>Something at the end of one page</p>
<div class="page-break" />
<p>This is on a new page!</p>
```

### Section numbers

Want to number your sections? That's easy too. Here's an example of a two-level
numbering scheme.

```css
h1 {
  counter-increment: section;
  counter-reset: subsection;
}

h2 {
  counter-increment: subsection;
}

h1::before {
  content: counter(section) ". ";
}

h2::before {
  content: counter(section) "." counter(subsection) ". ";
}
```

And now our sections are numbered automatically for us.

### References

This is starting to get a little tricksy, but not much more so than before.
This will give us nice pretty engineering style references. They even act
as intra-document links on a PDF.

```css
#bibliography {
  counter-reset: ref;
}

#bibliography li {
  margin: 0.5em;
  counter-increment: ref;
}

#bibliography li::marker {
  content: "[" counter(ref) "]";
}

a.ref::after {
  font-style: normal;
  content: "[" target-counter(attr(href), ref) "]";
  font-size: 0.7em;
}
```

```html
<div id="main">

<p>
  In some cases, foos were found to be quite dangerous<a class=ref href="#foo"></a>.
  However, in <a class=ref href="#bar">A Study on Bars</a> this was disputed.
</p>

</div>

<div id="bibliography">
  <h1>Bibliography</h1>
  <ol>
    <li id="foo">
      Foos considered harmful.
      Author Name Here.
      2010.
    </li>
    <li id="bar">
      A talk on foos considered harmful.
      Someone Else
      2012.
    </li>
  </ol>
</div>
```

And the result is perfect. You can use the same technique to number and refer
to figures easily.


### Footnotes

These turn out to be even easier. Try it.

```css
.footnote {
  float: footnote;
}
```

```html
<p>This is a sentence<span class=footnote>As sentences go though, it's pretty dull.</span>.</p>
```

### Columns

Also easy.

```css

#main {
  column-count: 3;

}

#main h1 {
  column-span: all; /* let <h1>s break through columns */
}
```

### Images

This I saved til last because it's what irritated me about LaTeX the most. I
wanted an image in my document scaled by half and aligned to the right, with
a black border and a caption beneath.

In LaTeX it's a nightmare. In HTML it's second nature.

```html
<div style="width: 50%; float: right; border: 1px solid black;">
  <img src="img.png" style="width: 100%;" />
  <p>This image is awesome.</p>
</div>
```

## Compiling

HTML is all very well, but how do you get your glorious PDF output at the end?
The answer is a wonderful tool called [PrinceXML](http://www.princexml.com).
It really is that good.

For commercial use there's a fee, but for non-commercial use you can download
and use it freely. The only caveat is that the free version adds an annotation
to the first page of the output PDF declaring that it was generated by Prince.
The annotation is easily removed using a standard PDF viewer, so it presents
no trouble to an academic. If you're looking to generate PDFs automatically,
I highly recommend you splash out on a license.

And yes, you can invoke it using GNU Make:

```makefile
report.pdf: report.html style.css
	prince -v -o $@ $<
```
