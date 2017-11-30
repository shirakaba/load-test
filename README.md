# load-test
A basic log-based experiment to investigate when each DOM element is loaded in turn.

# How to run

* Visit: https://shirakaba.github.io/load-test/

* Open the console (I'm assuming Chrome for the following instructions)

* In the 'network' tab, ensure that 'disable cache' is checked. 

* You may wish to adjust the network speed down to 'Slow 3G', too.

* Press `command-shift-R` to reload the page whilst clearing the cache.

* Examine the order in which each message appeared in the console.


# Results

## Materials

I did a cursory test on (from oldest to newest): Chrome 62.0.3202.94, Safari 11.0.1, Firefox 49.0.2, and Opera 12.16.

## Nomenclature to follow

* 'static' means that the element was specified purely by HTML declaration.

* 'dynamic' (by contrast) means that the element was specified by JS.

* 'inline' means that the contents of the element were written into `index.xhtml` directly, rather than necessitating a HTTP request.


## Data

Provided the cache was indeed disabled, the events consistently ran in this order whether over `localhost` or over `github.io`:

1. The `<head>`'s first inline `<script>` element finishes running.

2. The SECOND static `<link>` element (small, local) loads.

3. The FIRST static `<link>` element (large, external) loads.

4. The first static `<script>` (large, external) element loads.

5. The second static `<script>` (small, local) element loads.

6. The `<head>`'s second inline `<script>` element finishes running.

7. The `<body>`'s first inline `<script>` finishes running.

8. The `<body>`'s second inline `<script>` (the 'listing task') finishes running.

9. `DOMContentLoaded` fires.

10. The `<body>`'s static `<img>` element loads.

11. The first dynamic `<link>` element (large filesize; external site) requested by the `<head>`'s second inline `<script>` execution loads.

12. The second dynamic `<link>` element (small filesize; local) requested by the `<head>`'s second inline `<script>` execution loads.

13. The first dynamic `<script>` element (large filesize; external site) requested by the `<head>`'s second inline `<script>` execution loads.

14. The second dynamic `<script>` element (small filesize; local) requested by the `<head>`'s second inline `<script>` execution loads.

15. `document.body.onload()` (same as `window.onload()`, I am told) is triggered.


# Interpretation

* The browser traverses the HTML structure from **top to bottom** (`<head>` is only traversed first because it's traditionally placed at the top! I confirmed this by briefly swapping `<head>` with `<body>` and the loading order did indeed change correspondingly).

* While traversing the `<head>`, it addressed each element in turn:

    * The first inline `<script>` (containing reporting functions) was run.

    * The first static `<link>` (large filesize; external site) was requested, but responded AFTER the following-specified element:

    * The second static `<link>` (small filesize; local) was requested, and responded shortly BEFORE the first static `<link>` responded.

    * The first static `<script>` (large filesize; external site) was requested, and this time responded before the small static `<script>` that would follow it. The network inspector shows that it was requested at the same time the previous two requests were made (a batch of three). I've read that HTTP allows batches of two requests to be made to each domain in parallel, and indeed, this shares the same CDN as the first static `<link>`. However, it's not clear then why the following static `<script>` wasn't also batched (as it shared the same domain as the second static `<link>`); maybe `index.xhtml` consumed one of the two requests for that domain's HTTP batch?

    * The second static `<script>` (small filesize; local) was requested in a new HTTP batch and so responded after the first static `<script>`.

    * The second inline `<script>` (specifying dynamic requests for CSS and JS files) was run. These dynamic requests were not sent until all the elements in the DOM had been fully loaded (ie. the request was postponed until exactly the moment that the latest-to-finish-loading element in the `<body>` – the `<img>` element – had finished loading!)

* While the aforementioned dynamic requests were queued, the browser started addressing each element inside the `<body>` in turn:

    * The request for the `<img>` element was launched, but further events proceeded while it awaited the response:

    * The first inline `<script>` (just containing a log message) was run.

    * The second inline `<script>` (editing a `<ol>` element declared shortly before it) was run.

    * `DOMContentLoaded` fires (as all DOM elements have been placed into the page, albeit awaiting the response for data in our `<img>` element).

    * The request for the `<img>` element finally completed, permitting the dynamic requests to finally complete:

    * The dynamic requests for both CSS and JS resources were all sent in one batch. Although the network inspector shows the small, local files (which were requested after the large, external files) responding before their counterparts, the log messages show their `onload` events firing in the same order as their requests were made in, giving the impression that they all responded synchronously. This is hard to explain – it makes it seem like `onload` messages are postponed until any prior in-flight requests respond, but that seems like it would be bad practice.

* Finally, the `<body>` element's `onload` callback triggers.