# How to Get Cloudflare Turnstile Keys (Site Key & Secret Key)

## Prerequisites

- Cloudflare account (free tier works)
- Access to Cloudflare Dashboard

---

## Step 1: Log in to Cloudflare Dashboard

1. Go to [https://dash.cloudflare.com/](https://dash.cloudflare.com/)
2. Sign in with your Cloudflare credentials

---

## Step 2: Navigate to Turnstile

1. In the **left sidebar**, look for **"Protect & Connect"** section
2. Under **"Application security"**, click **"Turnstile"**
3. Or directly navigate to: `https://dash.cloudflare.com/[your-account-id]/turnstile`

---

## Step 3: Add a New Turnstile Widget

1. Click **"Add Site"** or **"Add Widget"** button (usually in the top-right)
2. You'll be taken to the **"Add Widget"** configuration page

---

## Step 4: Configure Widget Settings

### a) Widget Name

- Enter a descriptive name (e.g., `SafeCast`, `SafeCast Production`, `MyApp-Bot-Protection`)
- This helps identify the widget if you have multiple

### b) Hostname Management

- Click **"Add Hostnames"** button
- Add domains/hostnames where Turnstile will be used:
  - **Local development**: `localhost`, `127.0.0.1`
  - **Production**: `yourdomain.com`, `www.yourdomain.com`
  - You can add multiple hostnames (up to 10 available)

### c) Widget Mode (Optional)

- **Managed**: Cloudflare automatically chooses best challenge (recommended)
- **Non-interactive**: Invisible challenge, best UX
- **Invisible**: No user interaction needed

### d) Additional Settings

- Usually defaults are fine
- Configure as per your security requirements

---

## Step 5: Create the Widget

1. Click **"Create"** or **"Add"** button at the bottom
2. Wait for success message: **"Successfully created Turnstile widget!"**

---

## Step 6: Copy Your Keys

After successful creation, you'll see **two keys**:

### ✅ Site Key (Public Key)

- **Format**: `0x4AAAAAACE1wIaICOHSKTWl` (example)
- **Usage**: Frontend/client-side code
- **Security**: Safe to expose publicly in HTML/JavaScript
- Click **"Click to copy"** to copy this key

### ✅ Secret Key (Private Key)

- **Format**: `0x4AAAAAACE1wIg9eztw2T6ZTLPm802AYIU` (example, longer)
- **Usage**: Backend/server-side verification ONLY
- **Security**: ⚠️ MUST be kept secret - NEVER expose in frontend
- Click **"Click to copy"** to copy this key

---

## Step 7: Add Keys to Your Project

### For Backend (.env file)

```env
# Cloudflare Turnstile Keys
CLOUDFLARE_TURNSTILE_SITE_KEY=0x4AAAAAACE1wIaICOHSKTWl
CLOUDFLARE_TURNSTILE_SECRET_KEY=0x4AAAAAACE1wIg9eztw2T6ZTLPm802AYIU
```

### For Frontend (Client-side)

**HTML Example:**

```html
<!-- Load Turnstile Script -->
<script
  src="https://challenges.cloudflare.com/turnstile/v0/api.js"
  async
  defer
></script>

<!-- Turnstile Widget -->
<div class="cf-turnstile" data-sitekey="YOUR_SITE_KEY_HERE"></div>
```

**React/Next.js Example:**

```jsx
import Turnstile from "@marsidev/react-turnstile";

<Turnstile siteKey="YOUR_SITE_KEY_HERE" />;
```

---

## Backend Verification (Node.js Example)

### Verification Endpoint

```
POST https://challenges.cloudflare.com/turnstile/v0/siteverify
```

### Payload

```json
{
  "secret": "YOUR_SECRET_KEY",
  "response": "TOKEN_FROM_CLIENT",
  "remoteip": "USER_IP_ADDRESS" // optional
}
```

### Example Code

```javascript
const verifyTurnstile = async (token, userIP) => {
  try {
    const response = await fetch(
      "https://challenges.cloudflare.com/turnstile/v0/siteverify",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          secret: process.env.CLOUDFLARE_TURNSTILE_SECRET_KEY,
          response: token,
          remoteip: userIP, // optional
        }),
      }
    );

    const data = await response.json();
    return data.success; // true if verification passed
  } catch (error) {
    console.error("Turnstile verification failed:", error);
    return false;
  }
};

// Usage in Express route
app.post("/api/submit", async (req, res) => {
  const { turnstileToken } = req.body;
  const userIP = req.ip;

  const isValid = await verifyTurnstile(turnstileToken, userIP);

  if (!isValid) {
    return res.status(400).json({ error: "Bot verification failed" });
  }

  // Proceed with form submission
  // ...
});
```

---

## Important Security Notes

### 🔒 Secret Key Security

1. **NEVER** commit Secret Key to version control (GitHub, GitLab, etc.)
2. Store in environment variables (`.env` file)
3. Add `.env` to `.gitignore`
4. Only use Secret Key on **backend/server-side**
5. Never expose in frontend JavaScript or HTML

### ✅ Site Key Usage

1. Safe to use in frontend/client-side code
2. Can be visible in browser's HTML source
3. Public exposure is expected and safe

### 🔄 Key Rotation

1. If Secret Key is compromised, regenerate immediately
2. Go to: Turnstile → Select your widget → Settings → Regenerate keys

---

## Troubleshooting

### 1. "Invalid site key" Error

**Cause**: Using wrong key or domain not allowed

**Solutions**:

- Verify you're using **Site Key** (not Secret Key) in frontend
- Check if your domain is added to allowed hostnames
- Go to Turnstile → Your widget → Add Hostnames

### 2. "Verification failed" Error

**Cause**: Backend verification issue

**Solutions**:

- Verify you're using **Secret Key** (not Site Key) in backend
- Check if token has expired (tokens are single-use and short-lived)
- Ensure you're calling the verification endpoint correctly

### 3. "Hostname not allowed" Error

**Cause**: Domain not in allowed list

**Solutions**:

- Add your domain/localhost to widget's hostname list
- Go to: Turnstile → Your widget → Hostname Management → Add Hostnames
- For local development, add: `localhost`, `127.0.0.1`

### 4. Widget Not Rendering

**Cause**: Script not loaded or site key incorrect

**Solutions**:

- Check browser console for errors
- Verify Turnstile script is loaded: `https://challenges.cloudflare.com/turnstile/v0/api.js`
- Ensure `data-sitekey` attribute has correct Site Key
- Check network tab for failed requests

---

## Integration Resources

### Official Documentation

- **Main Docs**: [https://developers.cloudflare.com/turnstile/](https://developers.cloudflare.com/turnstile/)
- **Client-side Integration**: [https://developers.cloudflare.com/turnstile/get-started/client-side-rendering/](https://developers.cloudflare.com/turnstile/get-started/client-side-rendering/)
- **Server-side Verification**: [https://developers.cloudflare.com/turnstile/get-started/server-side-validation/](https://developers.cloudflare.com/turnstile/get-started/server-side-validation/)

### Libraries

- **React**: `@marsidev/react-turnstile`
- **Vue**: `@f3ve/vue-turnstile`
- **Svelte**: `svelte-turnstile`

---

## Quick Reference

| Item                      | Value                                                     |
| ------------------------- | --------------------------------------------------------- |
| **Dashboard URL**         | https://dash.cloudflare.com/                              |
| **Turnstile Section**     | Application security → Turnstile                          |
| **Verification Endpoint** | https://challenges.cloudflare.com/turnstile/v0/siteverify |
| **Script CDN**            | https://challenges.cloudflare.com/turnstile/v0/api.js     |
| **Site Key Usage**        | ✅ Frontend (Public)                                      |
| **Secret Key Usage**      | ✅ Backend Only (Private)                                 |
| **Token Lifetime**        | Single-use, short-lived (5 minutes)                       |
| **Max Hostnames**         | 10 per widget                                             |

---

## Environment Variables Template

Add to your `.env` file:

```env
# ============================================
# CLOUDFLARE TURNSTILE CONFIGURATION
# ============================================

# Site Key (Public - used in frontend)
CLOUDFLARE_TURNSTILE_SITE_KEY=your_site_key_here

# Secret Key (Private - used in backend ONLY)
CLOUDFLARE_TURNSTILE_SECRET_KEY=your_secret_key_here

# Verification Endpoint (no need to change)
CLOUDFLARE_TURNSTILE_VERIFY_URL=https://challenges.cloudflare.com/turnstile/v0/siteverify
```

---

## What is Cloudflare Turnstile?

Cloudflare Turnstile is a **free, privacy-friendly CAPTCHA alternative** that:

- ✅ Protects websites from bots and spam
- ✅ Better user experience than traditional CAPTCHAs
- ✅ No frustrating image puzzles
- ✅ Privacy-focused (no tracking)
- ✅ Works invisibly for most users
- ✅ Free for unlimited use
- ✅ Easy integration with any website

---

**Last Updated**: December 2025  
**Project**: SafeCast  
**Status**: Production Ready ✅
