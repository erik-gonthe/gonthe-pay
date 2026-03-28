# Gonthe Pay — Setup Guide
# Guía de instalación

---

## STEP 1 — Create a Firebase project
## PASO 1 — Crear proyecto en Firebase

1. Go to https://firebase.google.com
2. Click "Get started" → "Create a project"
3. Name it: gonthe-pay (or any name)
4. Disable Google Analytics (not needed)
5. Click "Create project"

---

## STEP 2 — Enable Google Sign-In
## PASO 2 — Activar inicio de sesión con Google

1. In Firebase Console → left menu → "Authentication"
2. Click "Get started"
3. Under "Sign-in method" → click "Google"
4. Enable it → enter your email as support email → Save

---

## STEP 3 — Create the Firestore database
## PASO 3 — Crear la base de datos

1. Left menu → "Firestore Database"
2. Click "Create database"
3. Choose "Start in production mode" → Next
4. Select location: us-central (or closest to Texas)
5. Click "Enable"
6. Go to "Rules" tab and paste this:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Users collection - authenticated users can read their own doc
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
      allow read: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
      allow delete: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // First user auto-creation
    match /users/{userId} {
      allow create: if request.auth != null && request.auth.uid == userId;
    }
    
    // Authorized emails - admin only
    match /authorized_emails/{email} {
      allow read, write: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Payments - authenticated users can create, admins see all
    match /payments/{paymentId} {
      allow create: if request.auth != null;
      allow read: if request.auth != null && (
        resource.data.createdBy == request.auth.uid ||
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin'
      );
      allow delete: if request.auth != null && (
        resource.data.createdBy == request.auth.uid ||
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin'
      );
    }
    
    // Catalog - anyone authenticated can read, only admins can write
    match /catalog/{category}/items/{itemId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Meta (payment counter) - authenticated users
    match /meta/{doc} {
      allow read, write: if request.auth != null;
    }
  }
}
```

7. Click "Publish"

---

## STEP 4 — Enable Firebase Storage (for Zelle screenshots)
## PASO 4 — Activar Storage (para las capturas)

1. Left menu → "Storage"
2. Click "Get started" → "Start in production mode" → Next → Done
3. Go to "Rules" tab and paste this:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /receipts/{allPaths=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.resource.size < 5 * 1024 * 1024;
    }
  }
}
```

4. Click "Publish"

---

## STEP 5 — Get your Firebase config keys
## PASO 5 — Obtener las claves de configuración

1. Left menu → click gear icon → "Project settings"
2. Scroll down to "Your apps"
3. Click "</> Web" icon to add a web app
4. Name it: gonthe-pay-web → click "Register app"
5. You'll see a config object like this:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "gonthe-pay.firebaseapp.com",
  projectId: "gonthe-pay",
  storageBucket: "gonthe-pay.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

6. Copy these values

---

## STEP 6 — Paste your keys into the app
## PASO 6 — Pegar las claves en la app

1. Open the file `index.html` in a text editor (Notepad on PC, TextEdit on Mac)
2. Find this section near the bottom:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  ...
```

3. Replace each "YOUR_..." value with your actual values from Step 5
4. Save the file

---

## STEP 7 — Deploy to Firebase Hosting
## PASO 7 — Publicar la app

Option A — Using Firebase CLI (recommended):

1. Install Node.js from https://nodejs.org (LTS version)
2. Open Terminal (Mac) or Command Prompt (Windows)
3. Run: `npm install -g firebase-tools`
4. Run: `firebase login` (opens browser to sign in)
5. Navigate to the gonthe-app folder: `cd path/to/gonthe-app`
6. Run: `firebase init hosting`
   - Select your project
   - Public directory: `.` (just a dot)
   - Single-page app: Yes
   - Don't overwrite index.html: No
7. Run: `firebase deploy`
8. Your app URL will appear: `https://gonthe-pay.web.app`

Option B — Firebase Console drag & drop:
1. Go to Firebase Console → Hosting
2. Click "Get started" → follow prompts
3. Drag and drop your gonthe-app folder

---

## STEP 8 — Add the app to iPhone home screen
## PASO 8 — Agregar al iPhone

1. Open Safari on iPhone
2. Go to your app URL (e.g. https://gonthe-pay.web.app)
3. Tap the Share button (square with arrow)
4. Tap "Add to Home Screen"
5. Name it "Gonthe Pay" → tap Add
6. Done! It appears as an app icon

Share this same link with your employees.
Comparte este mismo link con tus empleados.

---

## USER MANAGEMENT
## Gestión de usuarios

- The FIRST person to sign in automatically becomes Admin
- As Admin, go to Export tab → Manage Users → enter employee email to authorize them
- Employees sign in with Google using their authorized email
- Admin sees all payments; Employees see only their own

---

## ICONS (optional)
## Íconos (opcional)

Place two icon files in the `icons/` folder:
- `icon-192.png` — 192x192 pixels
- `icon-512.png` — 512x512 pixels

You can create a simple truck icon at https://favicon.io or use any image editor.

---

## SUPPORT
For help contact: erik@gonthe.com (or however you prefer)
