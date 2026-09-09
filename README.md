# HTTPLeaks

## What is this?

This project enumerates all the ways a document can cause a browser (or a mail client, a proxy, a
server-side DOM) to send an HTTP request, in one single HTML file:
[`leak.html`](leak.html) ([raw](https://raw.githubusercontent.com/cure53/HTTPLeaks/main/leak.html)).

Every entry points at `https://leaking.via/<element>-<attribute>`, so when a request shows up you can
read off exactly which construct fired it from the path alone. The domain is not real; the file is a
corpus, not a beacon.

## What is it for?

With "HTTP leak" we mean a situation where some combination of markup, CSS, SVG or MathML causes a
request to an external resource when it should not. Think of the body of an HTML mail, where a leak
tells someone that you just opened it. Not always bad, almost never good.

Typical uses:

- **HTML sanitizers and filters.** Feed the file through and see what still fires. Sanitizers that
  strip `<script>` and `on*` and call it a day tend to miss half of this file.
- **Web-mailers and reader modes.** These render untrusted HTML without scripting, which is exactly
  where `<noscript>`, conditional comments and CSS-only leaks matter.
- **Web proxies and "anonymizers".** A proxy that rewrites `src` and `href` but not `srcset`,
  `ping`, `@import` or `image-set()` has no anonymity to offer.
- **CSP and browser testing.** Check which of these your policy actually blocks, and which requests
  leave the browser process without ever appearing in devtools.

Nobody really knows anymore which elements and attributes can request external resources, which is
why this project exists: to keep track.

## What is in the file

The file is organised in `%Section` comment blocks, roughly in document order: `<head>` metadata,
`<link>` relations, images, forms, media, `<object>`/`<embed>`, script and speculation rules, frames,
Declarative Shadow DOM, `<noscript>`, `<model>`, CSS (stylesheet, inline, exotic), SVG, XSLT, MSIE data
islands, VML, MathML, and finally a block of URL spellings that sanitizers tend to misclassify.

Three kinds of entries live side by side, and the comments say which is which:

- **Live leaks** in current engines. The bulk of the file.
- **Legacy leaks** that only fire in engines that are gone from the desktop (IE, old Firefox, the
  Flash and Java plugin era) but live on in mail clients and embedded webviews. Outlook still renders
  VML and `[if mso]` blocks. These stay in on purpose: the corpus is more useful complete.
- **Probes** that are expected *not* to fire and are there as regression inputs, for example the
  `attr()` URL restriction in CSS Values 5. Their comments say "expected: no request".

A few entries need user interaction (`ping`, `formaction`, MathML `href`, `longdesc`); the visible
text says "click me" or "hover me" where that is the case.

## How to test with it

1. Serve `leak.html` over HTTPS from an origin that is not `leaking.via`. Opening it from disk changes
   the rules for several loaders (mixed content, `file:` origin, CSP `<meta>` handling).
2. Watch the network at the DNS or wildcard-listener level, not only in devtools. Several requests are
   made by the browser process (OpenSearch), after the page is idle (compression dictionaries), or by a
   prefetch/prerender pipeline, and do not all show in the Network panel.
3. Run it more than once: with scripting on and off (the `<noscript>` block only fires with scripting
   off), in each engine you care about, and in the actual client you are protecting (Outlook, a
   webmail sandbox, your proxy). Browser results do not transfer to mail clients and vice versa.
4. Later CSS declarations of the same property win, so where the file lists several `url()` variants
   on one selector you will only see the last one fire. Split them up if you need each individually.

## Contributing

Pull requests with new leaks are very welcome. Please:

- Use the naming scheme: `https://leaking.via/<element>-<attribute>` (or `<context>-<property>` for
  CSS), unique per entry.
- Put it under the matching `%Section`, or add a new one with a `%` comment.
- Say in a comment where it fires (engine and version, or client) and whether interaction is needed.
  If it is a probe that should *not* fire, say that too.
- Do not remove legacy entries. Annotate them instead.

Ideas for other presentations of this data (JSON, per-engine tables, a scripted runner) are also
welcome; the single file is the source of truth.

## Related

- [DOMPurify](https://github.com/cure53/DOMPurify) sanitizes HTML, SVG and MathML; its
  [hooks-proxy-demo](https://github.com/cure53/DOMPurify/blob/main/demos/hooks-proxy-demo.html) shows
  how to route or drop resource-loading attributes with a hook, and the
  [threat model](https://github.com/cure53/DOMPurify/wiki/Security-Goals-&-Threat-Model) explains why
  stopping HTTP leaks is explicitly a non-goal of a sanitizer.
- [DOMFortify](https://github.com/cure53/DOMFortify) retrofits Trusted Types sanitization onto legacy
  pages; this file is a good input for testing what a sanitizer-backed policy still lets through.

## Acknowledgements

Thanks [@masatokinugawa](https://github.com/masatokinugawa), [@hasegawayosuke](https://twitter.com/hasegawayosuke),
[@masa141421356](https://twitter.com/masa141421356), [@mramydnei](https://twitter.com/mramydnei),
[@avlidienbrunn](https://twitter.com/avlidienbrunn), [@orenhafif](https://twitter.com/orenhafif),
[@freddyb](https://twitter.com/freddyb), [@tehjh](https://twitter.com/tehjh), [@webtonull](https://twitter.com/webtonull),
[@mikewest](https://github.com/mikewest), [@hackvertor](https://github.com/hackvertor), [@shhnjk](https://github.com/shhnjk),
[@parrot409](https://github.com/parrot409), [@mtrzos](https://github.com/mtrzos), [@johannburkard](https://github.com/johannburkard),
[@xem](https://github.com/xem), [@emersion](https://github.com/emersion), [@DaKnOb](https://github.com/DaKnOb),
[@rohansgit](https://github.com/rohansgit), @intchloe, [@Boldewyn](https://github.com/Boldewyn) and many others for adding
content and smaller fixes here and there!
