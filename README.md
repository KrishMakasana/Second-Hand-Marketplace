# ReBuyz — Second-Hand Marketplace

ReBuyz is a React + TypeScript second-hand electronics marketplace. It supports a public catalogue, search, product details, customer registration/sign-in, seller verification, product listing moderation, cart, wishlist and a demo order workflow. It can use Supabase Auth, PostgreSQL, Row Level Security (RLS) and Supabase Storage, or run in local-demo mode if no Supabase URL/key is configured.

## Requirements

- Node.js 20 or newer (Node.js 22 recommended)
- npm
- A Supabase project (for shared accounts and database storage)

## Connect your existing Supabase project

### 1. Database connection (already configured for you)

This project ZIP is preconfigured for your Supabase project **Second-Hand Marketplace** (`urqjoouqzhwertqcsbcn`). I applied the schema and verified that these six tables exist with row-level security enabled: `profiles`, `products`, `favorites`, `cart_items`, `orders`, and `order_items`. The database also contains 12 sample product listings.

The `supabase/schema.sql` file is included as a reproducible setup script if you need to create the schema in another project. Do not disable RLS to “fix” a permissions error.

### 2. Configure environment variables locally

Copy `.env.example` to `.env`, then set your Supabase project URL and publishable/anon key. Keep `.env` local; it is intentionally excluded from shared project ZIPs. If you plan to use the starter-account seed script, paste your Supabase **secret key** (or legacy `service_role` key) on this line:

```dotenv
SUPABASE_SERVICE_ROLE_KEY=paste_your_secret_key_here
```

Find the key in the Supabase dashboard under **Project Settings → API Keys** (or **Connect**). Keep it only in this local `.env` file. Never put it in a `VITE_` variable, frontend code, a screenshot or a public repository, and do not send it in chat. The seed script needs it to create users securely through Supabase Auth's admin API.

### 3. Create or verify starter accounts

After adding the secret key, use the install/run commands in step 4 and run `npm run supabase:seed-users` once. The script creates missing users in Supabase Auth and upserts their profile/role rows. Existing users with the same email keep their existing password. Starter credentials for newly created accounts are:

| Role | Email | Password |
|---|---|---|
| Admin | `admin@rebuyz.local` | `Admin@123` |
| Customer | `customer@rebuyz.local` | `Customer@123` |
| Seller | `seller@rebuyz.local` | `Seller@123` |

The starter seller is marked approved so you can test listing submission immediately. These simple credentials are for local testing only; change them or override them in `.env` before using anything outside a private development project. Supabase Auth passwords are not stored in the `profiles` table.

If you previously registered accounts in the local-demo version, those records exist only in that browser's `localStorage` and cannot be copied as valid Supabase Auth accounts. Use the seed script for the three starter users and register other users again through the app.

### 4. Install dependencies and run

```powershell
npm install --legacy-peer-deps
npm run supabase:seed-users
npm run dev
```

The seed command is safe to rerun for these three accounts; existing passwords are not changed. If you only want to run the app, you can skip the seed command.

Open the local URL printed by the terminal, usually `http://localhost:3000/`. Keep PowerShell open while testing. Stop the server with `Ctrl+C`.

If Supabase environment variables are present, the application uses Supabase Auth and PostgreSQL. If they are absent, the app falls back to browser-local demo mode. Restart `npm run dev` after changing `.env` so Vite reads the new values. The optional legacy OAuth callback logs a warning when `OAUTH_SERVER_URL` is not configured; configure that variable only if you intend to use that separate OAuth-provider flow. The Supabase email/password flow uses the Supabase variables above.

## Included database features

- **Authentication:** Supabase email/password sign-up and sign-in; a database trigger creates a profile for customer/seller registrations. Public sign-up cannot grant an admin role.
- **Profiles:** Role, seller category/type, verification status and profile image metadata.
- **Products:** Shared approved catalogue and seller-owned pending listings; public images use Supabase Storage.
- **Seller verification:** Identity/sample verification files are stored in a private storage bucket and the app stores only the object path. Admin role policies control access to those objects.
- **Wishlist/cart:** Stored per user in Supabase.
- **Orders:** The database order function rechecks approved product IDs and prices in PostgreSQL before inserting a sample order. It does not charge a customer.
- **Admin actions:** Seller approval, listing approval/rejection and order status changes use database functions that check the signed-in user's database role.

## Testing workflows

1. Sign in as `admin@rebuyz.local` to open the Admin Panel.
2. Sign in as `seller@rebuyz.local` to submit listings; new listings must be reviewed by admin before being public.
3. Sign in as `customer@rebuyz.local` to save favorites, use the cart and place sample orders.
4. Use the admin account to update the order status; the customer dashboard reads the shared database state.

## Important limits and security notes

- Checkout is still **demo checkout**: no Razorpay/Stripe charge, shipping label or delivery tracking is created.
- Use only sample identity images during course/demo testing. Real ID documents require an appropriate legal basis, user consent, retention/deletion controls and stricter operational security before production use.
- The secret/service-role key bypasses RLS and must only be used in the local seed script, never in browser code or public source control. The four SQL RPCs flagged by the Supabase security advisor are intentional server-side operations; they check the signed-in database role before performing admin actions or order placement.
- A production deployment should use unique strong credentials, configure Supabase Auth email settings and allowed redirect URLs, use proper rate limits and monitoring, and test all RLS policies before handling real users or payments.

## Build and production start

```powershell
npm run build
npm start
```


## Expanded demo product catalogue

The local demo catalogue includes a larger set of sample electronics listings with detailed descriptions and remote image URLs. In offline/demo mode, new seed products are merged into the existing local catalogue without deleting user-created listings.

For a Supabase-backed project that already has the database schema, open Supabase Dashboard → SQL Editor and run `supabase/seed-products.sql`. This inserts the additional sample products as approved demo listings. The SQL is safe to re-run because it ignores existing product handles. Replace demo descriptions, prices, and imagery with verified seller listings before using the catalogue commercially.
