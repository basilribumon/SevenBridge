# SEVEN BRIDGE Supermarket

A responsive React storefront and Express/MongoDB API for browsing grocery products, maintaining daily prices, preparing WhatsApp orders, and storing web-submitted orders.

## Requirements

- Node.js 20 or newer and npm
- MongoDB (local or hosted)

## Run locally with MongoDB Atlas

1. Create a MongoDB Atlas account and project, then deploy a free cluster.
2. In Atlas Security Quickstart, create a **database user** and add your current IP address. Keep the database password private. Atlas requires connections to come from an IP address on the project's access list; avoid opening access to every IP for production.
3. In the cluster's Connect dialog, choose the Node.js driver and copy its connection string. Replace the username/password placeholders and use `seven-bridge` as the database name.
4. Open the existing `.env` file. Set `MONGODB_URI` to that connection string and replace `ADMIN_PASSWORD=change-this-before-seeding` with a unique password of at least 12 characters. `ADMIN_EMAIL` is already set to the Gmail supplied by the owner. Do not share either password in chat or commit `.env`.
5. In VS Code Terminal, run `node server/seed.js`. This creates the admin account and four sample products.
6. Start the API in one terminal with `node server/index.js`.
7. Start the storefront in a second terminal with `node ./node_modules/vite/bin/vite.js --host 127.0.0.1`; open the **Local** URL it prints.

The Atlas database user in step 2 is separate from the website admin login in step 4. If Atlas reports an IP access error, update the project's IP Access List for your current connection. When deploying the API, add the hosting server's outbound IP to Atlas and update `CLIENT_ORIGIN` to the deployed storefront URL.

The storefront has a demo catalogue for visual preview when the database is empty. Website checkout and the admin panel require a running API and MongoDB. Product and store settings changes require admin login. Configure store details, WhatsApp, delivery availability, hours, and Maps link under “Store owner sign in.” Delivery stays disabled until an administrator enables it. The backend rechecks product prices, stock, product availability, integer-only packaged quantities, and delivery configuration at submission time; price and subtotal snapshots are saved with each order.

## Brand and store setup

The supplied official logo is included at `public/sblogo.png` and appears in the navbar, footer, and favicon. The theme uses the logo’s red and gold colors, and the slogan reads “It’s your happiness.” The temporary phone number is 9847328745 and the address is Muttippalam, Manjeri, Malappuram, Kerala, near HP Petrol Pump. The Maps button uses the location link supplied by the store owner. Confirm these details with the store owner before launch; WhatsApp remains separate and unconfigured until its number is provided. Unsplash image URLs are used for product and lifestyle imagery; set product image URLs through the product API when deploying.

## Features

- Responsive home, category browsing, product search, sorting, and quick quantity controls.
- Cart saved in the browser, checkout review, explicit optional geolocation sharing, and WhatsApp click-to-chat message generation.
- Express API with Mongoose models for products, admins, orders, and store settings.
- Password hashing with bcrypt, expiring JWT admin sessions, protected mutation routes, Helmet, rate-limited login, Zod validation, and server-side order price/stock validation.
- Admin can add and archive products, update daily price/stock via the product API, manage store contact settings, inspect order counts/orders, and change order status.
- Customers can look up orders using their checkout name and phone; order status refreshes while the tracking view stays open.

## API overview

- `GET /api/health`, `/api/products`, `/api/settings`
- `POST /api/auth/login`
- Admin bearer token: `POST/PUT/DELETE /api/products`, `PUT /api/settings`, `GET /api/orders`, `PATCH /api/orders/:id/status`, `GET /api/admin/summary`
- `POST /api/orders` (server-calculated prices and totals)
- `GET /api/orders/track?name=...&phone=...` (customer order lookup)

For editing a product, `PUT /api/products/:id` accepts the same fields as product creation. Deleting a product archives it so old orders retain their snapshots.

## Deployment

The repository includes `render.yaml` for a single Render Node web service. It builds the React/Vite site and serves the site and Express API from the same HTTPS origin. In Render, choose **New → Blueprint**, connect this repository, and provide `MONGODB_URI`, `ADMIN_EMAIL`, and a new `ADMIN_PASSWORD` when prompted. Render generates `JWT_SECRET`. Never commit `.env` or put secrets in frontend variables. The deployment uses the configured MongoDB Atlas database; make sure its database user has only the required access to the `seven-bridge` database and its network access allows the service to connect.

Render's free web service sleeps after 15 minutes without traffic, so the first visit after an idle period may take about a minute to wake. Render documents its free services as suitable for testing and previews, not production use. Consider a paid always-on service and a custom domain before relying on this for customer orders.

## Notes

- WhatsApp opens a pre-filled draft; the customer reviews and sends it. Opening WhatsApp is not recorded as an order.
- Website orders are saved only when the API responds successfully. The customer sees a clear fallback prompt when the API is unavailable.
- Payment is not collected or represented as paid. Delivery fee defaults to zero and is included in the saved total only when delivery is enabled.
- The favicon currently uses the browser's default; replace/add a branded favicon when the official logo arrives.
