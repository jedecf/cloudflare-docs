---
type: example
summary: Fetch and modify response properties which are immutable by creating a
  copy first.
tags:
  - Middleware
  - Headers
pcx_content_type: configuration
title: Modify response
weight: 1001
layout: example
---

{{<tabs labels="js/esm | js/sw">}}
{{<tab label="js/esm" default="true">}}

```js
export default {
  async fetch(request) {
    /**
     * @param {string} headerNameSrc Header to get the new value from
     * @param {string} headerNameDst Header to set based off of value in src
     */
    const headerNameSrc = 'foo'; //"Orig-Header"
    const headerNameDst = 'Last-Modified';

    /**
     * Response properties are immutable. To change them, construct a new
     * Response and pass modified status or statusText in the ResponseInit
     * object. Response headers can be modified through the headers `set` method.
     */
    const originalResponse = await fetch(request);

    // Change status and statusText, but preserve body and headers
    let response = new Response(originalResponse.body, {
      status: 500,
      statusText: 'some message',
      headers: originalResponse.headers,
    });

    // Change response body by adding the foo prop
    const originalBody = await originalResponse.json();
    const body = JSON.stringify({ foo: 'bar', ...originalBody });
    response = new Response(body, response);

    // Add a header using set method
    response.headers.set('foo', 'bar');

    // Set destination header to the value of the source header
    const src = response.headers.get(headerNameSrc);

    if (src != null) {
      response.headers.set(headerNameDst, src);
      console.log(
        `Response header "${headerNameDst}" was set to "${response.headers.get(
          headerNameDst
        )}"`
      );
    }
    return response;
  },
};
```
{{</tab>}}
{{<tab label="js/sw">}}
```js
/**
 * @param {string} headerNameSrc Header to get the new value from
 * @param {string} headerNameDst Header to set based off of value in src
 */
const headerNameSrc = 'foo'; //"Orig-Header"
const headerNameDst = 'Last-Modified';

async function handleRequest(request) {
  /**
   * Response properties are immutable. To change them, construct a new
   * Response and pass modified status or statusText in the ResponseInit
   * object. Response headers can be modified through the headers `set` method.
   */
  const originalResponse = await fetch(request);

  // Change status and statusText, but preserve body and headers
  let response = new Response(originalResponse.body, {
    status: 500,
    statusText: 'some message',
    headers: originalResponse.headers,
  });

  // Change response body by adding the foo prop
  const originalBody = await originalResponse.json();
  const body = JSON.stringify({ foo: 'bar', ...originalBody });
  response = new Response(body, response);

  // Add a header using set method
  response.headers.set('foo', 'bar');

  // Set destination header to the value of the source header
  const src = response.headers.get(headerNameSrc);

  if (src != null) {
    response.headers.set(headerNameDst, src);
    console.log(
      `Response header "${headerNameDst}" was set to "${response.headers.get(headerNameDst)}"`
    );
  }
  return response;
}

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});
```
{{</tab>}}
{{</tabs>}}
