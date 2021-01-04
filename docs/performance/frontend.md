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

## HTTP Caching & assets

- Mobile cache sizes are small 100-500MB
- Using CDNs for all external assets -- while good idea in theory -- would imply that your users would have all your assets cached somehow to get considerable performance gains, if not, this would lead to extra connections, which are expensive. We are better of with concatenation and using single application.js over a connection that will be reused.

## Resource Hints

There are various hints we can give to browsers to improve perceived speed. These may or may not be acted upon depending on the our browser's support.

### DNS Prefetch, Preconnect, Prefetch, Prerender

`Preconnect`

```html
<!-- 

    Resolve the DNS, if not done already (1 round-trip)
    Open a TCP connection (1.5 round-trips)
    Complete a TLS handshake if the connection is HTTPS (2-3 round-trips)

 -->
<link rel="preconnect" href="//example.com">
```

> Only thing it wouldn't do is to download the resource.

`Prefetch`

```html
<!-- 

    Everything that we did to set up a connection in the preconnect hint (DNS/TCP/TLS).
    But in addition, the browser will also actually download the resource.
    However, prefetch only works for resources required by the next navigation, not for the current page.

 -->
<link rel="prefetch" href="//example.com/some-image.gif">
```

> Consider using prefetch in any case where you have a good idea what the user might do next.

`Prerender`

Prerender is prefetch on steroids -- it will actually download the whole page. This means prerendering works on HTML documents -- not on scripts or sub-resources.

> Be careful when using prefetch and prerender together. If you’re prefetching something on your own server, you’re effectively adding another request to your server load for every prefetch directive. A prerender directive can be even more load-intensive because the browser will also fetch all sub resources (CSS/JS/images, etc), which may also come from your servers. It’s important to only use prerender and prefetch where you can be pretty certain a user will actually use those resources on the next navigation.

### Checklist 2

- Reduce number of connections
- HTTP caching is great, but don't rely on particular resource being cached
- Use resource hints -- particularly `preconnect` and `prefetch`