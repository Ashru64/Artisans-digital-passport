# Artisans Digital Passport

A full-stack platform that gives every handmade product a **digital passport**: a QR-scannable page showing who made it, the story behind it, the materials used, and whether a cooperative has verified it. Buyers can check authenticity in seconds, and artisans get a transparent record of their work and earnings.

**Live demo:** https://artisans-digital-passportfrontend.vercel.app

---

## Why this project

Handmade crafts are often copied or sold without credit to the people who made them. Buyers have no easy way to verify what they are buying, and artisans have little visibility into how much of the sale price reaches them. Artisans Digital Passport addresses both problems with a verifiable, shareable record for each product.

## Features

**For artisans**
- Register and sign in with an OTP sent to their phone number
- Create a passport for each product with its story, materials, region, price and payout
- Upload a short product video and poster image
- Download or share a QR code for each passport
- Track earnings and manage (including delete) their products

**For cooperatives**
- Register a cooperative and sign in with an OTP
- Onboard artisans under the cooperative
- Review products and mark them as verified
- Use the payout transparency dashboard to compare listed price with artisan payout

**For buyers**
- Scan a QR code to open a public passport page, with no account needed
- See the artisan, craft story, materials, region, cooperative and verification status
- Report a suspicious product, which flags it for cooperative review

**General**
- Multilingual interface: English, Hindi and Odia (i18next), with the choice remembered between visits
- Responsive UI built with Tailwind CSS

## Tech stack

| Layer | Technologies |
|---|---|
| Frontend | React 19, Vite, React Router, Tailwind CSS 4, i18next, Lucide icons |
| Backend | Node.js, Express 5, Mongoose |
| Database | MongoDB (Atlas or local) |
| Auth | OTP login (hashed, expiring, rate-limited) with JWT sessions, bcrypt for passwords |
| Uploads | Multer (MP4/WebM/QuickTime video up to 10 MB, plus image posters) |
| Deployment | Vercel (client) |

## Project structure

```
.
├── client/                 # React + Vite frontend
│   └── src/
│       ├── api/            # API client
│       ├── components/     # Shared UI (QR code, media, headers)
│       ├── locales/        # en.json, hi.json, or.json
│       ├── pages/          # Landing, artisan portal, cooperative, public passport
│       └── utils/
├── server/                 # Express API
│   ├── config/             # Database connection
│   ├── controllers/        # artisan, auth, cooperative, passport, upload
│   ├── middleware/         # auth, uploads, error handling
│   ├── models/             # Artisan, Cooperative, Product
│   ├── routes/
│   ├── utils/              # OTP, JWT, phone helpers
│   └── seedCooperative.js  # Sample cooperative seeder
└── package.json            # Root scripts to run client and server together
```

## Getting started

### Prerequisites
- Node.js 18 or newer
- A MongoDB connection string (MongoDB Atlas or a local instance)

### Installation

```bash
git clone https://github.com/Ashru64/Artisans-digital-passport.git
cd Artisans-digital-passport

# Install dependencies for the root, server and client
npm install
npm install --prefix server
npm install --prefix client
```

### Configuration

Create `server/.env` from the example and fill in your values:

```bash
cp server/.env.example server/.env
```

| Variable | Description |
|---|---|
| `MONGO_URI` | MongoDB connection string |
| `JWT_SECRET` | Long random secret used to sign tokens |
| `JWT_EXPIRES_IN` | Token lifetime (for example `7d`) |
| `PORT` | API port (default `5000`) |
| `CLIENT_URL` | Comma-separated allowed frontend origins for CORS |
| `OTP_EXPIRY_MINUTES` | How long an OTP stays valid |
| `OTP_MAX_ATTEMPTS` | Wrong attempts allowed before a new OTP is needed |
| `OTP_RESEND_COOLDOWN_SECONDS` | Minimum wait between OTP requests |
| `SERVE_CLIENT` | Set to `true` to serve the built client from the API |

The client reads `VITE_API_URL` (see `client/.env.example`). Leave it empty in local development, because Vite proxies `/api` and `/uploads` to `http://localhost:5000`. In production, set it to your API origin.

> **Never commit real secrets.** Keep `server/.env` out of version control.

### Seed a sample cooperative (optional)

```bash
cd server
node seedCooperative.js
```

This inserts a sample verified cooperative. If no database is reachable, it falls back to an in-memory MongoDB instance.

### Run the app

```bash
# From the repository root: starts API (port 5000) and client (port 5173)
npm start
```

Open http://localhost:5173.

## OTP login in development

SMS delivery is not integrated yet. When `NODE_ENV` is not `production` (or `DEMO_OTP=true`), the API logs the OTP to the server console and returns it in the response so the UI can display it. Set `NODE_ENV=production` and leave `DEMO_OTP` unset to stop returning the OTP, and connect an SMS provider before real-world use.

## API overview

| Method | Endpoint | Auth | Purpose |
|---|---|---|---|
| GET | `/api/health` | none | Health check |
| POST | `/api/artisans/register` | none | Register an artisan |
| POST | `/api/auth/send-otp` | none | Send a login OTP to an artisan |
| POST | `/api/auth/verify-otp` | none | Verify OTP and receive a JWT |
| GET | `/api/artisans/me` | artisan | Current artisan profile |
| GET, POST | `/api/artisans/products` | artisan | List or create products |
| GET, DELETE | `/api/artisans/products/:productId` | artisan | View or delete a product |
| POST | `/api/artisans/uploads/video` | artisan | Upload product video and poster |
| GET | `/api/artisans/earnings` | artisan | Earnings summary |
| POST | `/api/cooperatives/register` | none | Register a cooperative |
| POST | `/api/cooperatives/send-otp` | none | Send a cooperative login OTP |
| POST | `/api/cooperatives/verify-otp` | none | Verify OTP and receive a JWT |
| GET | `/api/cooperatives/me` | cooperative | Current cooperative profile |
| POST | `/api/cooperatives/artisan` | cooperative | Register an artisan under the cooperative |
| GET | `/api/cooperatives/artisans` | cooperative | List the cooperative's artisans |
| GET, POST | `/api/cooperatives/products` | cooperative | List or create products |
| PATCH | `/api/cooperatives/products/:productId/verify` | cooperative | Verify a product |
| GET | `/api/cooperatives/transparency` | cooperative | Payout transparency data |
| GET | `/api/passports/:passportId` | none | Public passport data (QR target) |
| POST | `/api/passports/:passportId/report` | none | Report a product for review |

## Data model

- **Artisan**: name, email, phone, craft, region, experience, cooperative reference, phone-verified flag
- **Cooperative**: name, region, craft type, registration number, phone, status (`pending`, `verified`, `rejected`)
- **Product**: name, description, category, artisan, cooperative, status (`Waiting`, `Checked`, `Reported`), listed price, artisan payout, materials, craft story, region, image/video/poster URLs, verification date

## Roadmap

- Integrate a real SMS provider for OTP delivery
- Move uploads from local disk to cloud storage (Cloudinary is already configured in the environment template)
- Add automated tests and CI
- Add screenshots and a short demo video to this README

## Author

**Ashrumochan Rath**: [GitHub](https://github.com/Ashru64) · [LinkedIn](https://www.linkedin.com/in/ashrumochanrath/)

## License

ISC
