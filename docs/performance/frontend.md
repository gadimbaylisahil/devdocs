# Performance tooling and tips on frontend

Even though server response times are important to measure and improve, they usually account for 10-20% of performance for overall user experience.

Network latency, page rendering, painting, scripts and assets loading take considerable amount of time.

Even when assets are `gzipped` and is very low size for the network transfer, they will still need to be uncompressed and parsed by the browser.

## Remove blocking javascript

Read more [here](https://developers.google.com/speed/docs/insights/BlockingJS)

## Checklist

- Only single remote JS and CSS files. Having this, eliminates extra network roundtrips.
- Use `async` where possible for external javascript
- CSS before Javascript. External CSS does not block further rendering of the page
- Minimize javascript
- $(document).ready() is not free. Prefer DOMContentLoaded instead.
- Use Turbolinks or any other pjax solution if you are not on SPA train

## Optimal head tag

Time to paint( more importantly -- time to paint usable content) is much important metric than the `window.load`. Decreasing total load time is much difficult than `hacking` perceived load times.

### Encoding

When browser downloads the web content, it's all in bits and bytes, which it should convert into text. Before it can read the data, it needs to know which encoding it should use. Browsers detect this by:

`Content-Type:` HTTP header - Best option as browser knows what encoding to use from the response

`meta` tag - <meta charset="UTF-8"> - Make sure this is the first thing at the top of the page

If there is not Content-Type header nor meta tag, browser will have to parse the page and guess -- using something like byte ordering of the characters -- which will be the slowest option, and may even have compatibility issues with older IE versions.

### Viewports

If you specify viewport, do it at the top of the document. Why?

Browsers translates this:

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

Into this:

```html
<style>
@viewport { 
  zoom: 1.0;
  width: device-width;
}
</style>
```

If viewport comes after your stylesheets, you will cause [a layout reflow](https://developers.google.com/speed/docs/insights/browser-reflow?hl=en), which may slow down the page.

### Async & defer

```html
<script type="text/javascript" src="//some.shitty.thirdpartymarketingsite.com/craptracker.js"></script>
```

When external scripts are added to the page, they are synchronous and blocking, which means the browser has to download and execute them before proceeding to render the page.

Add `async` tag to external scripts in use, that is not required in page rendering. This will let browser to continue parsing the page without waiting for the script to be downloaded and executed.

Add `defer` for compatibility with IE9<. See differences between async and defer [here](https://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html)



