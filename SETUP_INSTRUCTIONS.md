# INDlop Project - Setup Instructions

## Overview
This is a complete React + Vite + Firebase project for INDlop. The project features a public landing website and a Firestore-based admin panel with role-based access control.

## Project Structure
```
indlop/
├── client/                 # React frontend
│   ├── public/            # Static assets
│   ├── src/
│   │   ├── pages/         # Page components (Home, Contact, Admin, etc.)
│   │   ├── components/    # Reusable UI components
│   │   ├── contexts/      # React contexts (Auth, Theme)
│   │   ├── lib/           # Utilities (Firebase config)
│   │   ├── App.tsx        # Main app component with routing
│   │   ├── main.tsx       # Entry point
│   │   └── index.css      # Global styles
│   └── index.html         # HTML template
├── server/                # Express server (for production)
├── package.json           # Dependencies
├── vite.config.ts         # Vite configuration
├── tsconfig.json          # TypeScript configuration
└── ideas.md              # Design system documentation
```

## Prerequisites
- Node.js 18+ and npm/pnpm
- Firebase project (already configured with indiacast-72ee2)
- Firestore database enabled in Firebase

## Installation & Setup

### 1. Install Dependencies
```bash
cd indlop
pnpm install
# or
npm install
```

### 2. Configure Firebase
The Firebase configuration is already set in `client/src/lib/firebase.ts` using your project:
- Project ID: `indiacast-72ee2`
- Auth Domain: `indiacast-72ee2.firebaseapp.com`
- Database URL: `https://indiacast-72ee2-default-rtdb.firebaseio.com`

No additional configuration needed for Firebase setup.

### 3. Set Up Firestore Admin Collection
To enable admin access for your account, you need to create a Firestore document:

#### Step 1: Enable Firestore Database
1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project: `indiacast-72ee2`
3. Go to **Firestore Database** (left sidebar)
4. Click **Create Database**
5. Choose **Start in production mode**
6. Select region (e.g., `us-central1`)
7. Click **Create**

#### Step 2: Create Admin Document
1. In Firestore, click **Create collection**
2. Collection name: `admins`
3. Click **Next**
4. Create document with:
   - Document ID: Your Firebase UID (see below how to find it)
   - Field 1: `role` (string) = `admin`
   - Field 2: `email` (string) = `smmshopsp.in@gmail.com`
   - Field 3: `createdAt` (timestamp) = current timestamp

#### Step 3: Find Your Firebase UID
1. Go to Firebase Console → **Authentication**
2. Click on your user (smmshopsp.in@gmail.com)
3. Copy the **User UID** from the details panel
4. Use this UID as the Document ID in the `admins` collection

### 4. Set Up Firestore Security Rules
To protect the admin collection, update Firestore rules:

1. Go to Firestore Database → **Rules** tab
2. Replace the rules with:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow anyone to read public data
    match /public/{document=**} {
      allow read: if true;
    }
    
    // Protect admin collection - only admins can read/write
    match /admins/{uid} {
      allow read: if request.auth.uid == uid;
      allow write: if request.auth.uid == uid && get(/databases/$(database)/documents/admins/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Default: deny all other access
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```
3. Click **Publish**

## Running the Project

### Development Mode
```bash
pnpm dev
# or
npm run dev
```
The app will run at `http://localhost:3000`

### Build for Production
```bash
pnpm build
# or
npm run build
```

### Preview Production Build
```bash
pnpm preview
# or
npm run preview
```

## Project Features

### Public Website (No Login Required)
- **Home Page** (`/`): Landing page with hero, features, about sections
- **Contact Page** (`/contact`): Contact form for inquiries
- **Privacy Policy** (`/privacy`): Privacy policy page
- **Terms of Service** (`/terms`): Terms and conditions
- **Navigation**: Sticky header with responsive menu

### Admin Panel (Firestore Role-Based Access)
- **Login** (`/admin`): Email/password authentication
- **Dashboard** (`/admin/dashboard`): Admin overview (protected)
- **Access Control**: Automatically checks Firestore `admins` collection for user role

## Authentication Flow

1. User enters email and password on `/admin`
2. Firebase Authentication verifies credentials
3. On successful login, AuthContext checks Firestore `admins` collection
4. If user document exists with `role: "admin"`, access is granted
5. If no admin document, user is redirected to login
6. Dashboard is protected and only accessible to verified admins

## Adding More Admins
To add another admin user:

1. Have the user sign up on `/admin` (they'll be created in Firebase Auth)
2. Get their Firebase UID from Firebase Console → Authentication
3. In Firestore, create a new document in the `admins` collection with:
   - Document ID: Their UID
   - `role`: `admin`
   - `email`: Their email address

## Deployment Options

### Option 1: Vercel (Recommended for React)
```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel
```

### Option 2: Firebase Hosting
```bash
# Install Firebase CLI
npm i -g firebase-tools

# Login to Firebase
firebase login

# Initialize Firebase in project
firebase init hosting

# Deploy
firebase deploy
```

### Option 3: GitHub Pages
Push to GitHub and enable GitHub Pages in repository settings.

## Environment Variables
The project uses environment variables from `vite.config.ts`. No `.env` file needed for Firebase as it's configured in `client/src/lib/firebase.ts`.

## Troubleshooting

### "You do not have admin access" Error
- Check that your user document exists in Firestore `admins` collection
- Verify the document ID matches your Firebase UID
- Ensure the `role` field is set to `"admin"`

### Login Not Working
- Verify Firebase Authentication is enabled
- Check that your email is registered in Firebase Authentication
- Ensure Firestore security rules allow read access to the `admins` collection

### Firestore Connection Issues
- Verify Firestore database is created and active
- Check Firebase project ID in `client/src/lib/firebase.ts`
- Ensure Firestore security rules are properly configured

## Technology Stack
- **Frontend**: React 19 + TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS 4
- **UI Components**: shadcn/ui
- **Animations**: Framer Motion
- **Routing**: Wouter
- **Authentication**: Firebase Auth
- **Database**: Firestore
- **Icons**: Lucide React

## Support
For issues or questions, refer to:
- [Firebase Documentation](https://firebase.google.com/docs)
- [React Documentation](https://react.dev)
- [Vite Documentation](https://vitejs.dev)
- [Tailwind CSS Documentation](https://tailwindcss.com)

## License
MIT
