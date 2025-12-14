# Google Drive OAuth Bridge (Client-Side)

A lightweight, static **OAuth 2.0 Bridge** hosted on GitHub Pages.

This project solves the **"Invalid Origin"** and **"Redirect URI"** issues faced when developing Google Drive integrations inside sandboxed environments like **Google AI Studio**, **StackBlitz**, **CodePen**, or local files without a backend server.

---

## 🛑 The Problem

When you develop an app inside a web-based editor (like Google AI Studio), your code often runs inside a sandboxed iframe with a dynamic URL or a `null` origin.

**Google OAuth 2.0 requires a fixed, public, and verifiable domain** (e.g., `https://username.github.io`). It strictly rejects authentication requests coming from dynamic sandboxes or unverified origins.

## ✅ The Solution

This repository acts as a secure **Auth Bridge** (Middleman):

1. **Your App** opens this GitHub Page in a popup, passing your `client_id` in the URL.
2. **This Bridge** initiates the authentication with Google (using its valid, fixed domain).
3. **Google** redirects back to this Bridge with the Access Token.
4. **This Bridge** securely sends the token back to your App using `window.opener.postMessage` and closes itself.

**No backend required. Zero configuration in the code.**

---

## 🚀 Setup Guide (3 Steps)

### 1. Fork this Repository
Click the **Fork** button at the top right of this page to create your own copy.
1. Go to your new repository's **Settings** > **Pages**.
2. Set the **Source** to `Deploy from a branch` (usually `main` or `master`).
3. Your bridge will be live at: `https://YOUR-USERNAME.github.io/googledrive-bridge/`

### 2. Configure Google Cloud Console
1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create OAuth 2.0 Credentials of type **Web Application**.
3. **Authorized JavaScript Origins:**
   * `https://YOUR-USERNAME.github.io`
4. **Authorized Redirect URIs:**
   * `https://YOUR-USERNAME.github.io/googledrive-bridge/`
   * *(Note: Ensure there are no query parameters here. Just the clean URL).*
5. **Copy your Client ID**.

### 3. Usage in Your App
You do **not** need to edit the HTML code in this repository. Just pass your `client_id` as a query parameter when opening the popup.

#### React Hook Example (`useGoogleAuth.js`)

```javascript
import { useState, useEffect } from 'react';

// 1. Replace with your FORKED URL
const BRIDGE_URL = "[https://YOUR-USERNAME.github.io/googledrive-bridge/](https://YOUR-USERNAME.github.io/googledrive-bridge/)";

// 2. Replace with your Google Client ID
const CLIENT_ID = "YOUR-CLIENT-ID.apps.googleusercontent.com";

export const useGoogleDriveAuth = () => {
  const [token, setToken] = useState(null);

  const login = () => {
    // Center the popup on screen
    const width = 500;
    const height = 600;
    const left = window.screen.width / 2 - width / 2;
    const top = window.screen.height / 2 - height / 2;

    // Pass the Client ID dynamically
    const url = `${BRIDGE_URL}?client_id=${CLIENT_ID}`;

    window.open(
      url, 
      'googleAuth', 
      `width=${width},height=${height},top=${top},left=${left}`
    );
  };

  useEffect(() => {
    const handleMessage = (event) => {
      // Optional: Check event.origin for extra security
      // if (!event.origin.includes('github.io')) return;

      if (event.data.type === 'GOOGLE_AUTH_SUCCESS') {
        console.log('Token received:', event.data.token);
        setToken(event.data.token);
      }
    };

    window.addEventListener('message', handleMessage);
    return () => window.removeEventListener('message', handleMessage);
  }, []);

  return { token, login };
};
````

#### Using the Hook

```jsx
import React from 'react';
import { useGoogleDriveAuth } from './useGoogleDriveAuth';

const App = () => {
  const { token, login } = useGoogleDriveAuth();

  return (
    <div>
      <h1>My App</h1>
      {!token ? (
        <button onClick={login}>Connect Google Drive</button>
      ) : (
        <p>Authenticated! Token: {token.slice(0, 10)}...</p>
      )}
    </div>
  );
};
```

-----

## 🔒 Privacy & Security

  * **Generic Privacy Policy:** A generic `privacy.html` is included in this repository. You can use it as your "Privacy Policy URL" in the Google OAuth Consent Screen configuration:
    `https://YOUR-USERNAME.github.io/googledrive-bridge/privacy.html`
  * **Public Client ID:** It is safe (and necessary) to expose your `Client ID` in frontend code.
  * **No Client Secret:** This implementation uses the Implicit Flow (Token). It does **not** use or store a Client Secret.
  * **Scope:** This bridge defaults to `https://www.googleapis.com/auth/drive.file`, meaning the app can only access files it created itself, protecting the user's personal data.

## 📄 License

MIT License. Feel free to fork and use for your own projects.
