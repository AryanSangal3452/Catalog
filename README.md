# Haat

Haat is a responsive React website and installable progressive web app for neighbourhood shops. The Node.js server provides the API, login sessions, and SQLite database.

## Run locally

Requirements: Node.js 22.13 or later.

```powershell
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). The Node server runs Vite in development middleware mode and creates `haat.sqlite` for account, shop, product, and order data. For a production build and server, run `npm start`.

## Features

- Shopkeeper sign-up and login with salted password hashing and an HTTP-only session cookie
- Per-shop product catalog with product codes, categories, descriptions, prices, and hosted photo/video URLs
- Product photo upload from the current device, stored by the Node server under `uploads/`
- Optional Google Drive image picker, importing the selected image into the shop's photo storage
- Public customer page at `/?shop=shop-slug`, searchable by product name or code
- Customer basket and pickup checkout that stores orders in SQLite
- Shopkeeper order search and status updates from new through picked up
- QR code and shareable public shop link
- Responsive layout and installable PWA shell

## Deployment notes

Deploy the Node server behind HTTPS and keep `haat.sqlite` and `uploads/` on persistent storage. QR sharing points to the deployed public URL. Video media still uses a hosted URL; payment processing and password reset are not included.

### Deploy from GitHub with Render

The repository includes a Render Blueprint in `render.yaml`. In Render, choose **New → Blueprint**, connect `AryanSangal3452/Catalog`, and review the service before deploying. The service builds the React frontend and runs the Node server from `main`; its 1 GB persistent disk stores the SQLite database and uploaded photos. Free Render web services do not support persistent disks, so this Blueprint uses the smallest paid web plan. Current published rates are about **$7/month** for compute plus **$0.25/month** for a 1 GB disk, before any applicable bandwidth charges. Review the current [Render pricing](https://render.com/pricing) in the deployment dashboard before creating it.

### Deploy on Cloudflare Workers (free tier)

The Cloudflare version uses Workers for the API and website, D1 for catalog and order data, and R2 for uploaded photos. It uses new Cloudflare storage and starts with an empty database; the local SQLite file is not deployed.

1. Create a D1 database named `haat-catalog` and an R2 bucket named `haat-catalog-uploads` in the Cloudflare dashboard.
2. Copy the database ID from the D1 page into `wrangler.toml` in place of `REPLACE_AFTER_CREATING_D1`.
3. In a terminal with Wrangler authenticated (`npx --yes wrangler login`), run `npx --yes wrangler d1 execute haat-catalog --remote --file=schema.sql` to create the tables.
4. Connect this GitHub repository under **Workers & Pages → Create application → Workers → Import a repository**. Set the build command to `npm run build` and the deploy command to `npx --yes wrangler deploy`.

The first deployment creates a `*.workers.dev` public URL. Free tier usage limits apply. For a command-line deployment instead, run `npm run deploy:cloudflare` after completing the database, bucket, and ID setup above.

### Google Drive setup

The device upload works without extra credentials. To enable Google Drive selection:

1. Create a Google Cloud project and enable the Google Picker API and Google Drive API.
2. Create a website API key and OAuth client ID. Configure the OAuth consent screen and add your app's origin under authorized JavaScript origins.
3. Add your API key, OAuth client ID, and Cloud project number to `.env` using the names in `.env.example`, then restart the app or rebuild it.
4. Restrict the API key to the Picker API and Drive API, and allow your app's origin plus `https://docs.google.com/*`.

Haat requests Google's per-file `drive.file` access. When the shopkeeper selects a photo, Haat copies that image into its own `uploads/` storage, so the catalog does not depend on the Drive file remaining public. See Google's [web Picker setup](https://developers.google.com/workspace/drive/picker/guides/web-picker) and [Drive scope guide](https://developers.google.com/workspace/drive/api/guides/api-specific-auth).
