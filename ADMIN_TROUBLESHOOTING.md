# Admin Portal - Troubleshooting Guide

## Issue: Admin Portal Loading Infinitely

### Quick Diagnosis Steps

1. **Open Browser Developer Console**
   - Press `F12` on your keyboard
   - Go to **Console** tab
   - Look for any red errors or warning messages
   - Share these errors for debugging

2. **Check Network Tab**
   - Go to **Network** tab in DevTools
   - Refresh the page
   - Look for failed requests (red X)
   - Check if Firebase CDN is loading properly

3. **Look for These Console Messages** (If you see them, auth is working)
   ```
   "Firebase initialized successfully"
   "Auth state changed. User: ..."
   "Authorization successful"
   ```

---

## Common Issues & Solutions

### Issue 1: "Firebase initialized successfully" but no further messages
**Problem**: Firebase loads but auth listener never fires  
**Solution**:
- Clear browser cache (Ctrl+Shift+Delete)
- Clear cookies for this domain
- Try incognito/private window
- Check if you're logged into Google account

### Issue 2: "Auth state changed. User: none"
**Problem**: No user is signed in  
**Solution**:
- Go to `/login.html` first
- Click "Sign in with Google"
- Complete the Google OAuth flow
- Then navigate back to `/admin-x7q/`

### Issue 3: "User not authorized" message
**Problem**: You're logged in but not with the right email  
**Solution**:
- Current authorized email: `aaravhfs@gmail.com`
- Sign out and try with the correct email
- Or contact admin to change authorized email in code

### Issue 4: Network errors or "Connection error"
**Problem**: Can't reach Firebase  
**Solution**:
- Check internet connection
- Try a different browser
- Try disabling VPN/proxy if using one
- Check if `cresco-scientiam.firebaseapp.com` is accessible

### Issue 5: Blank white/dark screen after auth
**Problem**: Page loads but nothing appears  
**Solution**:
- Check console for JavaScript errors
- Try refreshing the page
- Make sure you're using a modern browser (Chrome, Firefox, Safari, Edge)
- Check if JavaScript is enabled

---

## Debug Mode - Step by Step

### Step 1: Test Firebase Connection
Open browser console and run:
```javascript
// Should return the Firebase app object
console.log(app);
```
If you see an error or undefined, Firebase didn't load properly.

### Step 2: Test Auth State
```javascript
// Should show current user or null
console.log(auth.currentUser);
```

### Step 3: Check Email
If you have a user logged in:
```javascript
console.log(auth.currentUser.email);
```
Should show: `aaravhfs@gmail.com`

### Step 4: Test Firestore Connection
```javascript
console.log(db);
```
Should show Firestore database object.

---

## Browser Console Errors - What They Mean

| Error | Meaning | Fix |
|---|---|---|
| `Failed to fetch Firebase` | Network issue | Check internet connection |
| `Auth/invalid-api-key` | Wrong Firebase config | Check firebaseConfig in HTML |
| `Permission denied` | Wrong Firestore rules | Check Firestore rules are deployed |
| `User.email is undefined` | User exists but no email | Re-login with Google |
| `Cannot read property 'display'` | HTML element missing | Refresh page, clear cache |

---

## Nuclear Options (Try These Last)

1. **Hard Refresh**
   - Windows/Linux: `Ctrl+Shift+R`
   - Mac: `Cmd+Shift+R`

2. **Clear All Browser Data**
   - Go to Settings → Clear browsing data
   - Select "All time"
   - Check: Cookies, Cache
   - Clear

3. **Try Different Browser**
   - Chrome, Firefox, Safari, Edge
   - Sometimes browser-specific issues

4. **Check Firestore Rules**
   - Go to Firebase Console
   - Firestore Database → Rules tab
   - Should see green "Rules are deployed"
   - If not, deploy them:
     ```bash
     firebase deploy --only firestore:rules
     ```

---

## Getting Help

When reporting issues, include:

1. ✅ **Screenshot of browser console errors**
2. ✅ **The console messages you see** (copy-paste)
3. ✅ **Your Google account email**
4. ✅ **Browser & OS** (Chrome on Windows 11, etc.)
5. ✅ **When it last worked** (if applicable)

### Example Report:
```
Browser: Chrome on Windows 11
Email: aaravhfs@gmail.com
Error: "Auth state changed. User: none"
Then: "No user logged in, redirecting to login"
```

---

## Testing Checklist

- [ ] Can access `/login.html`?
- [ ] Can sign in with Google?
- [ ] Are you using `aaravhfs@gmail.com`?
- [ ] Does browser console show no red errors?
- [ ] Does "Firebase initialized successfully" appear?
- [ ] Does "Authorization successful" appear?
- [ ] Do you see the admin dashboard?

---

## Performance Optimization

If page loads slowly but eventually works:

1. **Reduce Firestore queries**
   - Limit participant list display
   - Paginate logs (show 50 at a time, not 1000)

2. **Enable Firestore offline persistence**
   - Add to script:
   ```javascript
   enableIndexedDbPersistence(db).catch(() => {});
   ```

3. **Use Firestore data caching**
   - Already implemented in this version

---

## Quick Fix Template

If you see infinite loading:

1. Open console (F12)
2. Copy any error messages
3. Go to `/login.html`
4. Sign in with `aaravhfs@gmail.com`
5. Navigate to `/admin-x7q/`

---

---

# Security Portal - Troubleshooting Guide

## Issue: Camera Won't Start

### Quick Diagnosis
1. **Open Browser Console** (F12 → Console)
2. **Look for error message** like:
   - `NotAllowedError` → Permission denied
   - `NotFoundError` → No camera device
   - `NotReadableError` → Camera in use by another app
   - `Html5Qrcode not available` → Library failed to load

### Solutions by Error

#### NotAllowedError (Permission Denied)
**Problem**: Browser permission not granted  
**Fix**:
- Click camera icon in address bar
- Change from "Block" to "Allow"
- Refresh page (F5)
- If no permission prompt appears, check browser settings:
  - **Chrome**: Settings → Privacy and security → Site settings → Camera
  - **Safari**: Preferences → Websites → Camera
  - **Firefox**: Preferences → Permissions → Camera

#### NotFoundError (No Camera)
**Problem**: Device has no camera  
**Fix**:
- Use a device with a camera (laptop, phone, tablet)
- Check if camera is enabled in BIOS/device settings
- Try USB camera if available

#### NotReadableError (Camera In Use)
**Problem**: Another app is using the camera  
**Fix**:
- Close other apps using camera (Zoom, Teams, FaceTime, etc.)
- Restart browser
- Restart device if still blocked

#### Html5Qrcode Library Error
**Problem**: QR scanning library didn't load from CDN  
**Fix**:
- Check internet connection
- Check if cdn.jsdelivr.net is accessible (not blocked by firewall)
- Hard refresh: `Ctrl+Shift+R` (Windows) or `Cmd+Shift+R` (Mac)
- Try different browser

---

## Issue: Camera Starts But No Video Feed

### Check These
1. **Wait 2-3 seconds** - Sometimes camera takes time to initialize
2. **Look at overlay** - Scan corners and animated line should be visible
3. **Check lighting** - Too dark? Try better lighting
4. **Check camera position** - Sensor might be covered/dirty

### Solutions
1. Click "Switch Camera" button to try front camera instead
2. Try different app/browser to verify camera hardware works
3. Disable hardware acceleration (Chrome Settings → Advanced)
4. Try different device

---

## Issue: QR Code Won't Scan

### Step-by-Step Fix

1. **Test with known QR**
   - Go to [qr-code-generator.com](https://www.qr-code-generator.com/)
   - Generate any QR code
   - Try scanning it in security portal
   - If that works, problem is QR code quality, not software

2. **Verify QR Code Quality**
   - Check QR code not damaged/faded
   - Check QR code is printed clearly (not pixelated)
   - Check QR code is at least 1cm × 1cm (larger is better)
   - Check no glare/reflection on QR code

3. **Improve Scanning Technique**
   - Hold camera 6-12 inches from QR code
   - Keep camera steady (don't shake)
   - Ensure QR code is centered in frame
   - Ensure good lighting (not too dark, avoid harsh glare)

4. **Check QR Code Content**
   - In admin portal, verify QR ID is correct
   - Check QR ID matches what's in system
   - Try manual entry with that QR ID first

5. **Use Manual Entry as Workaround**
   - Click "Manual QR Entry" at bottom of modal
   - Type or paste the QR ID
   - Click "Go" button
   - This bypasses camera completely

---

## Issue: "Access Denied" at Login

### Solutions

1. **Check Volunteer List**
   - Go to Firebase Console
   - Firestore Database → Collections → `volunteers`
   - Search for your email address (must be exact match)
   - Email must be the one you're signing in with

2. **Check Email Case**
   - Email must match exactly: `John.Doe@gmail.com` ≠ `john.doe@gmail.com`
   - Check what Google account shows in browser

3. **Add Yourself as Volunteer**
   - Go to admin portal
   - Settings tab (if available) → Add volunteer
   - OR: Go to Firebase Console → Create new document in `volunteers` collection
   - Document ID: your email
   - Fields: name, role, checkpoint, enabled (true)

4. **Try Admin Account**
   - If you're the admin (`aaravhfs@gmail.com`), you should have access
   - Sign out and sign in with admin email
   - Admin has access without needing `volunteers` collection entry

---

## Issue: Scan Succeeds but Participant Not Found

### Check These

1. **Verify QR ID in Database**
   - Go to admin portal → Participants
   - Search for participant by name
   - Copy their QR ID exactly
   - Try scanning that ID manually in security portal

2. **Check Participant is Enabled**
   - In admin portal, click participant
   - Check if "Enabled" is ON
   - If disabled, can't scan (will show warning)

3. **Check Participant Name**
   - Verify participant was added to system
   - Check spelling/name matches

4. **Force Refresh Participant List**
   - Go to security portal
   - Refresh page (F5)
   - Wait for "Recently Scanned" section to load
   - Then try scanning again

---

## Issue: Duplicate Scan Warning (Participant Already Checked In)

### This is Expected Behavior

When you scan a participant who's already completed a checkpoint:
- System shows modal: "Already Completed"
- Shows when they checked in and who confirmed it
- You can choose to close or override
- This prevents accidental double-entry

### To Allow Duplicates (Override)
- Admin can modify Firestore data directly
- OR: Use manual checkpoint reset in settings

---

## Browser Console Debugging Commands

Open Console (F12 → Console) and type:

```javascript
// Check if library loaded
typeof Html5Qrcode

// Check scanner status
html5QrCode

// Check loaded participants
PARTICIPANTS.length

// Check recent scans
recentLogsCache.slice(0,5)

// Check current volunteer
CURRENT_USER

// Check current checkpoint
currentCheckpoint
```

---

## Performance Issues

### Scanning is Very Slow (5+ seconds per QR)

**Causes**:
- Old device/phone (2015 or older)
- Poor internet connection
- Firestore being slow
- Browser using too much CPU

**Fixes**:
1. Close other tabs/apps
2. Try better WiFi connection
3. Use newer device if available
4. Reduce browser tabs open
5. Disable browser extensions (can slow things down)

---

## Network Debugging

### Check Firestore Connectivity

In browser console:
```javascript
db.collection('participants').limit(1).get()
  .then(() => console.log('Firestore: OK'))
  .catch(err => console.log('Firestore: ERROR', err))
```

Should print `Firestore: OK` if connected.

### Check Internet Connection
```javascript
navigator.onLine  // true if online, false if offline
```

### Check Logs Upload
```javascript
// Scan a participant, then check:
recentLogsCache  // Should show your recent scan
```

---

## Testing Checklist

Security Portal Startup:
- [ ] Can access `/security/` in browser?
- [ ] See Google Sign-In button?
- [ ] Can sign in with your Google account?
- [ ] After sign-in, see dashboard (statistics, search)?

Camera & Scanning:
- [ ] Click "Scan QR Code" button?
- [ ] Camera permission prompt appears?
- [ ] Grant permission and camera starts?
- [ ] See live video feed?
- [ ] Scan corners and animation visible?
- [ ] Scan a participant QR code?
- [ ] Participant info appears?
- [ ] Can click "Confirm"?

End-to-End:
- [ ] Participant marked as "Checked In" in Firestore?
- [ ] Log entry created with volunteer name?
- [ ] Dashboard statistics updated?
- [ ] Recently scanned list shows new entry?

Fallback:
- [ ] Click "Manual QR Entry"?
- [ ] Type a QR ID?
- [ ] Click "Go"?
- [ ] Same verification flow works?

---

## Getting Help

When reporting issues, include:

1. ✅ **Screenshot of browser console** (F12)
2. ✅ **Any error messages** (copy-paste text)
3. ✅ **Your email address**
4. ✅ **Device type** (iPhone 12, MacBook Pro, Samsung tablet, etc.)
5. ✅ **Browser** (Chrome 120, Safari 17, etc.)
6. ✅ **What you were trying to do** when error happened

### Example:
```
Device: iPhone 13
Browser: Safari
Email: john@example.com
Issue: Camera shows black screen after permission granted
Steps:
1. Opened /security/
2. Signed in
3. Clicked "Scan QR Code"
4. Granted camera permission
5. See "Point camera at QR code" text but video is black

Console shows: No errors
```
6. If still loading, hard refresh (Ctrl+Shift+R)
7. If errors persist, share console logs

---

**Last Resort**: Delete browser profile and create new user account in Chrome/Firefox, then try again.
