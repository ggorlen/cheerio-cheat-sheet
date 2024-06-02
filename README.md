# Cheerio Cheat Sheet

Accurate as of 1.0.0-rc.12

Note: Cheerio does not execute JavaScript. It operates on static HTML and XML strings.

## Load HTML from string

```javascript
const cheerio = require("cheerio");

const html = `<h1>Hello World</h1>`;
const $ = cheerio.load(html);
console.log($("h1").text()); // => Hello World
```

## Load HTML from file

```javascript
const cheerio = require("cheerio");
const fs = require("node:fs/promises");

(async () => {
  // test.html contains <h1>Hello World</h1>
  const html = (await fs.readFile("test.html")).toString();
  const $ = cheerio.load(html);
  console.log($("h1").text()); // => Hello World
})();
```

## Load HTML from HTTP response

With fetch (Node 18+ or with node-fetch):

```javascript
const cheerio = require("cheerio");

const url = "https://www.example.com";
const ua =
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36";

fetch(url, {headers: {"User-Agent": ua}})
  .then(res => {
    if (!res.ok) {
      throw Error(res.statusText);
    }

    return res.text();
  })
  .then(html => {
    const $ = cheerio.load(html);

    // select element by text with :contains
    console.log($("h1:contains('Example')").text()); // => Example Domain
  })
  .catch(err => console.error(err));
```

With axios:

```javascript
const axios = require("axios");
const cheerio = require("cheerio");

const url = "https://example.com";
const ua =
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36";

axios
  .get(url, {headers: {"User-Agent": ua}})
  .then(({data: html}) => {
    const $ = cheerio.load(html);
    console.log($("h1:contains('Example')").text()); // => Example Domain
  })
  .catch((err) => console.error(err));
```

With Gooogle Apps Script:

Install [cheeriogs](https://github.com/tani/cheeriogs).

```javascript
function myFunction() { // default GAS function name
  const url = "https://www.example.com";
  const response = UrlFetchApp.fetch(url);

  // if debugging is necessary, bypass log truncation:
  // DriveApp.createFile("test.html", response.getContentText());

  const $ = Cheerio.load(response.getContentText());
  console.log($('h1:contains("Domain")').text()); // => Example Domain
}
```

## Get text contents of one element

```javascript
const html = `<p>test</p>`;
const $ = cheerio.load(html);
console.log($("p").first().text()); // => test
```

## Get text contents of multiple elements

```javascript
const html = `<p>test</p><p>test 2</p>`;
const $ = cheerio.load(html);
const text = $("p").map((i, e) => $(e).text()).get();
console.log(text); // => [ 'test', 'test 2' ]
```

Or:

```javascript
const text = [...$("p")].map(e => $(e).text());
```

Or:

```javascript
const text = [];
$("p").each(function (i, e) {
  text.push($(e).text());
});
```

Or:

```javascript
const text = [];
$("p").each(function () {
  text.push($(this).text());
});
```

## Get attribute of one element

```javascript
const html = `<p class="foo">test</p>`;
const $ = cheerio.load(html);
console.log($("p").first().attr("class")); // => foo
```

## Get attributes of multiple elements

```javascript
const html = `<p class="baz">test</p><p class="quux">test 2</p>`;
const $ = cheerio.load(html);
const classes = [...$("p")].map(e => $(e).attr("class"));
console.log(classes); // => [ 'baz', 'quux' ]
```

(other iteration strategies shown above also work)

## Get all attributes of an element

```javascript
const html = `<p id="foo" class="baz">test</p>`;
const $ = cheerio.load(html);
const attributes = $("p").attr();
console.log(attributes); // => { id: 'foo', class: 'baz' }
```

## Get value of an element

```javascript
const html = `<input value="foo">`;
const $ = cheerio.load(html);
console.log($("input").val()); // => foo
```

## Get data attribute from an element

```javascript
const html = `<p data-testid="foo">`;
const $ = cheerio.load(html);
console.log($("p").data("testid")); // => foo
```

## Get the `innerHTML` of an element

```javascript
const html = `<p><b>foo</b></p>`;
const $ = cheerio.load(html);
console.log($("p").html()); // => <b>foo</b>
```

## Get the `outerHTML` of an element

```javascript
const html = `<p>foo</p>`;
const $ = cheerio.load(html);
console.log($.html($("p"))); // => <p>foo</p>
```

## Get the first element

```javascript
const html = `<p>one</p><p>two</p><p>three</p>`;
const $ = cheerio.load(html);
const text = $("p").first().text();
console.log(text); // => one
```

Or:

```javascript
const text = $("p:first").text();
```

## Get the last element

```javascript
const html = `<p>one</p><p>two</p><p>three</p>`;
const $ = cheerio.load(html);
const text = $("p").last().text();
console.log(text); // => three
```

Or:

```javascript
const text = $("p:last").text();
```

## Get the next element

```javascript
const html = `<p>one</p><p>two</p><p>three</p>`;
const $ = cheerio.load(html);
const text = $("p").next().text();
console.log(text); // => two
```

## Get the next element matching selector

```javascript
const html = `<p>one</p><p>two</p><p>three</p>`;
const $ = cheerio.load(html);
const text = $("p").next(':contains("three")').text();
console.log(text); // => three
```

## Get the previous element

```javascript
const html = `<p>one</p><p>two</p><p>three</p>`;
const $ = cheerio.load(html);
const text = $(":contains('two')").prev().text();
console.log(text); // => one
```

## Get previous sibling elements

```javascript
const html = `
<ul>
  <li>a</li>
  <li>b</li>
  <li>c</li>
  <li>d</li>
</ul>`;
const $ = cheerio.load(html);
const text = $("li:contains('d')")
  .first()
  .prevAll("li:contains('a'), li:contains('b')") // optional filter
  .addBack(/* optional filter could go here */) // optionally include 'd' and reverse
  .get()
  .map(e => $(e).text());
console.log(text); // => [ 'a', 'b', 'c', 'd' ]
```

(`.nextAll` is also available)

### Get previous elements up to a selector

```javascript
const html = `
<ul>
  <li>a</li>
  <li>b</li>
  <li>c</li>
  <li>d</li>
</ul>`;
const $ = cheerio.load(html);
const text = $("li:contains('d')")
  .first()
  .prevUntil("li:contains('a')")
  .get()
  .map(e => $(e).text());
console.log(text); // => [ 'c', 'b' ]
```

(`.nextUntil` is also available)

## Get the n-th element

```javascript
const html = `<p>one</p><p>two</p><p>three</p>`;
const $ = cheerio.load(html);
const text = $($("p").get(1)).text();
console.log(text); // => two
```

Or:

```javascript
const text = $($("p").eq(1)).text();
```

## Get the n-th element from the rear

```javascript
const html = `<p>one</p><p>two</p><p>three</p>`;
const $ = cheerio.load(html);
const text = $($("p").get(-2)).text();
console.log(text); // => two
```

## Get the parent element

```javascript
const html = `<div class="outer"><div class="inner"></div></div>`;
const $ = cheerio.load(html);
const cls = $(".inner").parent().attr("class");
console.log(cls); // => outer
```

## Find nearest parent by selector

```javascript
const html = `
<div><p>foo</p></div>
<div><p>bar</p></div>`;
const $ = cheerio.load(html);
const el = $(":contains('bar')").closest("div");
console.log($.html(el)); // => <div><p>bar</p></div>
```

## Find path to a parent by selector

```javascript
const html = `
<div class="A">
  <div class="B">
    <div class="C">
      <p></p>
    </div>
  </div>
</div>`;
const $ = cheerio.load(html);
const classes = $("p")
  .parentsUntil("body")
  .map((_, e) => $(e).attr("class"))
  .get();
console.log(classes); // => [ 'C', 'B', 'A' ]
```

## Get the next text node

```javascript
const html = `<p>foo</p>bar<br>`;
const $ = cheerio.load(html);
const text = $("p")[0].nextSibling.nodeValue;
console.log(text); // => bar
```

## Get multiple next text nodes

```javascript
const html = `<p>foo</p>bar<p>baz</p>quux`;
const $ = cheerio.load(html);
const text = [...$("p")].map(e => e.nextSibling.nodeValue);
console.log(text); // => [ 'bar', 'quux' ]
```

## Get index of element among its siblings

```javascript
const html = `<p>foo</p><p>bar</p>`;
const $ = cheerio.load(html);
const i = $("p:contains('bar')").index();
console.log(i); // => 1
```

## Get child elements

```javascript
const html = `<div class="outer"><div>A</div><div>B</div></div>`;
const $ = cheerio.load(html);
const text = $(".outer").children().map((i, e) => $(e).text()).toArray();
console.log(text); // => [ 'A', 'B' ]
```

## Get the immediate text contents

```javascript
const html = `<div>one<p>ignore</p>two<p>ignore</p>three</div>`;
const $ = cheerio.load(html);
const text = [...$("div").contents()]
  .filter(e => e.type === "text" && $(e).text().trim())
  .map(e => $(e).text());
console.log(text); // => [ 'one', 'two', 'three' ]
```

## Query immediate children

```javascript
const html = `
<div>
  <p>
    <b>foo</b>
  </p>
</div>`;
const $ = cheerio.load(html);
const results = $("div").children("b");
console.log(results.length); // => 0
```

## Query descendents

```javascript
const html = `
<div>
  <p>
    <b>foo</b>
  </p>
</div>`;
const $ = cheerio.load(html);
const text = $("div").find("b").text();
console.log(text); // => foo
```

## Filter by presence of element among children

```javascript
const html = `<p>foo<br></p><p>bar</p><p><br>baz</p>`;
const $ = cheerio.load(html);
const text = $("p")
  .has("br")
  .map((i, e) => $(e).text())
  .toArray();
console.log(text); // => [ 'foo', 'baz' ]
```

## Filter by presence of text among children

```javascript
const html = `
<div><p>foo</p></div>
<div><p>bar</p></div>
<div><p>foobar</p></div>`;
const $ = cheerio.load(html);
const text = $("div")
  .has(":contains(foo)")
  .map((_, e) => $(e).text())
  .get();
console.log(text); // => [ 'foo', 'foobar' ]
```

## Scrape a table

```javascript
const html = `
<table>
  <tbody>
    <tr>
      <th>Name</th>
      <th>Age</th>
    </tr>
    <tr>
      <td>Mike Mulligan</td>
      <td>37</td>
    </tr>
    <tr>
      <td>Joe Schmoe</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
`;
const $ = cheerio.load(html);
const tableData = [...$("tr")].map(row =>
  [...$(row).find("td")].map(e => $(e).text())
);
console.table(tableData);
/*
┌─────────┬─────────────────┬──────┐
│ (index) │        0        │  1   │
├─────────┼─────────────────┼──────┤
│    0    │                 │      │
│    1    │ 'Mike Mulligan' │ '37' │
│    2    │  'Joe Schmoe'   │ '35' │
└─────────┴─────────────────┴──────┘
*/
```

## Get HTML between two siblings

```javascript
const html = `
<h2>A</h2>
<p>a1</p>
<p>a2</p>
<h2>B</h2>
<p>b1</p>
<p>b2</p>`;
const $ = cheerio.load(html);
const newHTML = $.html($("h2").first().nextUntil("h2"));
console.log(newHTML); // => <p>a1</p><p>a2</p>
```

## Get text between two siblings

```javascript
const html = `
<h2>A</h2>
<p>a1</p>
<p>a2</p>
<h2>B</h2>
<p>b1</p>
<p>b2</p>`;
const $ = cheerio.load(html);
const text = $("h2")
  .first()
  .nextUntil("h2")
  .map((i, e) => $(e).text())
  .get();
console.log(text); // => [ 'a1', 'a2' ]
```

## Get all text between siblings

```javascript
const html = `
<h2>A</h2>
<p>a1</p>
<p>a2</p>
<h2>B</h2>
<p>b1</p>
<p>b2</p>`;
const $ = cheerio.load(html);
const groups = [...$("h2")]
  .map(e => [...$(e).nextUntil("h2")].map(e => $(e).text()));
console.log(groups); // => [ [ 'a1', 'a2' ], [ 'b1', 'b2' ] ]
```

## Get tag name of a single element

```javascript
const html = `<p class="test"></p>`;
const $ = cheerio.load(html);
const tag = $(".test").eq(0).tagName
console.log(tag); // => p
```

## Get tag names of multiple elements

```javascript
const html = `<p class="test"></p><div class="test"></div>`;
const $ = cheerio.load(html);
const tags = [...$(".test")].map(e => e.tagName);
console.log(tags); // => [ 'p', 'div' ]
```

Or:

```javascript
const html = `<p class="test"></p><div class="test"></div>`;
const $ = cheerio.load(html);
const tags = [...$(".test")].map(e => $(e).prop("tagName"));
console.log(tags); // => [ 'P', 'DIV' ]
```

## Modify text

```javascript
const html = `<p>foo</p>`;
const $ = cheerio.load(html);
console.log($("p").text()); // => foo
$("p").text("bar");
console.log($("p").text()); // => bar
```

## Modify attributes

```javascript
const html = `<p class="foo"></p>`;
const $ = cheerio.load(html);
console.log($("p").attr("class")); // => foo
$("p").attr("class", "bar");
console.log($("p").attr("class")); // => bar
$("p").removeAttr("class");
console.log($("p").attr("class")); // => undefined
```

## Add and remove classes

```javascript
const html = `<p class="foo"></p>`;
const $ = cheerio.load(html);
console.log($("p").attr()); // => { class: 'foo' }
$("p").addClass("bar");
console.log($("p").attr()); // => { class: 'foo bar' }
$("p").removeClass("foo");
console.log($("p").attr()); // => { class: 'bar' }
$("p").toggleClass("foo");
console.log($("p").attr()); // => { class: 'bar foo' }
$("p").toggleClass("foo");
console.log($("p").attr()); // => { class: 'bar' }
console.log($("p").hasClass("bar")); // => true
```

## Append/prepend HTML

```javascript
const html = `<div></div>`;
const $ = cheerio.load(html);
$("div")
  .append("<p>foo</p>")
  .prepend("<p>bar</p>");
console.log($.html("div")); // => <div><p>bar</p><p>foo</p></div>
```

Or:

```javascript
const html = `<p>one</p><p>two</p>`;
const $ = cheerio.load(html);
$("<b>PRE</b>").prependTo("p");
$("<b>POST</b>").appendTo("p");
console.log($("body").html()); // => <p><b>PRE</b>one<b>POST</b></p><p><b>PRE</b>two<b>POST</b></p>
```

## Insert HTML

```javascript
const html = `<div><p>foo</p></div>`;
const $ = cheerio.load(html);
$("p:contains('foo')")
  .before("<p>baz</p>")
  .after("<p>quux</p>");
console.log($.html("div"));
// => <div><p>baz</p><p>foo</p><p>quux</p></div>
```

Or:

```javascript
const html = `<div><p>foo</p></div>`;
const $ = cheerio.load(html);
$("<p>baz</p>").insertBefore("p:contains('foo')");
$("<p>quux</p>").insertAfter("p:contains('foo')");
console.log($.html("div"));
// => <div><p>baz</p><p>foo</p><p>quux</p></div>
```

## Replace HTML

```javascript
const html = `<p>foo</p>`;
const $ = cheerio.load(html);
$("p").replaceWith("<div>bar</div>");
console.log($.html("div")); // => <div>bar</div>
```

## Remove elements

```javascript
const html = `<div><p>foo</p></div><div><p>bar</p></div>`;
const $ = cheerio.load(html);
$("div").first().remove();
console.log($("body").html()); // => <div><p>bar</p></div>
```

## Remove children of an element

```javascript
const html = `<div><p>foo</p><p>bar</p></div>`;
const $ = cheerio.load(html);
$("div").empty();
console.log($("body").html()); // => <div></div>
```

## Lift children up to a parent

```javascript
const html = `
<div>
  <p>foo</p>
  <p>bar</p>
  <div class="to-remove">
    <p>baz</p>
    <p>garply</p>
  </div>
</div>`;
const $ = cheerio.load(html);
$(".to-remove p").unwrap();
console.log($("body").html()); // => <div></div>
/*
<div>
  <p>foo</p>
  <p>bar</p>

    <p>baz</p>
    <p>garply</p>

</div>
*/
```

## Wrap children together

```javascript
const html = `
<div>
  <p>foo</p>
  <p>bar</p>
  <p>baz</p>
  <p>garply</p>
</div>`;
const $ = cheerio.load(html);
$(":nth-child(3), :nth-child(4)").wrapAll("<div></div");
console.log($("body").html());
/*
<div>
  <p>foo</p>
  <p>bar</p>
  <div><p>baz</p><p>garply</p></div>

</div>
*/
```

## Wrap children individually

```javascript
const html = `
<div>
  <p>foo</p>
  <p>bar</p>
  <p>baz</p>
  <p>garply</p>
</div>`;
const $ = cheerio.load(html);
$(":nth-child(3), :nth-child(4)").wrap("<div></div");
console.log($("body").html());
/*
<div>
  <p>foo</p>
  <p>bar</p>
  <div><p>baz</p></div>
  <div><p>garply</p></div>
</div>
*/
```

## Get text in a comment

```javascript
const html = `<p>foo<!-- bar --><!-- baz --></p>`;
const $ = cheerio.load(html);
const comments = [...$("p").contents()]
  .filter(e => e.type === "comment")
  .map(e => e.data);
console.log(comments); // => [ ' bar ', ' baz ' ]
```

## Get text in a `<noscript>` tag

```javascript
const html = `<noscript><p>get me</p></noscript>`;
const $ = cheerio.load(html, {xml: true});
const text = $("p").text();
console.log(text); // => get me
```

## Use `$` in helper functions without having to pass it each time

Best I can offer is a closure--is there a better way in newer Cheerio versions?

```javascript
const bindCheerioHelpers = $ => ({
  textAll: s => [...$(s)].map(e => $(e).text().trim()),
});

const main = () => {
  const html = `<p>foo</p><p>bar</p>`;
  const $ = cheerio.load(html);
  const {textAll} = bindCheerioHelpers($);
  console.log(textAll("p")); // => [ 'foo', 'bar' ]
};
main();
```

## Query XML CDATA

```javascript
const xml = `<description><![CDATA[<p>foo</p>]]></description>`;
const $ = cheerio.load(xml, {xml: true});
const text = $.load($("description").text())("p").text();
console.log(text); // => foo
```
