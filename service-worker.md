# Service worker (PWA)

## Tips
- the service worker file need to be at the root of your web server to be able to use.

## caching strategy

https://medium.com/dev-channel/service-worker-caching-strategies-based-on-request-types-57411dd7652c

## When using workbox-webpack-plugin

It will automatically create ALL files emitted by webpack in a `precache.manifest-xxxxxxx.js` file. But if you want to include other files in precache other than those that **Webpack** processed, the `globDirectory`, `globPatterns` & sometimes `modifyUrlPrefix` should be set, see https://developers.google.com/web/tools/workbox/modules/workbox-webpack-plugin

## About update
- As the name indicates, the service worker will update in background, so a newer service worker will not take effects until user closed ALL tabs of the current domain, and reopen. Even if you set `workbox.clientsClaim()` &
`workbox.skipWaiting()` to ask the newer service worker be installed & activated immediately.
- to update the service worker file itself, it’s bad practice to add hash to sw.js file, if you absolutely need the service worker updated as soon as possible, you need to manually call `reg.update()` inside the promise register,  EX: `navigator.serviceWorker.register(swUrl).then((reg) => {reg.update(); ...})`
- To understand the lifecycle => https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle#manual_updates
- A example of standard process (no force update) for install & activate sw file => https://github.com/facebook/create-react-app/blob/b50590f7f42c75ca653fb6f831216c09c34a0f74/packages/react-scripts/template/src/serviceWorker.js

## SPA (single page application)
the app-shell is simply an html file, and for it continue to work offline, you need to **cache api call results**. For Example:
```js
// routing for page fetch API returned JSON
const apiMatchFunction = ({url, event}) => {
  // Return true if the route match our format
  return url.host.indexOf('api') >= 0 && url.pathname.indexOf('creativemedia') >= 0;
};
workbox.routing.registerRoute(
  apiMatchFunction,
  workbox.strategies.networkFirst({
    cacheName: 'creative-media-api-cache'
  })
);
```
Then the next time the api call will be responded by service worker cache if offline.

## SSR (server-side-rendering)
Don't complicate things. As the service worker is purely something in browser, even the server-side has already generated validated html files, we better use a seperate html file generated when compile client side files using webpack. Cache that template as app shell, and cache the api calls' results, as listed above.

Unless you have a complicated & dynamic application, like twitter-pwa, then you better further split the app to different parts (nav, sidebar, contents, etc), and cache them seperatly. See this answer for more details: https://stackoverflow.com/questions/49615772/how-to-cache-html-page-in-react-ssr-with-workbox
