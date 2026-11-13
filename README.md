# SplitEase 💸

SplitEase is a full-stack expense sharing app. Users create groups, add shared expenses, see who owes whom, and settle up either online (Razorpay) or manually (cash / UPI / bank transfer) by uploading a payment receipt that the other person approves.

## 🚀 Features

- 🔐 **Authentication** – register / login with JWT
- 👥 **Groups** – create groups, add and remove members
- 💰 **Expenses** – add expenses, split between members, categorise them (Food, Travel, Rent, etc.)
- 📊 **Balances & stats** – see what you owe and are owed, with spending charts on the dashboard
- 🧮 **Debt simplification** – optimised settlement suggestions to minimise the number of payments
- 💳 **Online payments** – Razorpay checkout and webhook verification
- 🏦 **Manual settle-up** – pay by cash / UPI / bank transfer, upload a receipt screenshot, and the receiver approves or rejects it
- 🔒 **Expense locking** – expenses are locked once a payment is approved
- 📄 **PDF receipts** – generated after approval, with a QR code that opens a verification page (`/verify/:paymentId`)
- 📧 **Email notifications** – receipt submitted / approved / rejected, plus daily owing reminders (9 AM cron job)
- 📱 **Responsive UI** – sidebar layout with a mobile menu

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 18, React Router 6, Axios, Recharts, Vite |
| Backend | Node.js, Express |
| Database | MongoDB with Mongoose |
| Auth | JSON Web Tokens, bcryptjs |
| Uploads | Multer (saved in `server/uploads`) |
| Payments | Razorpay |
| Email | Nodemailer (Gmail SMTP) |
| PDF / QR | PDFKit, qrcode |
| Scheduling | node-cron |

## 📂 Project Structure

```
splitease/
├── client/                     # React frontend (Vite)
│   └── src/
│       ├── pages/              # Login, Register, Dashboard, Groups, Expenses, SettleUp, Profile
│       ├── components/         # Navbar, Sidebar, cards, charts, Razorpay button
│       ├── context/            # AuthContext
│       └── utils/              # cache helper
├── server/                     # Express backend
│   ├── config/                 # MongoDB connection
│   ├── controllers/            # auth, expense, group, payment, razorpay
│   ├── middleware/             # JWT auth guard
│   ├── models/                 # User, Group, Expense, Payment
│   ├── routes/                 # API + /verify routes
│   ├── utils/                  # debt simplifier, email service, PDF generator
│   ├── uploads/                # uploaded receipt images
│   └── server.js               # app entry point
├── package.json                # root build / start scripts
└── README.md
```

## ⚙️ Installation

### Prerequisites

- Node.js 18 or newer
- A MongoDB database (local or MongoDB Atlas)
- A Gmail account with an App Password (for emails)
- Razorpay keys (only if you want online payments)

### Clone and install

```bash
git clone https://github.com/alokraj427845-web/Expensetracker.git
cd Expensetracker

cd client && npm install
cd ../server && npm install
```

## 🔑 Environment Variables

Create a file named `.env` inside the **`server`** folder:

```env
# Server
PORT=5000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_long_random_secret

# Frontend URL (used for CORS and cron reminders)
# Local development: http://localhost:5173
# Production: your deployed site URL
APP_URL=http://localhost:5173

# Gmail SMTP
# 1. Enable 2-step verification on your Google account
# 2. Go to myaccount.google.com/apppasswords
# 3. Generate an App Password for "Mail" and paste it below (no spaces)
GMAIL_USER=your_email@gmail.com
GMAIL_APP_PASS=your_16_char_app_password

# Razorpay (optional – only needed for online payments)
RAZORPAY_KEY_ID=your_key_id
RAZORPAY_KEY_SECRET=your_key_secret
RAZORPAY_WEBHOOK_SECRET=your_webhook_secret
```

> ⚠️ Never commit your `.env` file. Make sure `.env` is listed in `.gitignore`.

## ▶️ Running Locally

Open two terminals.

**Terminal 1 – backend (port 5000)**

```bash
cd server
npm run dev
```

**Terminal 2 – frontend (port 5173)**

```bash
cd client
npm run dev
```

Open **http://localhost:5173**.

The Vite dev server proxies `/api` and `/uploads` to the backend on port 5000 (see `client/vite.config.js`), so uploaded receipts open correctly in development.

## 🌐 Production Build

The Express server serves the built React app, so everything runs on a single domain.

```bash
# from the project root
npm run build     # installs dependencies and builds client/dist
npm start         # starts the server (serves API + frontend)
```

Set the environment variables from above on your hosting platform, with `APP_URL` set to your live site URL.

> **Note:** Hosts such as Render and Railway use a temporary filesystem, so files in `server/uploads` are deleted on every redeploy. For permanent receipt storage, use a persistent disk or a cloud service such as Cloudinary or AWS S3.

## 🔄 How Manual Settle-Up Works

1. The payer opens **Settle Up**, chooses **Settle manually**, picks the payment mode and uploads a receipt screenshot.
2. The server creates a payment with status `pending_receipt`, then saves the file and moves it to `receipt_submitted`. The receiver is emailed.
3. The receiver opens the receipt (**View →**) and either:
   - **Approves** – the payment becomes `approved`, linked expenses are locked, and PDF receipts are emailed to both people.
   - **Rejects** – the payment returns to `pending_receipt` and the payer uploads again.
4. After approval, a PDF receipt with a QR code can be downloaded.


## 🧰 Troubleshooting

**Clicking "View receipt" takes me to the dashboard**

- Make sure `client/vite.config.js` proxies `/uploads` to `http://localhost:5000`, then restart `npm run dev`.
- If the file no longer exists on the server (for example after a redeploy), the server returns `Receipt file not found`. The receipt must be uploaded again.

**CORS errors**

- `APP_URL` in `server/.env` must exactly match the URL you open in the browser (for example `http://localhost:5173`).

**Emails are not sending**

- Use a Gmail App Password, not your normal password, and make sure 2-step verification is enabled.

## 👨‍💻 Author

Your Name – [GitHub](https://github.com/alokraj427845-web)
