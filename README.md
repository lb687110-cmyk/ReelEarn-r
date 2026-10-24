# ReelEarn (Expo + Firebase)

Black theme. Backend is Firebase only: Authentication, Cloud Firestore and Cloud Storage for project reelearn-c1fd5 (config in src/firebase.ts). No other backend code, credentials or dependencies remain in this project.

## Run
npm install
npx expo start -c

Phone sign-in needs one more package:
npx expo install expo-firebase-recaptcha react-native-webview

Email sign-in works without it.

## One-time Firebase setup
Firebase Console: https://console.firebase.google.com/ > reelearn-c1fd5

1. Authentication > Sign-in method: enable Email/Password and Phone.
2. Firestore Database: create the database (production mode).
3. Storage: click Get started.
4. Deploy the rules:
npm i -g firebase-tools && firebase login && firebase deploy --only firestore:rules,storage --project reelearn-c1fd5
5. Create the fund rules document:
cd admin && npm i && export GOOGLE_APPLICATION_CREDENTIALS=/path/to/serviceAccountKey.json && node seed.js
Service-account key: Project settings > Service accounts > Generate new private key. Never put it in the app.
6. Optional: Authentication > Templates to customise verification and reset emails.

## Screens (13 steps)
Splash, Login (email + password, or mobile number + text code), Forgot password (reset link by email), Create account, Choose method, Mobile number (+91 default) / Email, Verification (6-digit text code with resend timer, or check your email link screen), Create password (email accounts), First + Last name, Date of birth with 18+ block, Username (live availability), Profile photo (gallery or camera).
After sign-up the app opens on Reels (ad after every 3rd/4th reel), Wallet and Profile.

## Data (Firestore)
users/{uid}: private name, date of birth, contact, onboarding step; owner writes.
profiles/{uid}: public username, name, photo; owner writes.
usernames/{name}: claims a username; create once by that user.
posts, posts/{id}/likes, follows: reels, likes and follows; their owners write.
wallets/{uid} + ledger: balance, totals and history.
withdrawals/{id}: withdrawal requests (UPI / bank); create only by owner.
fundSettings/main, fundPayouts: fund rules and monthly credits; you/server code only.

Wallet protection: firestore.rules let the app lower its own balance in exactly one way: a withdrawal created in the same write, for exactly that amount, within min/max. The app cannot raise a balance, edit or delete a withdrawal, or change its status. Credits, refunds and payments are written only by trusted server code (functions/, admin/).

## Creator fund and paying withdrawals
Monthly credit: Rs 0.10 per eligible follower, from 5,000 followers, up to Rs 10,00,000 a month; verified accounts older than 7 days only.
Blaze plan: cd functions && npm i && firebase deploy --only functions (also creates a wallet for every new user).
Spark free plan: run node admin/run-fund.js on the 1st of each month. Wallets are also created by the app on first sign-in.

Withdrawals are a manual review queue. After you pay one:
node admin/set-withdrawal.js <withdrawalId> paid "UTR 123"
processing and rejected also work; rejected returns the money to the wallet. No payment gateway is connected.

## AdMob
Snack / Expo Go show a placeholder ad card (src/ads/AdSlot.tsx). For real ads copy src/ads/AdSlot.admob.tsx over it and make a development build. Test ad IDs are used until you put in your own.

## Honest limits
Not run on a device from the build environment; rules and functions were written to Firebase's documented behaviour but not executed against a real project or emulator. Test with a dummy account and a small withdrawal first.
Username login and Google/Facebook buttons were removed: Firebase needs a server to look up a username, and social login needs OAuth client IDs from your console.
The age check uses the date the person enters; it is not identity verification.
No data was migrated from the old backend (it had no users or posts).
