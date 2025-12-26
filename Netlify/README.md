

# 🚀 NETLIFY CI/CD – STEP BY STEP (BEGINNER → DEVOPS)

---

## ✅ STEP 0: Prerequisites

Make sure you have:

* ✅ GitHub repository (your project pushed to `main`)
* ✅ Node.js installed
* ✅ Netlify account
* ✅ Project builds locally (`npm run build` works)

---

## 🟢 STEP 1: Install Netlify CLI

```bash
npm install -g netlify-cli
```

⚠️ Ignore warnings – they are safe.

If `netlify` command does not work on Windows:

```bash
npx netlify --version
```

👉 From now on, **use `npx netlify` if needed**

---

## 🟢 STEP 2: Login to Netlify

```bash
npx netlify login
```

✔ Browser opens
✔ Login succeeds
✔ Come back to terminal

Verify:

```bash
npx netlify status
```

---

## 🟢 STEP 3: Create Netlify Site (Get Site ID)

```bash
npx netlify sites:create --name ai-cloud-community-hub
```

You will see:

```text
Site created
Admin URL: https://app.netlify.com/sites/ai-cloud-community-hub
Site ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

🎯 **Copy the Site ID** (VERY IMPORTANT)

---

## 🟢 STEP 4: Get Netlify Auth Token

```bash
npx netlify auth:token
```

🎯 Copy the token
⚠️ Keep it secret

---

## 🟢 STEP 5: Add Secrets to GitHub

### Go to:

**GitHub Repo → Settings → Secrets and variables → Actions**

Click **New repository secret**

Add these **EXACTLY**:

| Secret Name          | Value              |
| -------------------- | ------------------ |
| `NETLIFY_SITE_ID`    | Site ID you copied |
| `NETLIFY_AUTH_TOKEN` | Auth token         |

✅ Secrets added → Done

---

## 🟢 STEP 6: Create GitHub Actions Workflow

In your repo, create this file:

```text
.github/workflows/netlify-deploy.yml
```

---

## 🟢 STEP 7: Paste This Workflow (Production-Ready)

### 🔹 React (CRA)

```yaml
name: CI/CD – Netlify Deployment

on:
  push:
    branches: [main]

jobs:
  build_deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Build app
        run: npm run build

      - name: Deploy to Netlify
        run: npx netlify deploy --prod --dir=build
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### 🔹 Vite

Change only this line:

```yaml
run: npx netlify deploy --prod --dir=dist
```

---

## 🟢 STEP 8: Commit & Push

```bash
git add .
git commit -m "Add Netlify CI/CD pipeline"
git push origin main
```

---

## 🟢 STEP 9: Verify Deployment

1. Go to **GitHub → Actions**
2. Open the workflow
3. Watch steps run
4. ✅ Deployment successful

---

## 🟢 STEP 10: Open Your Live Site

```text
https://ai-cloud-community-hub.netlify.app
```

(or your Netlify-generated domain)

🎉 **AUTO DEPLOYMENT IS LIVE**

---

# 🔁 WHAT HAPPENS NOW (IMPORTANT)

* Every push to `main` → 🔄 auto deploy
* No manual Netlify upload needed
* Secure secrets
* Production-grade CI/CD

---

## 🧠 DEVOPS BEST PRACTICES (FOR YOU)

* 🔐 Never commit tokens
* 🔁 Rotate auth token if leaked
* 🌿 Use preview deploys for PRs
* 🌐 Add custom domain later

---

## 🔥 NEXT THINGS I CAN SET UP FOR YOU

1. ✅ Preview deploys on Pull Requests
2. 🌐 Custom domain via Netlify
3. 🔄 Monorepo support
4. 🧪 Lint + test stages
5. 🔐 Environment variables
6. 📦 Backend + Netlify Functions

👉 **Tell me what you want next and I’ll guide you step by step.**


## Step 1: Set Up GitHub Secrets

GitHub Secrets are secure environment variables used by your workflow. Here's how to set them up:

### 1.1 Get Netlify Credentials

**a) Get your Netlify Auth Token:**
1. Go to [Netlify Dashboard](https://app.netlify.com)
2. Click your profile picture → **User settings**
3. Go to **Applications** → **Personal access tokens**
4. Click **New access token**
5. Name it (e.g., "GitHub Actions")
6. Copy the token and save it temporarily

**b) Get your Netlify Site ID:**
1. Go to [Netlify Dashboard](https://app.netlify.com)
2. Click on your site
3. Go to **Site settings** → **General**
4. Find **Site ID** - copy it

### 1.2 Add Secrets to GitHub Repository

1. Go to your GitHub repository
2. Click **Settings** tab
3. In the left sidebar, click **Secrets and variables** → **Actions**
4. Click **New repository secret**

**Create two secrets:**

| Secret Name | Value |
|---|---|
| `NETLIFY_AUTH_TOKEN` | Your Netlify personal access token |
| `NETLIFY_SITE_ID` | Your Netlify site ID |

For each secret:
- Click **New repository secret**
- Enter the name exactly as shown
- Paste the value
- Click **Add secret**

✅ **Done!** You should see both secrets listed now.
