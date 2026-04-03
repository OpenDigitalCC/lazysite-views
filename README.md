# lazysite-views

A collection of views for [lazysite](https://lazysite.io) -
the Markdown-driven static site processor for Apache.

A **view** is a Template Toolkit file (`view.tt`) that controls
the visual presentation of every page on your lazysite site.
lazysite includes a built-in fallback view, so a custom view
is optional - install one when you want a specific design.

## Installing a view

### Option 1 - copy a single view file

Download `view.tt` from the view directory and place it at:

    public_html/lazysite/templates/view.tt

Example using curl:

    curl -o public_html/lazysite/templates/view.tt \
      https://raw.githubusercontent.com/OpenDigitalCC/lazysite-views/main/default/view.tt

### Option 2 - clone the repo and copy

    git clone https://github.com/OpenDigitalCC/lazysite-views.git
    cp lazysite-views/default/view.tt public_html/lazysite/templates/

### Option 3 - use as a theme (per-page switching)

Place the view in the themes directory:

    public_html/lazysite/themes/default/view.tt

Then set in `lazysite/lazysite.conf`:

    theme: default

Or per-page in front matter:

    layout: default

## Available views

| View | Description |
| ---- | ----------- |
| default | Clean neutral light theme, no external dependencies |

## nav.conf

Each view directory includes a `nav.conf` example. Copy it to
`public_html/lazysite/nav.conf` and edit to match your site structure.

## Creating your own view

A view is a Template Toolkit file that receives these variables:

- `[% page_title %]` - page title from front matter
- `[% page_subtitle %]` - subtitle from front matter
- `[% content %]` - converted page body HTML
- `[% site_name %]` - from lazysite/lazysite.conf
- `[% site_url %]` - from lazysite/lazysite.conf
- `[% nav %]` - navigation array from lazysite/nav.conf
- `[% request_uri %]` - current page path
- `[% page_modified %]` - last modified date

See the [lazysite authoring guide](https://lazysite.io/docs/authoring)
for the full variable reference.
