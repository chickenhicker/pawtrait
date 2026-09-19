# Pawtrait × Printify backend

This is the piece that actually connects to **your own** Printify account. Your
API token lives here, on the server — never in the website's front-end code —
because anything shipped to a browser is visible to whoever opens dev tools.

## 1. Get a Printify store and API token

1. Create a Printify account at printify.com if you don't have one.
2. In Printify, open the menu in the top-left and choose **Add a new store**.
   You need at least one store on the account for products/orders to belong
   to — pick whichever sales channel option fits (this backend talks to
   Printify directly, so the site behind the "store" doesn't matter).
3. Go to **My Profile → Connections**, click **Generate**, name the token,
   and copy it somewhere safe. It's only shown once.

## 2. Configure this project

```bash
cd printify-backend
npm install
cp .env.example .env
```

Open `.env` and paste in `PRINTIFY_API_TOKEN`. Leave `PRINTIFY_SHOP_ID` blank
for now.

## 3. Find your shop ID

```bash
npm start
# in another terminal:
curl http://localhost:3001/api/shops
```

You'll get back a list of your stores with their `id`. Copy the one you want
and put it in `.env` as `PRINTIFY_SHOP_ID`, then restart the server.

## 4. What each route does

| Route | Purpose |
|---|---|
| `GET /api/shops` | List your Printify stores (one-time setup) |
| `GET /api/catalog/blueprints` | Browse product types (mug, tee, tote, etc.) |
| `GET /api/catalog/blueprints/:id/providers` | Print providers for that product |
| `GET /api/catalog/blueprints/:id/providers/:providerId/variants` | Sizes/colors + Printify's base cost |
| `GET /api/products` | Products currently in your shop |
| `POST /api/upload-image` | Send a photo into your Printify media library — returns an `image_id` |
| `POST /api/create-product` | Create a real product in your shop using that image |
| `POST /api/products/:id/publish` | Mark a product published (for storefront-connected shops) |
| `POST /api/orders` | Submit a paid order to production |

The full request/response shape for each Printify endpoint is documented at
https://developers.printify.com/.

## 5. A typical "upload a pet photo → real product" flow

```
1. POST /api/upload-image        { fileName, base64 }        → { id: imageId }
2. GET  /api/catalog/blueprints   (once, to find blueprint_id + print_provider_id you want)
3. GET  /api/catalog/blueprints/:id/providers/:providerId/variants  (to get variant ids/prices)
4. POST /api/create-product      { title, blueprintId, printProviderId, imageId, variantIds }
```

Step 4 returns Printify-generated mockup images of the real product —
those are what you'd show the customer as the final preview, instead of the
illustrated SVG mockups in the demo site.

## 6. Deploying it

This is a normal Node/Express app, so any of these work:

- **Render / Railway** — connect the repo, set the two env vars in their
  dashboard, done.
- **Vercel / Netlify** — wrap each route as a serverless function (a small
  rewrite, ask if you want this version).
- **Your own VPS** — `npm start` behind a process manager like `pm2`.

Wherever it lands, update the `fetch()` calls in `pawtrait.html` to point at
your backend's URL instead of the built-in demo data.

## Why this has to be a separate server

The Pawtrait site you have as a Claude-published page runs in a locked-down
sandbox that only allows a handful of script/font hosts — it can't call
`api.printify.com` (or any other external API) directly, and it has nowhere
safe to keep a secret token even if it could. A tiny server like this one,
hosted wherever you like, is what makes the real connection.
