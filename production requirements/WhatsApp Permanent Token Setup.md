# 🔐 WhatsApp Permanent Token Setup (Production)

[![Security](https://img.shields.io/badge/Security-Critical-red.svg)]()
[![Environment](https://img.shields.io/badge/Environment-Production-blue.svg)]()
[![Meta](https://img.shields.io/badge/Provider-Meta%20Business-blue.svg)]()

> **Guide to generating a non-expiring System User Token for the SafeCast production environment.** > _Note: Temporary developer tokens expire in 24 hours. For production, you must follow this procedure._

---

## 📋 Table of Contents

- [Step 1: Access Business Settings](#-step-1-navigate-to-meta-business-settings)
- [Step 2: Create System User](#-step-2-create-system-user)
- [Step 3: Assign Assets](#-step-3-assign-assets-to-system-user)
- [Step 4: Generate Token](#-step-4-generate-permanent-token)
- [Step 5: Configuration](#-step-5-update-environment-variables)
- [Security Notes](#-security--best-practices)

---

## ⚙️ Step 1: Navigate to Meta Business Settings

1. Go to **[Meta Business Settings](https://business.facebook.com/settings)**.
2. Log in with your Admin Meta account.
3. **Check Portfolio:** Ensure you are in the correct Business Portfolio (check the dropdown in the top-left corner).

---

## 👤 Step 2: Create System User

1. On the left sidebar, navigate to **Users** → **System Users**.
2. Click the **"Add"** button (top right).
3. **Configure User:**
   - **System Username:** `[YourAppName]-Bot` (e.g., `SafeCast-Bot`)
   - **System User Role:** `Admin`
4. Click **Create System User**.

---

## 📦 Step 3: Assign Assets to System User

Once the user is created, select it from the list and click the **Add Assets** button. You must assign **two** types of assets:

### A. Assign the App

1. Select **Apps** in the asset type menu.
2. Find and check your **WhatsApp Business App**.
3. Toggle permissions: **Full control** → **Manage app**.

### B. Assign the WhatsApp Account

1. Select **WhatsApp accounts** in the asset type menu.
2. Find and check your **WhatsApp Business Account**.
3. Toggle permissions: **Full control** → **Everything**.

Click **Assign Assets** to finalize.

---

## 🎫 Step 4: Generate Permanent Token

With the System User selected, click the **Generate New Token** button.

### Configuration Wizard:

1. **Select App:** Choose your WhatsApp Business App from the dropdown.
2. **Expiration:** Select **Never** ⚠️.
   - _(Do NOT select the 60-day option, or your production app will break in two months.)_
3. **Permissions:** You must check the following boxes:
   - ✅ `whatsapp_business_messaging`
   - ✅ `whatsapp_business_management`

### Finalize:

1. Click **Generate Token**.
2. **COPY IMMEDIATELY:** The token is shown **once**.

> 🛑 **CRITICAL WARNING:**
> If you close this window without copying the token, it is lost forever. You will have to revoke it and generate a new one.

---

## 📝 Step 5: Update Environment Variables

Add the permanent token to your production `.env` file (or your cloud provider's secret manager).

```bash
# Production Token Configuration
WHATSAPP_SYSTEM_USER_TOKEN=EAAB...<PASTE_YOUR_PERMANENT_TOKEN_HERE>

# Map to the variable your app uses
WHATSAPP_API_TOKEN=${WHATSAPP_SYSTEM_USER_TOKEN}

# Additional Identifiers (Found in Business Settings)
WHATSAPP_PHONE_ID=109...
WHATSAPP_BUSINESS_ACCOUNT_ID=115...

```

| Item           | Details                                                                                          |
| -------------- | ------------------------------------------------------------------------------------------------ |
| System User ID | The numerical ID (e.g., 6158...) is NOT used in the `.env` file. Only the token is required.     |
| Scope          | One System User Token can handle both messaging and management if permissions are set correctly. |
| Security       | Treat this token like a root password. If leaked, attackers can send messages as your business.  |
| Isolation      | Use separate Business Accounts and tokens for Staging and Production environments.               |
