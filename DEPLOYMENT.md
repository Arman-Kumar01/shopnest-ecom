# 🚀 ShopNest Deployment Guide

This guide walks you step-by-step through deploying **ShopNest** to production for free.

---

## 📋 Table of Contents
1. [Step 1: Get a Free Cloud Database (MongoDB Atlas)](#step-1-get-a-free-cloud-database-mongodb-atlas)
2. [Step 2: Deploy to Render (Recommended - Free & Easiest)](#step-2-deploy-to-render-recommended---free--easiest)
3. [Step 3: Seed Initial Data (Admin Account & Products)](#step-3-seed-initial-data-admin-account--products)
4. [Alternative A: Deploy with Docker & Docker Compose](#alternative-a-deploy-with-docker--docker-compose)
5. [Alternative B: Deploy Frontend on Vercel + Backend on Render](#alternative-b-deploy-frontend-on-vercel--backend-on-render)
6. [Environment Variables Reference](#environment-variables-reference)

---

## Step 1: Get a Free Cloud Database (MongoDB Atlas)

ShopNest requires MongoDB to store products, orders, and user sessions. You can set up a 100% free database in under 3 minutes:

1. Go to [mongodb.com/cloud/atlas/register](https://www.mongodb.com/cloud/atlas/register) and create a free account.
2. Under deployment type, select **M0 Free** (Shared Cluster, free forever).
3. Under **Security & Quickstart**:
   - Create a database user (e.g. username `shopnest_admin` and set a strong password). **Save this password.**
   - Under **Where would you like to connect from?**, select **Allow Access from Anywhere** (`0.0.0.0/0`).
4. Click **Connect** > **Drivers** (Node.js).
5. Copy your connection string. It will look like:
   ```text
   mongodb+srv://shopnest_admin:<db_password>@cluster0.xxxxx.mongodb.net/shopnest?retryWrites=true&w=majority
   ```
   *(Replace `<db_password>` with your actual password and ensure `/shopnest` is specified as the database name).*

---

## Step 2: Deploy to Render (Recommended - Free & Easiest)

Because `backend/server.js` serves both the Express API and the React production bundle, ShopNest runs as a single web service.

### Option 2A: Using the Render Blueprint (1-Click)
1. Commit and push your changes to your GitHub repository:
   ```bash
   git add .
   git commit -m "Add deployment configuration"
   git push origin main
   ```
2. Log in to [render.com](https://render.com).
3. Click **New +** > **Blueprint**.
4. Connect your `Arman-Kumar01/shopnest-ecom` repository.
5. Render will automatically detect [`render.yaml`](render.yaml) and prompt you for the `MONGO_URI`.
6. Paste your MongoDB connection string and click **Apply**.

---

### Option 2B: Manual Web Service Setup on Render
1. In the Render Dashboard, click **New +** > **Web Service**.
2. Connect your GitHub repository `shopnest-ecom`.
3. Configure the service settings:
   - **Name:** `shopnest-ecom`
   - **Region:** Choose the region closest to you (e.g. Frankfurt or Singapore)
   - **Branch:** `main`
   - **Root Directory:** *(leave blank)*
   - **Runtime:** `Node`
   - **Build Command:**
     ```bash
     npm run build
     ```
   - **Start Command:**
     ```bash
     npm start
     ```
   - **Instance Type:** `Free`
4. Expand **Environment Variables** and add:
   | Key | Value | Notes |
   |---|---|---|
   | `NODE_ENV` | `production` | Enables production SPA serving |
   | `MONGO_URI` | `mongodb+srv://...` | Your Atlas connection string |
   | `JWT_SECRET` | *(click Generate or enter any 32+ character random string)* | Used to sign auth tokens |
   | `PORT` | `10000` | Render standard port |
   | `CLOUDINARY_CLOUD_NAME` | *(optional)* | For image uploads |
   | `CLOUDINARY_API_KEY` | *(optional)* | |
   | `CLOUDINARY_API_SECRET` | *(optional)* | |
   | `RAZORPAY_KEY_ID` | *(optional)* | For payment gateway |
   | `RAZORPAY_KEY_SECRET` | *(optional)* | |

5. Click **Deploy Web Service**.
6. Render will build the React app and start the Express server. When done, your app will be live at `https://shopnest-ecom-xxxx.onrender.com`!

---

## Step 3: Seed Initial Data (Admin Account & Products)

Once the app is connected to MongoDB, seed sample products and the default administrator account:

### Method 1: Using Render Shell (Easiest)
1. On your Render dashboard, navigate to your web service.
2. Click **Shell** in the left sidebar.
3. Run:
   ```bash
   npm run seed
   ```
4. You will see:
   ```text
   MongoDB Connected: ...
   ✅ Data Imported Successfully!
   ```

### Default Credentials Created:
- **Admin Email:** `admin@shopnest.com`
- **Password:** `password123`
- Includes pre-seeded products with high-resolution Unsplash images.

---

## Alternative A: Deploy with Docker & Docker Compose

If deploying on a VPS (AWS EC2, DigitalOcean Droplet, Linode) or running locally in production mode:

1. Install Docker & Docker Compose.
2. Clone the repository and navigate into the folder:
   ```bash
   git clone https://github.com/Arman-Kumar01/shopnest-ecom.git
   cd shopnest-ecom
   ```
3. Start the application and bundled MongoDB database:
   ```bash
   docker compose up -d --build
   ```
4. Seed the database inside the container:
   ```bash
   docker exec -it shopnest-app npm run seed
   ```
5. Access your app at `http://localhost:5000` (or `http://<your-vps-ip>:5000`).

---

## Alternative B: Deploy Frontend on Vercel + Backend on Render

If you prefer deploying the frontend onto Vercel's Edge CDN:

1. **Deploy Backend to Render** as a Web Service:
   - Root Directory: `backend`
   - Build Command: `npm install`
   - Start Command: `npm start`
   - Env Vars: `MONGO_URI`, `JWT_SECRET`, `FRONTEND_URL` (set to your Vercel domain).
2. **Deploy Frontend to Vercel**:
   - Import repository on [vercel.com](https://vercel.com).
   - Root Directory: `frontend`
   - Build Command: `npm run build`
   - Output Directory: `build`
   - Set up API reverse proxy or define backend URL in [`vercel.json`](vercel.json).

---

## Environment Variables Reference

| Variable | Required | Description |
|---|---|---|
| `NODE_ENV` | Yes | Set to `production` in live environments |
| `PORT` | Auto | Server listening port (default: `5000`, Render injects `10000`) |
| `MONGO_URI` | Yes | MongoDB connection URI (Atlas or local) |
| `JWT_SECRET` | Yes | Secret key used to sign and verify JSON Web Tokens |
| `FRONTEND_URL` | No | Frontend URL for CORS (when frontend and backend are hosted on separate domains) |
| `CLOUDINARY_CLOUD_NAME` | No | Cloudinary storage account name for product image uploads |
| `CLOUDINARY_API_KEY` | No | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | No | Cloudinary API secret |
| `RAZORPAY_KEY_ID` | No | Razorpay merchant key ID for checkout payments |
| `RAZORPAY_KEY_SECRET` | No | Razorpay key secret for verifying payment signatures |
