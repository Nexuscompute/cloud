---
title: Static Websites & Assets
menuText: Static Websites & Assets
description: Serve static assets like images, CSS, and JavaScript, and host front-end apps and websites.
menuOrder: 7
parent: Building Applications
---

# Static Websites & Assets

Serverless Cloud allows you to serve files from your application URL. This is useful for serving static assets such as images, CSS, and JavaScript, allowing you to host front-end apps and websites. By convention, static assets must be stored in the `static` directory at the root of your application.

You can have sub-directories in the static directory, but there are some reserved sub-directory names:

- `public`: this is reserved for public files created using the "storage" interface, which are available from the path `/public/*`

## Static HTML pages

Serverless Cloud also supports serving static HTML pages:

- requests for `/` will return `static/index.html` if it exists
- requests for `/<page>` will return `static/page.html` if it exists

This also applies for sub-directories in the `static` directory. For example, a request for `/admin` will return `/static/admin/index.html` if it exists, and a request for `/admin/page` will return `/admin/page.html`.

**NOTE:** Avoid having static pages that have corresponding API routes. For example, if you have a `/users` route, and also a `/static/users.html` page, Serverless Cloud will return the static page, and the API route will be unreachable.

## Custom server error pages

When a request is made for a page that doesn't exist, Serverless Cloud will return `static/404.html` if it exists, with a 404 status code. If this file doesn't exist, Serverless Cloud will return a default 404 page.

You can also customize the response that is sent when your application throws an error by creating a static file `static/500.html`.

## Dynamic error pages

If you would like your application to return a dynamic `404` or `500` response instead of using static files, you can write your own error handlers that override the default behavior.

See the [Express documentation on handling errors](https://expressjs.com/en/guide/error-handling.html) for more information.

## Falling back to index.html for single-page applications (SPAs)

To return your `index.html` page for any missing path, you can add a 404 handler with the `http` interface:

```javascript
import { http } from "@serverless/cloud";

http.on(404, "index.html");
```

## Using React, Vue, and other SPAs

If you are using a SPA framework or static site generator that requires a _build_ step, you can store your source files in your project directory and configure your output directory to be `/static`. To prevent Serverless Cloud from syncing your source files, you can add a **`.serverlessignore`** file in the root of your project and add a list of directories and files you do not wish to sync.

For example, if your front-end source files are stored in `/src`, your **`.serverlessignore`** file should contain the following:

```
src
```

This will allow you to run a separate terminal with your SPA build scripts running and only sync when it generates output files.

**NOTE:** Please be sure to restart your cloud shell after changing the `.serverlessignore` file.

## Static asset caching

Static assets are automatically cached in Cloud's Content Delivery Network (CDN) in edge locations around the world so download speeds will be very fast.

Caching works differently in "stage" and "personal" instances.

In stage instances, static assets are cached in the CDN for up to 24 hours. Responses will include a `Cache-Control` header that tells the CDN to cache the asset for 24 hours, and tells the browser to cache the asset and "revalidate" it before using it. When you deploy a new version of your application, Cloud will automatically clear the CDN cache so your users will get the latest version when they refresh the browser.

Caching is disabled in developer sandboxes so you can update your assets and immediately see the latest version when you reload your browser. Responses for static assets in your developer sandbox will include a `Cache-Control` header that disables caching in the CDN, and an additional `X-Cache-Control` header that shows you the value of the header that will be used in stage instances.

**NOTE:** You should always use a stage instance for your "production" instance, to take advantage of the CDN and ensure the best performance for your users.

## Reading static assets from application code

Although static assets can be read from the file system in sandboxes, they are not stored in the file system when your application is deployed to a stage. We recommend that you _not_ read static assets from your application. Any files your application needs at runtime should be stored within your project outside the "static" folder. For example, you can create an "assets" folder to hold images or html files that your application can then read from the file system at runtime.

If your application still needs to read static files, it is possible to do so using the `http.assets.readFile(path)` method. This will return a readable stream you can use to read the file. For example to read an image in your project that is in "static/images/image.jpeg" and process it using `jimp` you could use:

```javascript
import { http } from "@serverless/cloud";

const stream = await http.assets.readFile("images/image.jpeg");

// convert to Buffer for Jimp
const chunks = [];
for await (const chunk of stream) {
  chunks.push(chunk);
}
const buffer = Buffer.concat(chunks);

const image = await Jimp.read(buffer);
```

## Prioritizing API routes over static assets

Static assets normally take precedence over API routes that may overlap. For example, a request for the root path "/" will return your `static/index.html` file if it exists, even if you have an API handler for it.

If your use case includes overlapping API routes and static assets, you can make your API take precedence using the `apiBeforeStatic` option in `http.config`:

```javascript
import { api, http } from "@serverless/cloud";

http.config({
  apiBeforeStatic: true,
});

// ignore index.html and send a JSON response instead
api.get("/", (req, res) => {
  res.status(200).send({ ok: true });
});
```
