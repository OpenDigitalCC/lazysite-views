# Creating views for lazysite

A view is a Template Toolkit file (`view.tt`) that controls the visual
presentation of every page on a lazysite site. This guide covers all
the variables, patterns, and conventions needed to create a view.

## Template variables

Every page render passes these variables to `view.tt`:

| Variable | Type | Description |
| -------- | ---- | ----------- |
| `page_title` | string | Page title from front matter. Always set. |
| `page_subtitle` | string | Subtitle from front matter. May be empty. |
| `content` | string | Converted page body as HTML. Output unescaped. |
| `site_name` | string | From `lazysite.conf`. May be empty if not set. |
| `site_url` | string | From `lazysite.conf`. May be empty. |
| `nav` | arrayref | Navigation from `nav.conf`. Empty array if no file. |
| `request_uri` | string | Current page path, e.g. `/about`. |
| `page_modified` | string | Human-readable mtime, e.g. "3 April 2026". |
| `page_modified_iso` | string | ISO 8601 mtime, e.g. "2026-04-03". |

Plus any additional variables defined in `lazysite.conf`.

### Checking for optional variables

Always test optional variables before using them:

```
[% IF page_subtitle %]<p class="subtitle">[% page_subtitle %]</p>[% END %]
[% IF page_modified %]<time datetime="[% page_modified_iso %]">[% page_modified %]</time>[% END %]
[% IF site_name %]<a href="/">[% site_name %]</a>[% END %]
```

## Navigation pattern

The `nav` variable is an array of hashrefs. Each item has:

- `label` - display text (always present)
- `url` - link target (empty string for non-clickable group headings)
- `children` - array of child items (empty array if none)

Each child has `label` and `url` only.

### Recommended nav loop

```
<nav>
[% FOREACH item IN nav %]
  [% IF item.url %]
  <a href="[% item.url %]"
    [% IF request_uri == item.url %]class="active"[% END %]>
    [% item.label %]</a>
  [% ELSE %]
  <span class="nav-group">[% item.label %]</span>
  [% END %]
  [% IF item.children.size %]
  <ul class="nav-children">
    [% FOREACH child IN item.children %]
    <li><a href="[% child.url %]"
      [% IF request_uri == child.url %]class="active"[% END %]>
      [% child.label %]</a></li>
    [% END %]
  </ul>
  [% END %]
[% END %]
</nav>
```

Use `aria-current="page"` instead of or alongside `class="active"` for
accessibility.

### Empty nav

Always handle the case where `nav` is empty (no `nav.conf` file):

```
[% IF nav.size %]
<nav>
  ...
</nav>
[% END %]
```

## CSS classes

Views should style these elements which appear in rendered page content:

### From Markdown conversion

- `h2` through `h6` - content headings (`h1` is the page title)
- `p` - paragraphs
- `ul`, `ol`, `li` - lists
- `blockquote` - block quotes
- `table`, `thead`, `tbody`, `tr`, `th`, `td` - tables
- `pre`, `code` - code blocks
- `pre > code` - fenced code blocks (language class: `language-bash` etc.)
- `a` - links
- `img` - images
- `strong`, `em` - bold, italic
- `dl`, `dt`, `dd` - definition lists

### From fenced divs

Authors use `:::classname` to wrap content in styled divs. Common classes:

- `widebox` - full-width coloured band
- `textbox` - highlighted box for key points
- `marginbox` - margin pull quote
- `examplebox` - evidence or example highlight

Define CSS for these classes in your view's `<style>` block.

### From oEmbed

- `div.oembed` - embedded media wrapper
- `div.oembed--failed` - failed embed fallback (contains a plain link)

### From includes

- `span.include-error` - invisible by default, expose for debugging:
  ```css
  .include-error::before { content: "include failed: " attr(data-src); color: red; }
  ```

## Theme metadata

Include a TT comment block at the top of `view.tt`:

```
[%#
  name: My Theme
  version: 1.0.0
  author: Your Name
  requires: 0.9.0
  description: Brief description of the theme.
%]
```

This is for documentation only - the processor does not parse it.

## Minimal working view

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[% page_title %][% IF site_name %] - [% site_name %][% END %]</title>
</head>
<body>
[% IF nav.size %]
<nav>
[% FOREACH item IN nav %]
  [% IF item.url %]<a href="[% item.url %]">[% item.label %]</a>[% END %]
[% END %]
</nav>
[% END %]
<main>
    <h1>[% page_title %]</h1>
    [% IF page_subtitle %]<p>[% page_subtitle %]</p>[% END %]
    [% content %]
</main>
<footer>
    <p>Built with <a href="https://lazysite.io">lazysite</a></p>
</footer>
</body>
</html>
```

## Self-contained views

Views should be self-contained - all CSS inline in a `<style>` block,
no external dependencies. This ensures the view works immediately after
installation without needing to fetch additional files.

Use system fonts:

```css
body { font-family: system-ui, -apple-system, sans-serif; }
code, pre { font-family: ui-monospace, 'Cascadia Code', monospace; }
```

## Testing a view

Test with a mock variables hash:

```perl
perl -e '
use Template;
my $tt = Template->new(ABSOLUTE => 0);
my $vars = {
    page_title    => "Test Page",
    page_subtitle => "A subtitle",
    content       => "<p>Hello world</p><pre><code>code block</code></pre>",
    site_name     => "Test Site",
    site_url      => "http://localhost",
    nav           => [
        { label => "Home", url => "/", children => [] },
        { label => "About", url => "/about", children => [] },
        { label => "Resources", url => "", children => [
            { label => "GitHub", url => "https://github.com" },
        ]},
    ],
    request_uri   => "/",
    page_modified => "1 April 2026",
    page_modified_iso => "2026-04-01",
};
my $out = "";
$tt->process("view.tt", $vars, \$out) or die $tt->error();
print $out;
' > /tmp/test-output.html
```

Check:
- Valid HTML5 (`<!DOCTYPE html>`)
- Title contains page title and site name
- Content present
- Navigation renders with active state
- Non-clickable nav parents render as plain text
- Empty nav renders gracefully
- Footer present

## Directory structure

A view directory contains:

```
viewname/
  view.tt      <- the view template (required)
  nav.conf     <- example navigation (optional)
```

The `nav.conf` is a convenience for users - it shows the nav format
but is not used by the processor directly.
