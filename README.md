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

Deploy the Node server behind HTTPS and keep `haat.sqlite` and `uploads/` on persistent storage. QR sharing points to the deployed public URL. Video media still uses a hosted URL; payment processing, password reset, and production hosting configuration are not included.

### Google Drive setup

The device upload works without extra credentials. To enable Google Drive selection:

1. Create a Google Cloud project and enable the Google Picker API and Google Drive API.
2. Create a website API key and OAuth client ID. Configure the OAuth consent screen and add your app's origin under authorized JavaScript origins.
3. Add your API key, OAuth client ID, and Cloud project number to `.env` using the names in `.env.example`, then restart the app or rebuild it.
4. Restrict the API key to the Picker API and Drive API, and allow your app's origin plus `https://docs.google.com/*`.

Haat requests Google's per-file `drive.file` access. When the shopkeeper selects a photo, Haat copies that image into its own `uploads/` storage, so the catalog does not depend on the Drive file remaining public. See Google's [web Picker setup](https://developers.google.com/workspace/drive/picker/guides/web-picker) and [Drive scope guide](https://developers.google.com/workspace/drive/api/guides/api-specific-auth).
