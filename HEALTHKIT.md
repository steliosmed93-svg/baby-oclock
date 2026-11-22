# Baby O’Clock · Σύνδεση με Apple Watch (HealthKit)

## Σήμερα (web / PWA)
Η ενότητα **Σώμα & Ανάκαμψη** (`#body`) δέχεται δεδομένα με δύο τρόπους:
1. **Εισαγωγή export.xml** από την εφαρμογή Υγεία (Health → προφίλ → Εξαγωγή όλων των δεδομένων). Η ανάγνωση γίνεται με streaming μέσα στη συσκευή (`importAppleHealth`), τελευταίες 120 ημέρες.
2. **Χειροκίνητη καταγραφή** ανά ημέρα.

Όλα αποθηκεύονται στο `S.body.days[YYYY-MM-DD]`:
`hrv, rhr, rr, spo2, temp, sleepH, deep, rem, light, awake, bed, wake, steps, kcal, exMin, workouts[{type,min,int}], sys, dia, vo2, src`

## Μετρήσεις και αντιστοίχιση HealthKit
| Πεδίο | HealthKit identifier | Σημείωση |
|---|---|---|
| hrv | HKQuantityTypeIdentifierHeartRateVariabilitySDNN | Apple = SDNN (άλλες συσκευές = RMSSD). Συγκρίνεται μόνο με τον προσωπικό μέσο όρο |
| rhr | HKQuantityTypeIdentifierRestingHeartRate | |
| rr | HKQuantityTypeIdentifierRespiratoryRate | στον ύπνο |
| spo2 | HKQuantityTypeIdentifierOxygenSaturation | ×100 |
| temp | HKQuantityTypeIdentifierAppleSleepingWristTemperature / BasalBodyTemperature | Series 8+ |
| sleep | HKCategoryTypeIdentifierSleepAnalysis | Core→light, Deep, REM, Awake |
| steps / kcal / exMin | StepCount / ActiveEnergyBurned / AppleExerciseTime | άθροισμα ημέρας |
| workouts | HKWorkoutType | διάρκεια, τύπος |
| vo2 | HKQuantityTypeIdentifierVO2Max | |
| sys / dia | BloodPressureSystolic / Diastolic | |
| περίοδος | MenstrualFlow, OvulationTestResult, CervicalMucusQuality | γράφονται στο `S.period` |

## Βαθμολογίες (δικοί μας αλγόριθμοι)
- **Ανάκαμψη 1–99%**: z-scores HRV (55%), RHR (30%), RR (15%) έναντι βάσης 30 ημερών + απόδοση ύπνου. Ποινή για θερμοκρασία > +0,5°C εκτός ωχρινικής και SpO2 < 94%. Πράσινο ≥67, κίτρινο 34–66, κόκκινο ≤33.
- **Φόρτος 0–21**: 21·(1−e^(−load/220)), load = Σ λεπτά×ένταση + βήματα/1000×4 + λεπτά άσκησης χωρίς προπόνηση×2,5.
- **Ανάγκη ύπνου**: βάση (8 ώρες) + φόρτος χθες + 25% του χρέους 3 ημερών + 0,25 ώρες στην ωχρινική + 0,5 ώρες στην εγκυμοσύνη.
- **Προτεινόμενος φόρτος**: ανάλογα με την ανάκαμψη, με όρια στη διέγερση/μετά την ωοληψία (≤10), εγκυμοσύνη (≤13), έντονη περίοδο (≤11).
- **Ωορρηξία από θερμοκρασία**: 3 συνεχόμενες ημέρες ≥0,2°C πάνω από το μέγιστο των 6 προηγούμενων.

## Επόμενο βήμα: native εφαρμογή (Capacitor)
1. `npm i @capacitor/core @capacitor/ios` και ένα HealthKit plugin (π.χ. `capacitor-health` ή `@perfood/capacitor-healthkit`).
2. Xcode: Capabilities → HealthKit (+ Background Delivery). Info.plist: `NSHealthShareUsageDescription` «Το Baby O’Clock διαβάζει ύπνο, παλμούς, HRV και θερμοκρασία για να σου δείξει την ανάκαμψη ανά φάση του κύκλου».
3. Ζήτα άδεια ανάγνωσης για τους τύπους του πίνακα.
4. Στο `healthKitSync()` (index.html): αν υπάρχει `Capacitor.Plugins.Health`, διάβασε τις τελευταίες 30 ημέρες, γέμισε `S.body.days` με την ίδια δομή που φτιάχνει το `importAppleHealth`, κάλεσε `save()`.
5. Συγχρονισμός σε κάθε άνοιγμα της εφαρμογής και με HKObserverQuery στο παρασκήνιο.
6. Προαιρετικά: watchOS companion app για καταγραφή συσπάσεων και κινήσεων μωρού από τον καρπό.

Σημείωση App Store: τα δεδομένα HealthKit δεν χρησιμοποιούνται για διαφήμιση και δεν μοιράζονται χωρίς ρητή συναίνεση (κανόνας 5.1.3). Η εφαρμογή δεν κάνει διάγνωση.

## Δικό μας βραχιόλι (Bluetooth LE)
Στην ενότητα «Σώμα & Ανάκαμψη» υπάρχει κάρτα «Δική μας συσκευή» (Web Bluetooth: Chrome σε Android/υπολογιστή, στο iPhone μέσω της native εφαρμογής). Υποστηρίζει:
- Heart Rate 0x180D / 0x2A37 (παλμοί + RR → RMSSD), π.χ. Polar H10
- Health Thermometer 0x1809 / 0x2A1C
- Pulse Oximeter 0x1822 / 0x2A5E
- Battery 0x180F / 0x2A19
- Baby O’Clock Night `b0c10000-6f63-4c6b-9b0b-0000000000b0`, summary `b0c10001-…`: JSON (αντικείμενο ή πίνακας) με `date` και τα πεδία του `S.body.days`.
