# Deployment Guide: DCGAN Demo to Render + Vercel

This guide walks you through deploying the DCGAN demo application step-by-step.

**Architecture:**
- **Backend (FastAPI)** → Render (free tier)
- **Frontend (React)** → Vercel (free tier)

---

## Prerequisites

Before starting, make sure you have:
1. A GitHub account (https://github.com)
2. Your code pushed to a GitHub repository
3. A Render account (we'll create one)
4. A Vercel account (we'll create one)

---

## Step 1: Push Your Code to GitHub

If your code isn't on GitHub yet:

### 1.1 Create a GitHub Repository
1. Go to https://github.com
2. Click the **+** icon in the top right → **New repository**
3. Name it something like `dcgan-demo`
4. Keep it **Public** (required for free tier deployments)
5. Click **Create repository**

### 1.2 Push Your Code
Open terminal in your project folder and run:

```bash
# Initialize git if not already done
git init

# Add all files
git add .

# Commit your changes
git commit -m "Prepare for deployment"

# Add your GitHub repo as remote (replace with YOUR username and repo name)
git remote add origin https://github.com/YOUR_USERNAME/dcgan-demo.git

# Push to GitHub
git push -u origin main
```

---

## Step 2: Deploy Backend to Render

### 2.1 Create a Render Account
1. Go to https://render.com
2. Click **Get Started for Free**
3. Sign up with your **GitHub account** (easiest option)
4. Authorize Render to access your GitHub

### 2.2 Create a New Web Service
1. From the Render dashboard, click **New +** → **Web Service**
2. Connect your GitHub repository:
   - Click **Connect account** if prompted
   - Find and select your `dcgan-demo` repository
   - Click **Connect**

### 2.3 Configure the Service
Fill in these settings:

| Setting | Value |
|---------|-------|
| **Name** | `dcgan-backend` (or any name you like) |
| **Region** | Oregon (US West) or closest to you |
| **Branch** | `main` |
| **Root Directory** | `backend` |
| **Runtime** | Python 3 |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `uvicorn main:app --host 0.0.0.0 --port $PORT` |
| **Instance Type** | Free |

### 2.4 Add Environment Variables
Scroll down to **Environment Variables** and click **Add Environment Variable**:

| Key | Value |
|-----|-------|
| `PYTHON_VERSION` | `3.11` |
| `FRONTEND_URL` | `https://your-frontend-url.vercel.app` (we'll update this later) |

### 2.5 Deploy
1. Click **Create Web Service**
2. Wait for the build to complete (5-10 minutes first time)
3. Once deployed, you'll see a URL like: `https://dcgan-backend-xxxx.onrender.com`
4. **Copy this URL** - you'll need it for the frontend!

### 2.6 Test Your Backend
Open your backend URL in a browser. You should see:
```json
{
  "message": "DCGAN Demo API",
  "endpoints": {...}
}
```

---

## Step 3: Deploy Frontend to Vercel

### 3.1 Create a Vercel Account
1. Go to https://vercel.com
2. Click **Sign Up**
3. Sign up with your **GitHub account** (easiest option)
4. Authorize Vercel to access your GitHub

### 3.2 Import Your Project
1. From the Vercel dashboard, click **Add New...** → **Project**
2. Find your `dcgan-demo` repository and click **Import**

### 3.3 Configure the Project
Fill in these settings:

| Setting | Value |
|---------|-------|
| **Framework Preset** | Vite |
| **Root Directory** | Click **Edit** → type `frontend` → **Continue** |
| **Build Command** | `npm run build` (default) |
| **Output Directory** | `dist` (default) |

### 3.4 Add Environment Variables
Expand **Environment Variables** and add:

| Key | Value |
|-----|-------|
| `VITE_API_URL` | `https://dcgan-backend-xxxx.onrender.com` (your Render URL) |
| `VITE_WS_URL` | `wss://dcgan-backend-xxxx.onrender.com/ws` (same URL with wss://) |

**Important:**
- Use `https://` for the API URL
- Use `wss://` (not `ws://`) for the WebSocket URL in production

### 3.5 Deploy
1. Click **Deploy**
2. Wait for the build to complete (2-3 minutes)
3. Once deployed, you'll see a URL like: `https://dcgan-demo-xxxx.vercel.app`

---

## Step 4: Update Render with Frontend URL

Now that you have your Vercel URL, update Render:

1. Go to your Render dashboard
2. Click on your `dcgan-backend` service
3. Go to **Environment** tab
4. Update the `FRONTEND_URL` variable with your Vercel URL:
   - `https://dcgan-demo-xxxx.vercel.app`
5. Click **Save Changes**
6. Render will automatically redeploy

---

## Step 5: Test Your Deployment

1. Open your Vercel URL in a browser
2. You should see the DCGAN Demo interface
3. The connection status should show **Connected**
4. Try generating images (training will be slow on free tier - this is normal!)

---

## Troubleshooting

### "Disconnected" Status
- **Cause:** WebSocket connection failed
- **Fix:** Make sure `VITE_WS_URL` uses `wss://` (not `ws://`)

### CORS Errors in Console
- **Cause:** Frontend URL not in allowed origins
- **Fix:** Check that `FRONTEND_URL` in Render matches your exact Vercel URL

### Backend Build Fails
- **Cause:** Usually PyTorch installation issues
- **Fix:** The requirements.txt uses CPU-only PyTorch. If still failing, try removing version numbers.

### Slow Training
- **This is expected!** Free tier has no GPU
- Training will take 20-50 minutes per epoch
- **Tip:** Train models locally, save checkpoints, and deploy pre-trained models

### Service Goes to Sleep (Render Free Tier)
- Free tier services sleep after 15 minutes of inactivity
- First request after sleep takes ~30 seconds to wake up
- **Tip:** Use a service like UptimeRobot to ping your backend every 14 minutes

---

## Optional: Add Pre-trained Models

To provide a better experience, you can include pre-trained models:

1. Train models locally (MNIST and Fashion-MNIST)
2. Save them using the UI
3. Copy the `backend/checkpoints/` folder to your repo
4. Commit and push:
```bash
git add backend/checkpoints/
git commit -m "Add pre-trained models"
git push
```
5. Render will automatically redeploy with the models

---

## Summary of URLs

After deployment, you'll have:

| Service | URL |
|---------|-----|
| Backend API | `https://dcgan-backend-xxxx.onrender.com` |
| Frontend App | `https://dcgan-demo-xxxx.vercel.app` |
| WebSocket | `wss://dcgan-backend-xxxx.onrender.com/ws` |

---

## Cost

Both services are **free** with these limitations:
- **Render Free Tier:** 750 hours/month, sleeps after 15 min inactivity
- **Vercel Free Tier:** 100 GB bandwidth/month, unlimited deployments

For a demo/educational project, free tier is perfect!

---

## Need Help?

- Render Docs: https://render.com/docs
- Vercel Docs: https://vercel.com/docs
- Open an issue on the GitHub repo
