# Code Changes: HTTP to HTTPS Migration

## Overview
After setting up HTTPS on your EC2 instance, you need to update all URLs in your code from HTTP to HTTPS.

---

## 1. Update Environment Variables

### Local: `.env.production`

```bash
# Environment Configuration for Production

# Backend API URL - AWS EC2 Server with HTTPS
REACT_APP_API_URL=https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001

# Socket Server URL - AWS EC2 Server with HTTPS
REACT_APP_SOCKET_URL=https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001

# Or if you bought a domain:
# REACT_APP_API_URL=https://api.yourdomain.com
# REACT_APP_SOCKET_URL=https://api.yourdomain.com
```

### Local: `.env.local` (for development - keep as is)

```bash
# Keep these as HTTP for local development
REACT_APP_API_URL=http://localhost:3001
REACT_APP_SOCKET_URL=http://localhost:3001
```

---

## 2. Update GitHub Secrets

Go to: https://github.com/RushiK8626/Chat-Messaging-App/settings/secrets/actions

Update both **Repository secrets** AND **Environment secrets** (github-pages):

1. **REACT_APP_API_URL**
   - Old value: `http://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001`
   - New value: `https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001`

2. **REACT_APP_SOCKET_URL**
   - Old value: `http://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001`
   - New value: `https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001`

---

## 3. Files That Will Automatically Update

Since you're using environment variables, these files will automatically use HTTPS once you update the secrets:

✅ `src/utils/apiClient.js` - Uses `process.env.REACT_APP_API_URL`
✅ `src/utils/socket.js` - Uses `process.env.REACT_APP_SOCKET_URL`
✅ `src/App.js` - Uses environment variables
✅ `src/pages/ChatHome.js` - Uses environment variables

---

## 4. Files with Hardcoded URLs (Need Manual Update)

Check these files for any hardcoded HTTP URLs:

### `src/pages/Settings.js`

**Before:**
```javascript
await fetch(`${process.env.REACT_APP_API_URL || "http://localhost:3001"}/api/auth/logout`, {
```

**After:**
```javascript
// Already using environment variable - no change needed!
// Just make sure fallback is correct for development
await fetch(`${process.env.REACT_APP_API_URL || "http://localhost:3001"}/api/auth/logout`, {
```

### `src/pages/Profile.js`

All instances already use environment variables - no changes needed!

### `src/components/ChatInfoModal.js`

**Lines 78 and 129 have template literal syntax errors - fix these:**

**Before (Line 78):**
```javascript
const refreshRes = await fetch('${process.env.REACT_APP_API_URL || "http://localhost:3001"}/api/auth/refresh-token', {
```

**After (Line 78):**
```javascript
const refreshRes = await fetch(`${process.env.REACT_APP_API_URL || "http://localhost:3001"}/api/auth/refresh-token`, {
```

**Before (Line 129):**
```javascript
const refreshRes = await fetch('${process.env.REACT_APP_API_URL || "http://localhost:3001"}/api/auth/refresh-token', {
```

**After (Line 129):**
```javascript
const refreshRes = await fetch(`${process.env.REACT_APP_API_URL || "http://localhost:3001"}/api/auth/refresh-token`, {
```

---

## 5. Backend Server Configuration (if needed)

### If using Nginx as reverse proxy, update backend CORS settings

Your Node.js backend may need to accept requests from GitHub Pages:

**In your backend code (likely in `server.js` or `app.js`):**

```javascript
const cors = require('cors');

app.use(cors({
  origin: [
    'http://localhost:3000',  // Local development
    'https://rushik8626.github.io'  // GitHub Pages production
  ],
  credentials: true
}));
```

### If using Socket.IO, update CORS:

```javascript
const io = require('socket.io')(server, {
  cors: {
    origin: [
      'http://localhost:3000',
      'https://rushik8626.github.io'
    ],
    credentials: true
  }
});
```

---

## 6. Deployment Steps After HTTPS Setup

### Step 1: Update `.env.production` locally

```bash
# Change http:// to https:// in .env.production
REACT_APP_API_URL=https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001
REACT_APP_SOCKET_URL=https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001
```

### Step 2: Update GitHub Secrets

Update both Repository secrets AND Environment secrets (github-pages) with HTTPS URLs.

### Step 3: Trigger deployment

```bash
git add .
git commit -m "Update to HTTPS URLs"
git push origin main
```

### Step 4: Wait for deployment

1. Monitor GitHub Actions: https://github.com/RushiK8626/Chat-Messaging-App/actions
2. Wait 2-3 minutes for build
3. Wait 5-10 minutes for CDN propagation

### Step 5: Test in production

Open in incognito: https://rushik8626.github.io/Chat-Messaging-App

You should see:
```
✅ Socket URL: https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001
✅ Environment: production
✅ Server URL: https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001
✅ No Mixed Content errors!
```

---

## 7. Port Configuration

### If using Nginx as reverse proxy:

- Your backend still runs on port **3001** internally
- Nginx listens on port **443** (HTTPS) and **80** (HTTP)
- Nginx forwards traffic to `localhost:3001`
- Your URLs should NOT include the port: `https://your-domain.com` (not `:3001`)

**Update your environment variables to:**
```bash
REACT_APP_API_URL=https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com
REACT_APP_SOCKET_URL=https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com
```

Note: **No `:3001` port number!**

### If NOT using Nginx (direct Node.js HTTPS):

You need to configure your Node.js backend to support HTTPS directly:

```javascript
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('/path/to/private.key'),
  cert: fs.readFileSync('/path/to/certificate.crt')
};

https.createServer(options, app).listen(3001, () => {
  console.log('HTTPS Server running on port 3001');
});
```

Then keep the port in your URLs:
```bash
REACT_APP_API_URL=https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001
REACT_APP_SOCKET_URL=https://ec2-35-154-111-148.ap-south-1.compute.amazonaws.com:3001
```

---

## 8. Testing Checklist

After making all changes, test:

- [ ] Login/Registration works
- [ ] Socket.IO connects successfully
- [ ] No Mixed Content errors in console
- [ ] No CORS errors
- [ ] Messages send/receive in real-time
- [ ] File uploads work
- [ ] Profile pictures load correctly
- [ ] All API calls work

---

## 9. Troubleshooting

### Mixed Content errors persist:
- Clear browser cache completely
- Open in incognito window
- Verify GitHub Secrets are updated with HTTPS
- Check that new build deployed successfully

### CORS errors:
- Update backend CORS configuration to allow GitHub Pages origin
- Make sure credentials: true is set

### Socket.IO connection fails:
- Check Socket.IO CORS settings in backend
- Verify WebSocket support in Nginx config (if using proxy)
- Check EC2 Security Group allows port 443

### Certificate errors (self-signed):
- Click "Advanced" in browser
- Click "Proceed to site" (only for testing!)
- Or use a proper domain with Let's Encrypt

---

## Summary of Changes

1. ✅ Update `.env.production`: `http://` → `https://`
2. ✅ Update GitHub Secrets (both Repository & Environment): `http://` → `https://`
3. ✅ Fix template literal syntax in `ChatInfoModal.js` (lines 78, 129)
4. ✅ Update backend CORS to allow GitHub Pages
5. ✅ Test thoroughly in production

**Most important:** Since you're using environment variables everywhere, you only need to update the secrets and `.env.production` file!
