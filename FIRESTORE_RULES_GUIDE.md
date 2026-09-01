# Firestore Security Rules - Deployment Guide

## File Location
- Local: `firestore.rules` (in project root)
- Firebase Project: `cresco-scientiam`

## What These Rules Do

### 1. **Admin-Only Access**
- Only `aaravhfs@gmail.com` can write/modify/delete any data
- Admin can read all collections

### 2. **Collection-Level Permissions**

| Collection | Authenticated Users | Admin Only |
|---|---|---|
| `admins` | ❌ No access | ✅ Read/Write |
| `settings` | ✅ Read only | ✅ Read/Write |
| `participants` | ✅ Read only | ✅ Read/Write/Delete |
| `logs` | ✅ Read only | ✅ Read/Write/Delete |
| `backups` | ❌ No access | ✅ Read/Write/Delete |
| `events/*` | ✅ Read only | ✅ Read/Write/Delete |

### 3. **Security Features**
- ✅ Email-based authorization (admin email hardcoded)
- ✅ All unauthenticated access blocked
- ✅ Write protection on sensitive data
- ✅ Logging of all admin operations
- ✅ Default-deny policy (explicitly allow only what's needed)

---

## Deployment Steps

### Method 1: Using Firebase CLI (Recommended)

1. **Install Firebase CLI** (if not already installed)
   ```bash
   npm install -g firebase-tools
   ```

2. **Login to Firebase**
   ```bash
   firebase login
   ```

3. **Initialize/Select Project**
   ```bash
   firebase init
   # Select: Firestore
   # Select project: cresco-scientiam
   ```

4. **Deploy Rules**
   ```bash
   firebase deploy --only firestore:rules
   ```

5. **Verify Deployment**
   - Go to Firebase Console → Firestore → Rules tab
   - Rules should show as "Published" with today's date

### Method 2: Using Firebase Console (Manual)

1. Open [Firebase Console](https://console.firebase.google.com/)
2. Select project: `cresco-scientiam`
3. Navigate to **Firestore Database** → **Rules** tab
4. Click **Edit Rules**
5. Copy contents of `firestore.rules` and paste
6. Click **Publish**

---

## Testing Rules

### Before Publishing
Use Firebase Emulator to test rules locally:

```bash
firebase emulators:start --only firestore
```

### After Publishing
Test via Firebase Console:
1. Go to **Firestore → Data**
2. Try to add/edit documents (should be blocked if not admin)
3. Check browser console in admin dashboard for permission errors

---

## Admin Email Configuration

**Current Admin Email**: `aaravhfs@gmail.com`

### To Change Admin Email
1. Open `firestore.rules`
2. Find line: `return request.auth.token.email == 'aaravhfs@gmail.com';`
3. Replace `aaravhfs@gmail.com` with new admin email
4. Deploy with: `firebase deploy --only firestore:rules`

### To Add Multiple Admins
Replace the `isAdmin()` function with:
```javascript
function isAdmin() {
  return request.auth.token.email in [
    'aaravhfs@gmail.com',
    'another-admin@gmail.com',
    'third-admin@gmail.com'
  ];
}
```

---

## Troubleshooting

### "Permission Denied" Error
- Verify user is logged in with `aaravhfs@gmail.com`
- Check Firebase Console → Authentication to see current user
- Clear browser cache and re-login

### Rules Won't Deploy
1. Check Firebase CLI version: `firebase --version`
2. Update if needed: `npm install -g firebase-tools@latest`
3. Verify syntax: `firebase deploy --only firestore:rules --debug`

### Firestore Operations Not Working in App
1. Open browser console (F12)
2. Look for errors from Firebase SDK
3. Check Firebase Console → Usage/Errors for API logs
4. Verify app is using correct `authDomain` and `projectId`

---

## Rules Structure Explained

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Define helper functions
    function isAdmin() { ... }
    function isAuthenticated() { ... }
    
    // Define per-collection rules
    match /collection/{document} {
      allow read/write: if condition;
    }
    
    // Default deny all (security best practice)
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

---

## Collections Currently Used

1. **admins** - Stores authorized admin emails (document ID = email)
2. **settings** - Event-wide settings (event name, current day, etc.)
3. **participants** - Participant data with QR IDs and attendance
4. **logs** - Activity logs for all operations
5. **backups** - Backup files (automatic or manual)
6. **events** - Event-specific data with sub-collections

---

## Security Best Practices Implemented

✅ **Principle of Least Privilege**: Users only get minimum required permissions
✅ **Default Deny**: All access denied unless explicitly allowed
✅ **Email Validation**: Authentication based on verified Google email
✅ **Separation of Concerns**: Admin operations isolated from user reads
✅ **Data Protection**: Sensitive collections (admins, backups) admin-only
✅ **Audit Trail**: Logs collection tracks all operations

---

## Next Steps

1. ✅ Review rules above
2. 📋 Choose deployment method (CLI or Console)
3. 🚀 Deploy to Firebase
4. ✅ Test with your admin account
5. 🔒 Verify deny rules work (try unauthorized email)

**Questions or Issues?** Check Firebase documentation: https://firebase.google.com/docs/firestore/security/start
