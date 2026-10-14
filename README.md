# ShopSwift · Point of Sale (MERN)

A complete point-of-sale web app built with **M**ongoDB (live database), **E**xpress (API), **R**eact (screens) and **N**ode.js.

**Features**
- **Dashboard**: today's revenue, bills, customers, products, low-stock alerts, recent bills
- **Point of Sale**: scan a barcode or click products, quantity, discount, tax, Cash / Card / UPI, automatic change
- **Customers**: add, edit, delete, purchase history, loyalty points (1 point per Rs 100)
- **Products and categories**: barcode/SKU, price, stock, low-stock warnings
- **Receipts**: on screen, printed, or downloaded as `.txt`; reprint any past bill from Sales
- **Printing**: any printer, USB thermal printer, or network thermal printer (58mm or 80mm)
- **Settings**: shop name, address, tax, receipt footer, printer, auto-print, database status

---

## 1. What you need

- **Node.js 18 or newer**: https://nodejs.org (choose the LTS version)
- **Visual Studio Code**: https://code.visualstudio.com
- **A MongoDB database**: free MongoDB Atlas account (see section 3)

---

## 2. Open and install in VS Code

1. Extract the zip file.
2. In VS Code choose **File → Open Folder** and select the `ShopSwift-POS` folder.
3. Open a terminal: **Terminal → New Terminal**.
4. Install everything (server and website) with one command:

```bash
npm install
```

---

## 3. Connect the live database (MongoDB Atlas)

1. Create a free account at https://www.mongodb.com/cloud/atlas (you can sign in with GitHub).
2. Create a free **M0** cluster.
3. **Database Access**: add a database user and remember its password.
4. **Network Access**: add `0.0.0.0/0` (allow access from anywhere).
5. Click **Connect → Drivers** and copy the connection string.
6. In the `server` folder, copy `.env.example` to a new file named `.env` and paste your string:

```
PORT=5000
MONGODB_URI=mongodb+srv://USER:PASSWORD@cluster0.xxxxx.mongodb.net/shopswift?retryWrites=true&w=majority
```

Replace `USER` and `PASSWORD` with your database user. Never share the `.env` file or push it to GitHub. It is already listed in `.gitignore`.

If `MONGODB_URI` is not set, the app tries a local MongoDB at `mongodb://127.0.0.1:27017/shopswift`.

---

## 4. Run the app

```bash
npm run dev
```

Then open **http://localhost:5173** in your browser.

- The sidebar shows **Database connected** when the database is working.
- On the first run, demo categories, products and customers are added. You can edit or delete them.
- The API runs at http://localhost:5000/api (for example http://localhost:5000/api/health).

---

## 5. Set up the printer

Open **Settings & Printer** in the sidebar. There are three modes:

| Mode | Use it when | How it works |
|---|---|---|
| **Any printer (browser)** | You have any printer on this computer | Opens the normal print dialog. Choose your printer and print. |
| **USB thermal printer** | You have a USB receipt printer | Click **Connect USB printer**, pick your printer once, and receipts print with no dialog. Use Chrome or Edge. |
| **Network thermal printer** | Your receipt printer is on Wi-Fi or LAN | Enter the printer's IP address (usually port 9100). The server sends the receipt. Run the app on the shop network. |

Other printer settings:
- **Paper width**: 80mm (standard) or 58mm (small).
- **Print automatically after checkout**: the receipt prints as soon as a bill is saved.
- **Print test page**: checks the printer. Save your settings first.

**Tips**
- For thermal printers, set the paper size to 80mm (or 58mm) and margins to none in the print dialog.
- Thermal printers use basic characters. Names written in Urdu or other scripts may show as `?` on the paper. Browser printing supports them fully.
- A network printer must be reachable from the computer running the server. A cloud-hosted server (Render) cannot reach a printer inside your shop, so use USB or browser printing for live deployments.

---

## 6. Project structure

```
ShopSwift-POS/
├── package.json              # npm install and npm run dev for the whole project
├── README.md
├── server/                   # Express API + MongoDB
│   ├── .env.example          # copy to .env and add your database string
│   └── src/
│       ├── index.js          # starts the server and connects to MongoDB
│       ├── models.js         # Category, Product, Customer, Sale
│       ├── settings.js       # shop settings (stored in the database)
│       ├── routes.js         # products, categories, customers, sales, dashboard
│       ├── print-routes.js   # settings, health check, receipts and printing
│       ├── escpos.js         # builds thermal printer commands
│       ├── network.js        # sends data to a network printer
│       ├── utils.js          # helpers and plain-text receipt
│       └── seed.js           # demo data
└── client/                   # React website (Vite)
    ├── vercel.json           # deployment routing (see section 8)
    └── src/
        ├── App.jsx           # sidebar, navigation, status
        ├── printer.js        # USB, network and browser printing
        ├── context.js        # shared settings
        ├── pages/            # Dashboard, POS, Products, Customers, Sales, Settings
        ├── components/       # Modal, Receipt, CustomerModal, Toaster
        ├── styles.css        # main theme
        ├── pro.css           # sidebar, settings and polish
        └── print.css         # print layout (receipt only)
```

---

## 7. API summary

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | database status |
| GET / PUT | `/api/settings` | read and save shop and printer settings |
| GET / POST | `/api/products` | list or add products |
| PUT / DELETE | `/api/products/:id` | edit or delete a product |
| GET | `/api/products/barcode/:code` | find a product by barcode |
| GET / POST | `/api/categories` | list or add categories |
| DELETE | `/api/categories/:id` | delete a category (only if empty) |
| GET / POST | `/api/customers` | list or add customers |
| PUT / DELETE | `/api/customers/:id` | edit or delete a customer |
| GET | `/api/customers/:id/sales` | a customer's bills |
| GET / POST | `/api/sales` | list bills, or create a bill (totals are calculated on the server) |
| GET | `/api/sales/:id/receipt.txt` | download a receipt as text |
| GET | `/api/sales/:id/escpos` | receipt as printer data (USB) |
| POST | `/api/sales/:id/print/network` | print a receipt on the network printer |
| GET | `/api/print/test/escpos` | test page as printer data (USB) |
| POST | `/api/print/test/network` | test page on the network printer |
| GET | `/api/dashboard` | today's numbers |

Prices, totals and stock are always checked on the server, so the browser cannot change them.

---

## 8. Put it online (free)

1. **Database**: MongoDB Atlas (section 3).
2. **Code**: create a GitHub repository and upload the project. Do not upload `.env`.
3. **API (Render)**: New → Web Service → choose the repository.
   - Root Directory: `server`
   - Build Command: `npm install`
   - Start Command: `npm start`
   - Environment: add `MONGODB_URI` with your Atlas string.
4. **Website (Vercel)**: Add New → Project → choose the repository.
   - Root Directory: `client`
   - Framework Preset: Vite
   - Build Command: `npm run build`
   - Output Directory: `dist`
   - Open `client/vercel.json` and replace `YOUR-API-NAME.onrender.com` with your Render address, then commit.
5. Open the Vercel link. Anyone with the link can use the app, so keep the link private.

**Notes**
- Render's free plan sleeps when idle, so the first load after a pause can take about 30 seconds.
- The app has no login yet. Add one before using it with real customer data in a shared environment.
