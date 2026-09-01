# Security Portal - Camera Scanning Guide

## Overview
The Security Console (`/security/`) is for checkpoint scanning by volunteers and admins. It uses QR code scanning via device camera.

## Access & Authorization
- **Who can access**: Volunteers (security volunteers collection) + Admins
- **Authentication**: Google Sign-In
- **Check**: Looks for user email in `volunteers` or `admins` collection in Firestore

---

## Camera Scanning Features

### ✅ QR Code Scanner
- **Library**: Html5-qrcode (optimized for mobile)
- **Supported**: QR codes only
- **Speed**: ~15 FPS, real-time detection
- **Fallback**: Manual QR entry field
- **Torch**: Flashlight toggle for low light
- **Camera Switch**: Toggle between front/back camera

### How It Works
1. Click "Scan QR Code" button in Security Console
2. Camera starts (may request permission)
3. Point at participant's QR code
4. System reads and verifies participant
5. Confirm checkpoint completion
6. Automatic log entry

---

## Troubleshooting Camera Issues

### Issue 1: Camera Won't Start
**Error**: "Could not start camera: NotAllowedError"
**Solution**:
- ✅ Check browser permissions (click URL bar → Camera icon → Allow)
- ✅ Make sure page is served over HTTPS or localhost (http://localhost works, but https required for production)
- ✅ Reload page after granting permission
- ✅ Try a different browser (Chrome, Safari)

### Issue 2: "Camera access denied"
**Problem**: Browser permission prompt didn't appear  
**Solution**:
1. Check browser settings:
   - **Chrome**: Settings → Privacy → Site Settings → Camera
   - **Safari**: Preferences → Websites → Camera
   - **Firefox**: Preferences → Privacy → Permissions
2. Look for site in allowed/blocked list
3. Change from "Blocked" to "Allow"
4. Reload page

### Issue 3: Camera shows blank/black screen
**Problem**: Camera initialized but not showing video  
**Solution**:
- Wait 2-3 seconds (sometimes takes time to activate)
- Try switching camera (toggle button)
- Try using Manual QR Entry instead
- Try a different device/browser

### Issue 4: QR codes not scanning
**Problem**: Camera works but no QR detection  
**Solution**:
- ✅ Make sure QR code is in focus
- ✅ Ensure good lighting (not too dark, not glare)
- ✅ Hold steady (don't shake)
- ✅ Try closer distance (6-12 inches typically works)
- ✅ Try farther distance (if too close, may not detect)
- ✅ Use Manual QR Entry as fallback: copy the QR ID and paste manually

### Issue 5: Slow scanning / delays
**Problem**: Scanner takes long to detect QRs  
**Solution**:
- Normal FPS: 15 frames/second (expected slight delay)
- Check internet speed (affects Firebase updates)
- Try in better lighting
- Try a newer device (older devices process slower)
- Use Manual QR Entry for faster checking if device is slow

### Issue 6: Flashlight doesn't work
**Problem**: Torch button shows error  
**Solution**:
- ✅ Not all devices support flashlight via camera API
- ✅ Older phones/browsers may not support it
- ✅ Use device's built-in flashlight instead
- ✅ Just proceed without torch in bright light

### Issue 7: "Html5Qrcode not available"
**Problem**: QR library failed to load  
**Solution**:
- Check internet connection (CDN requires online)
- Check browser console for library load errors (F12 → Console)
- Try hard refresh: `Ctrl+Shift+R` (Windows) or `Cmd+Shift+R` (Mac)
- Check firewall blocking cdn.jsdelivr.net

---

## Manual QR Entry

If camera doesn't work:

1. Click "Open QR Scanner"
2. In the modal, at the bottom find: **"Or type QR ID manually"**
3. Scan QR with portable barcode scanner OR
4. Manually type the QR ID from the participant card
5. Click "Go" or press Enter
6. Confirm checkpoint same as camera scanning

---

## Browser Compatibility

| Browser | Desktop | Mobile | Notes |
|---------|---------|--------|-------|
| Chrome  | ✅ Yes  | ✅ Yes | Best support, BarcodeDetector acceleration |
| Safari  | ✅ Yes  | ✅ Yes | iOS 14.3+, may need to allow permission |
| Firefox | ✅ Yes  | ✅ Yes | Good support |
| Edge    | ✅ Yes  | ✅ Yes | Chromium-based, same as Chrome |
| Opera   | ✅ Yes  | ✅ Yes | Similar to Chrome |
| IE 11   | ❌ No   | N/A    | Not supported (too old) |

**Best Experience**: Chrome or Safari on modern phones (2020+)

---

## Mobile Device Tips

### iOS (iPhone/iPad)
- Open in Safari for best results
- Chrome may have permission issues
- Grant camera permission when prompted
- Flashlight: Use phone flashlight, not app torch

### Android
- Chrome recommended
- Grant camera permission when prompted
- Some older devices may be slow
- Flashlight usually works well

### Camera Quality
- Better camera = faster scanning
- Recent phones (2020+): ~1-2 second detection
- Older phones: ~3-5 seconds
- Low light: +50% slower

---

## Testing Camera

### Quick Test
1. Go to `/security/`
2. If logged in, click "Scan QR Code"
3. If modal opens with camera, ✅ camera works
4. If black/stuck, ❌ camera issue
5. Check browser console (F12) for errors

### Test QR Codes
Use any of these QR code generators to test:
- https://www.qr-code-generator.com/
- https://qr.io/
- Just generate any QR code with text like "TEST123"

Then test by:
1. Generate QR on another device
2. Scan from security console
3. Should detect the QR text

---

## Performance Optimization

### Make Scanning Faster
1. ✅ Good lighting (indoor: normal lights, outdoor: avoid harsh glare)
2. ✅ Clean camera lens (wipe phone camera)
3. ✅ Use newer device if possible
4. ✅ Don't move too fast (smooth motion)
5. ✅ Hold camera steady
6. ✅ Adequate distance (6-12 inches typical)

### Server-Side
- Firestore real-time updates (already optimized)
- Logs written async (non-blocking)
- No image uploads (reduces bandwidth)

---

## Debug Mode - Browser Console

Open DevTools (F12) and check:

### Check if library loaded
```javascript
console.log(typeof Html5Qrcode);  // Should show: "function"
```

### Check current camera mode
```javascript
console.log(usingFacingMode);  // Should show: "environment" or "user"
```

### Check if scanner instance exists
```javascript
console.log(html5QrCode);  // Should show scanner object or null
```

### Check recent logs
```javascript
console.log(recentLogsCache.slice(0,3));  // Show last 3 scans
```

---

## Firebase Collection Structure

### `volunteers` Collection
Document ID = email address
```json
{
  "name": "John Volunteer",
  "role": "checkpoint-scanner",
  "checkpoint": "checkin",  // or bagcheck, food, checkout
  "enabled": true,
  "createdAt": "2026-08-28T10:00:00Z"
}
```

### To Add a Volunteer
Go to Firebase Console:
1. Firestore Database → Collections
2. Create collection: `volunteers`
3. Add document with ID = `volunteer@email.com`
4. Fields: name, role, checkpoint, enabled

---

## Network Requirements

### Internet Speed
- Download: 1 Mbps minimum
- Upload: 0.5 Mbps minimum
- Ping: <100ms recommended
- Works offline: Limited (logs queue until online)

### Bandwidth Per Scan
- Camera stream: ~1-2 MB/minute
- Firebase write: ~5 KB
- Typical scan: ~100 KB total

---

## Security Features

✅ **Auth Check**: Only authorized volunteers/admins
✅ **QR Validation**: Checks participant exists in database
✅ **Duplicate Prevention**: Alerts if checkpoint already done
✅ **Override Warning**: Shows if participant disabled or unexpected day
✅ **Audit Log**: Every scan logged with timestamp, volunteer name, method
✅ **HTTPS Only**: Encrypted transmission (production)

---

## Known Limitations

- ❌ Cannot scan from images/gallery (only live camera)
- ❌ Portrait mode only (not landscape)
- ❌ No batch scanning (one QR at a time)
- ❌ No code type selection (QR only, not barcodes)
- ⚠️ Older devices may be slow
- ⚠️ Some phones may not support camera switching

---

## Offline Mode

When internet is offline:
- ✅ Scan still works (camera local)
- ✅ QR detection works
- ⚠️ Participant lookup fails
- ⚠️ Firebase updates queue
- ⚠️ Logs sync when connection returns

---

## Emergency Procedures

### If Camera Completely Fails
1. Use Manual QR Entry instead
2. Or use portable barcode scanner + manual input
3. Contact IT for device replacement if damaged

### If Firestore Down
1. Scans still work locally
2. Updates queue automatically
3. Sync resumes when Firestore back online
4. No data loss

### If App Crashes
1. Refresh page (F5)
2. Re-scan QR
3. System will detect if duplicate and prevent double-entry

---

## Contact & Support

**For Camera Issues**:
- Check browser console (F12 → Console)
- Try different browser
- Try different device
- Check camera permissions

**For Firestore Issues**:
- Check Firebase Console → Firestore
- Verify rules are deployed
- Check volunteer/admin collection exists

**For QR Code Issues**:
- Verify QR is correct format
- Test with generated QR codes
- Check QR code quality (not damaged)
