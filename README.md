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

2. The static `<link>` element loads.

3. The static `<script>` element loads.

4. The `<head>`'s second inline `<script>` element finishes running.

5. The `<body>`'s first inline `<script>` finishes running.

6. The `<body>`'s second inline `<script>` (the 'listing task') finishes running.

7. The `<body>`'s static `<img>` element loads.

8. The first dynamic `<link>` element (large filesize; external site) requested by the `<head>`'s second inline `<script>` execution loads.

9. The second dynamic `<link>` element (small filesize; local) requested by the `<head>`'s second inline `<script>` execution loads.

9. The first dynamic `<script>` element (large filesize; external site) requested by the `<head>`'s second inline `<script>` execution loads.

10. The second dynamic `<script>` element (small filesize; local) requested by the `<head>`'s second inline `<script>` execution loads.

11. `body.onload()` is triggered.


# Interpretation

* The browser traverses the HTML structure from **top to bottom** (`<head>` is only traversed first because it's traditionally placed at the top! I confirmed this by briefly swapping `<head>` with `<body>` and the loading order did indeed change correspondingly).

* While traversing the `<head>`, it ran each element in turn:

    * The inline `<script>` was run.

    * The static `<link>` (large filesize; external site) was requested. The 