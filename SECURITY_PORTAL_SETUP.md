# Security Portal Setup & Deployment

## Prerequisites

✅ Firebase project already created: `cresco-scientiam`
✅ Admin portal working at `/admin-x7q/`
✅ Google Sign-In configured in Firebase

---

## Step 1: Deploy Firestore Rules

**Why**: New rules support volunteers collection and checkpoint scanning permissions

### Option A: Firebase CLI (Recommended)
```bash
firebase login
firebase deploy --only firestore:rules
```

Expected output:
```
✔  Firestore Rules have been successfully deployed.
i  To view your deployed Firestore rules in the console, visit https://console.firebase.google.com
```

### Option B: Firebase Console
1. Go to Firebase Console → cresco-scientiam project
2. Firestore Database → Rules tab
3. Copy contents of `firestore.rules`
4. Click "Publish"

**Verify Deployment**:
1. In Firebase Console, check timestamp in Rules tab
2. Should show today's date
3. Click "Edit" to verify rules content

---

## Step 2: Create Volunteers Collection

Security volunteers need to be added to the `volunteers` collection before they can access the security portal.

### Via Firebase Console (Manual)
1. Go to Firebase Console → cresco-scientiam project
2. Firestore Database → Data tab
3. Click "Start Collection"
4. Name: `volunteers`
5. Create first document:
   - Document ID: `volunteer@email.com` (their Google email)
   - Add fields:
     ```
     name: (string) "Volunteer Name"
     role: (string) "checkpoint-scanner"
     checkpoint: (string) "checkin"  [optional: assign default checkpoint]
     enabled: (boolean) true
     createdAt: (timestamp) now
     ```

### Example Volunteers
```
volunteers/
├── john.volunteer@example.com
│   ├── name: "John Volunteer"
│   ├── role: "checkpoint-scanner"
│   ├── checkpoint: "checkin"
│   ├── enabled: true
│   └── createdAt: 2026-08-28T09:00:00Z
├── sarah.team@example.com
│   ├── name: "Sarah Team"
│   ├── role: "checkpoint-scanner"
│   ├── checkpoint: "bagcheck"
│   ├── enabled: true
│   └── createdAt: 2026-08-28T09:00:00Z
└── checkpoint.staff@example.com
    ├── name: "Checkpoint Staff"
    ├── role: "checkpoint-scanner"
    ├── checkpoint: "food"
    ├── enabled: true
    └── createdAt: 2026-08-28T09:00:00Z
```

### Via Script (Bulk Upload)
To add multiple volunteers at once, use the admin portal's settings page (if you add a batch upload feature).

For now, manual entry works for small teams.

---

## Step 3: Test Security Portal

### Access Checklist
- ✅ Open `/security/` in browser
- ✅ You should see Google Sign-In button
- ✅ Sign in with a Google account that's in the `volunteers` collection
- ✅ Should see: Dashboard, Scan Panel, Search, Schools, Recent Scans
- ✅ Click "Scan QR Code" button

### Camera Test
- ✅ Modal opens
- ✅ Camera starts (may ask for permission)
- ✅ You see live video feed
- ✅ Scan overlay corners visible
- ✅ Green line animates across frame

### QR Scan Test
1. In admin portal, create a participant with QR code
2. In security portal, click "Scan QR Code"
3. Point camera at QR code
4. Should detect and show participant verification
5. Click "Confirm Check-In"
6. Should show success toast
7. Participant marked as "Checked In" in portal

### Manual Entry Test
1. Click "Scan QR Code"
2. Click "Manual QR Entry" at bottom
3. Type a QR ID from a participant
4. Click "Go" or press Enter
5. Should show verification same as camera scan

### Access Denial Test
1. Sign out
2. Sign in with account NOT in `volunteers` collection
3. Should see "Access Denied" message
4. Should show email address
5. Click "Sign Out & Try Another Account"

---

## Step 4: Configure Firestore Rules (Optional Advanced)

### Limit Volunteers by Checkpoint (Field-Level)
If you want volunteers to only scan their assigned checkpoint, update rules:

```javascript
allow write: if isVolunteer() && (
  request.writeFields.size() <= 2 &&
  (request.writeFields.has('day1') || request.writeFields.has('day2'))
  // AND (get(/databases/$(database)/documents/volunteers/$(request.auth.token.email)).data.checkpoint == resource.data.currentCheckpoint)
);
```
⚠️ This requires storing current checkpoint in each participant doc.

### Restrict to Specific Users per Checkpoint
Alternative: Maintain `allowed_volunteers` array on settings:
```json
{
  "checkin_volunteers": ["john@email.com", "jane@email.com"],
  "bagcheck_volunteers": ["alex@email.com"],
  "food_volunteers": ["sam@email.com", "pat@email.com"],
  "checkout_volunteers": ["admin@email.com"]
}
```
Then check in rules against this list.

Currently, all volunteers can use all checkpoints (open model).

---

## Step 5: Monitor & Debug

### Check Logs
1. Firebase Console → Firestore → Data
2. Click `logs` collection
3. Should see entries like:
   ```
   {
     "participantId": "abc123",
     "participantName": "John Doe",
     "volunteer": "John Volunteer",
     "checkpoint": "checkin",
     "day": "day1",
     "method": "QR Scan",
     "timestamp": "2026-08-28T10:30:00Z",
     "device": "Mozilla/5.0...",
     "result": "Success"
   }
   ```

### Browser Console Debugging
Open DevTools (F12 → Console) while scanning:
```javascript
// Check if scanner started
console.log(html5QrCode);  // Should show scanner object

// Check participant data
console.log(PARTICIPANTS);  // Should show loaded participants

// Check current mode
console.log(usingFacingMode);  // Should show "environment" or "user"

// Check recent logs
console.log(recentLogsCache.slice(0,3));  // Show last 3 scans
```

### Common Issues

| Issue | Check |
|-------|-------|
| Camera won't start | Browser permissions (F12 → Console check for errors) |
| Volunteer access denied | Check `volunteers` collection has their email |
| QR not detecting | Check QR code quality, lighting, distance |
| Scan freezes | Check internet connection, Firestore status |
| Logs not appearing | Check `logs` collection in Firebase, check rules |

---

## Step 6: Event Day Preparation

### Before Event
- ✅ Test camera on multiple devices (phones, tablets)
- ✅ Add all volunteers to `volunteers` collection
- ✅ Brief volunteers on camera angle, QR code positioning
- ✅ Test manual QR entry as backup
- ✅ Verify internet connection at all checkpoints
- ✅ Have backup phone/tablet in case primary fails

### Camera Setup
- ✅ Grant camera permission to Chrome/Safari/etc. ahead of time
- ✅ Clean camera lens on all devices
- ✅ Test flashlight on each device (may not be supported)
- ✅ Test switching cameras (if phone has front+back)

### QR Code Printing
- ✅ Print participant QR codes in admin portal
- ✅ Verify QR codes print clearly (not too small)
- ✅ Laminate if reusing across days
- ✅ Have backup QR codes in case originals get damaged

### Checkpoint Staffing
- ✅ Assign volunteers to checkpoints
- ✅ Each checkpoint should have 2+ devices/tablets
- ✅ Have written list of participant names as backup
- ✅ Volunteers should know how to use manual entry

### Network
- ✅ Test WiFi signal at all checkpoint locations
- ✅ Have hotspot as backup if WiFi fails
- ✅ Know IT contact number for connection issues
- ✅ Test Firestore connectivity before event starts

---

## Troubleshooting Deployment

### Rules Won't Deploy
```bash
firebase deploy --only firestore:rules --debug
```
Check error message and fix syntax (JSON/Firebase rules format).

### Volunteers Can't Access Portal
1. Check volunteer email is in `volunteers` collection (case-sensitive!)
2. Check email matches Google account email exactly
3. Check `enabled` field is `true`
4. Try signing out and signing in again
5. Check browser console for errors (F12)

### Can't Write Logs During Scan
1. Check `logs` collection exists in Firestore
2. Check rules allow `isVolunteer() && write`
3. Check `volunteers` collection has the volunteer's email
4. Check volunteer is signed in (not admin account)
5. Check Firestore quota not exceeded

### Camera Works But QR Won't Detect
1. Print test QR code from [qr-code-generator.com](https://www.qr-code-generator.com/)
2. Test scanning that QR in security portal
3. If that works, QR code quality issue, not software
4. Improve lighting, QR code size, camera angle
5. Try manual entry instead

---

## Rollback Plan

If something goes wrong:

### Revert Firestore Rules
```bash
# Save current version first
firebase firestore:rules:describe > rules_backup.txt

# Edit firestore.rules back to previous version
# Remove volunteers references if needed

firebase deploy --only firestore:rules
```

### Disable Volunteer Access
Change `isVolunteer()` function:
```javascript
function isVolunteer() {
  return false;  // Temporarily disable all volunteers
}
```

Then add them back once fixed.

### Access Portal as Admin Only
Temporarily change security portal auth to check `isAdmin()` only:
```javascript
if (!snap.exists()) {  // Just check admins collection
```

---

## Files Updated

- ✅ `/security/index.html` - Camera scanning implementation
- ✅ `/firestore.rules` - Added volunteers collection & permissions
- ✅ `/SECURITY_CAMERA_GUIDE.md` - Camera troubleshooting guide
- ✅ `/SECURITY_PORTAL_SETUP.md` - This file

## Next Steps

1. ✅ Deploy firestore rules: `firebase deploy --only firestore:rules`
2. ✅ Add volunteers to `volunteers` collection in Firebase
3. ✅ Test camera in security portal
4. ✅ Test QR scanning end-to-end
5. ✅ Brief volunteers on procedures
6. ✅ Have event staff test on event day morning
