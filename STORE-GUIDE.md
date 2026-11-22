# Baby O'Clock: από το website στο App Store και στο Google Play

Η εφαρμογή είναι ήδη PWA (εγκαθίσταται από τον browser). Για να μπει στα stores, «τυλίγεται» με το **Capacitor** χωρίς να ξαναγραφτεί.

## Τι χρειάζεσαι
| | iPhone (App Store) | Android (Google Play) |
|---|---|---|
| Λογαριασμός | Apple Developer Program, 99 $/έτος | Google Play Console, 25 $ μία φορά |
| Υπολογιστής | Mac με Xcode | Οποιοσδήποτε με Android Studio |
| Νομικά | Πολιτική απορρήτου (υπάρχει: `#privacy`) | Ίδια, και φόρμα «Data safety» |
| Health apps | Δήλωση ότι δεν είναι ιατροτεχνολογικό προϊόν | Ίδια, δήλωση «Health app» |

## Βήματα (σε υπολογιστή με Node.js)
```bash
git clone https://github.com/steliosmed93-svg/baby-oclock.git && cd baby-oclock
mkdir www && cp index.html doctor.html manifest.webmanifest icon.svg sw.js www/
npm init -y
npm i @capacitor/core @capacitor/cli @capacitor/ios @capacitor/android @capacitor/local-notifications @capacitor/push-notifications
npx cap add ios && npx cap add android
npx cap sync
npx cap open ios       # Xcode → Signing → Archive → App Store Connect
npx cap open android   # Android Studio → Build → Generate Signed Bundle (.aab) → Play Console
```

## Πριν την υποβολή
1. Στο Firebase → Authentication → Settings → Authorized domains: πρόσθεσε `localhost` (υπάρχει) και, για iOS, ρύθμισε Google Sign-In με το iOS client ID.
2. Εικονίδια: από το `icon.svg` παράγονται όλα τα μεγέθη (π.χ. με `npx @capacitor/assets generate`).
3. Ειδοποιήσεις: με το plugin `@capacitor/local-notifications` οι υπενθυμίσεις φαρμάκων δουλεύουν και με κλειστή εφαρμογή.
4. Screenshots: 6,7" και 5,5" για iPhone, τηλέφωνο και tablet για Android.
5. Περιγραφή store, κατηγορία «Medical» ή «Health & Fitness», ηλικία 18+ (17+ στο iOS).
