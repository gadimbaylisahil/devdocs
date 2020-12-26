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