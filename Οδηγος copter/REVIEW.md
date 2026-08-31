# Έλεγχος ορθότητας και πληρότητας — Οδηγός Tuning ArduCopter

**Αντικείμενο ελέγχου:** `copter-tuning-guide/` στο branch `claude/ardupilot-tuning-course-s28nkl`
(13 αρχεία, 4.173 γραμμές, 46 διαγράμματα mermaid).

**Πηγή αλήθειας:** ο πηγαίος κώδικας αυτού του repository — ArduCopter **4.8.0-dev**
(`ArduCopter/version.h`), μαζί με το `apm.pdef.json` που παρήγαγε το
`Tools/autotest/param_metadata/param_parse.py --vehicle ArduCopter` (5.771 παράμετροι).

---

## Ετυμηγορία

Ο οδηγός είναι **σε πολύ καλή κατάσταση**. Ο ισχυρισμός του README ότι «όλα τα ονόματα
παραμέτρων, τα defaults και οι τιμές έχουν επαληθευτεί απευθείας από τον πηγαίο κώδικα»
ευσταθεί στη συντριπτική πλειοψηφία των περιπτώσεων: επαλήθευσα περίπου 150 ονόματα
παραμέτρων, default τιμές, λίστες enum, bitmasks και πεδία μηνυμάτων log, και βρήκα:

| | Πλήθος |
|---|---|
| **Λάθη προς διόρθωση** | 4 |
| **Μικρότερα / ελλείψεις** | 6 |
| Παράμετροι/τιμές που επαληθεύτηκαν ως σωστές | ~150 |
| Σπασμένοι εσωτερικοί σύνδεσμοι | 0 |
| Διαγράμματα mermaid με συντακτικό σφάλμα | 0 από 46 |
| Κάλυψη των lessons του course | πλήρης |

Κανένα από τα λάθη δεν είναι επικίνδυνο για πτήση. Δύο όμως είναι **ονόματα παραμέτρων που
δεν υπάρχουν**, οπότε ο αναγνώστης δεν θα τα βρει στο Mission Planner.

---

## Λάθη προς διόρθωση

### Ε1 — Το default του `ATC_INPUT_TC` είναι **0.10**, όχι 0.15

Ο οδηγός δίνει 0.15 ως εργοστασιακή τιμή. Στην 4.8 το 0.15 είναι το default του **Plane**·
για Copter (και Sub) είναι **0.10** («Crisp»).

**Πηγή:** `libraries/AC_AttitudeControl/AC_AttitudeControl.cpp:9-17`

```c
#if APM_BUILD_TYPE(APM_BUILD_ArduPlane)
 # define AC_ATTITUDE_CONTROL_INPUT_TC_DEFAULT  0.15f    // Medium
#else
 // default gains for Copter and Sub
 # define AC_ATTITUDE_CONTROL_INPUT_TC_DEFAULT  0.10f   // Crisp
#endif
```

Επιβεβαιώνεται και από το `ArduCopter/ReleaseNotes.txt:228`:
«ATC_INPUT_TC and Q_A_INPUT_TC param defaults reduced (@rmackay9, PR:32643)».

**Θέσεις προς διόρθωση**

| Αρχείο:γραμμή | Τι λέει τώρα |
|---|---|
| `07-Stabilisation-Mode-Tuning.md:211` | `` `ATC_INPUT_TC` \| 0.15 s `` |
| `07-Stabilisation-Mode-Tuning.md:219` | `\| **0.15** \| Default, ισορροπημένο \|` |
| `11-Tips-for-Tuning-Large-Drones.md:145` | «Αύξησε το `ATC_INPUT_TC` (π.χ. 0.15 → 0.25)» |
| `A-Parameter-Mapping.md:158` | Το `ATC_INPUT_TC` στη λίστα «τι δεν άλλαξε» — σωστό για το **όνομα**, αλλά η default τιμή άλλαξε στην 4.7 |

Οι πίνακες του κεφ. 11 (γρ. 76 και 371) που δίνουν «0.10 – 0.15» για 5" quad είναι ήδη
συμβατοί με το πραγματικό default.

> Χρήσιμη λεπτομέρεια για τον πίνακα αίσθησης: τα `@Values` του κώδικα είναι
> `0.05:Very Crisp, 0.1:Crisp, 0.15:Medium, 0.2:Soft, 0.5:Very Soft`.

---

### Ε2 — Η παράμετρος `PSC_JERK_D` δεν υπάρχει· το σωστό όνομα είναι `PSC_D_JERK`

**Πηγή:** `libraries/AC_AttitudeControl/AC_PosControl.cpp:145`

```c
AP_GROUPINFO("_D_JERK", 11, AC_PosControl, _shaping_jerk_d_msss, POSCONTROL_JERK_D_MSSS),
```

Με το prefix `PSC` της ομάδας (`ArduCopter/Parameters.cpp`, `GOBJECTPTR(pos_control, "PSC", …)`)
το πλήρες όνομα είναι **`PSC_D_JERK`**. Το ίδιο επιβεβαιώνει και το `apm.pdef.json`.
Η default τιμή 5.0 m/s³ που δίνει ο οδηγός είναι σωστή.

**Θέσεις:** `08-Altitude-Hold-Mode-Tuning.md:96`, `:168` (mermaid), `:341`,
και `A-Parameter-Mapping.md:44`.

---

### Ε3 — Η παράμετρος `PSC_JERK_NE` δεν υπάρχει· το σωστό όνομα είναι `PSC_NE_JERK`

**Πηγή:** `libraries/AC_AttitudeControl/AC_PosControl.cpp:136`

```c
AP_GROUPINFO("_NE_JERK", 10, AC_PosControl, _shaping_jerk_ne_msss, POSCONTROL_JERK_NE_MSSS),
```

Default 5.0 m/s³ — σωστό στον οδηγό.

**Θέσεις:** `09-Loiter-Mode-Tuning.md:17` (mermaid), `:81`, `:151` (mermaid), `:237` (τίτλος
ενότητας), και `A-Parameter-Mapping.md:58`.

---

### Ε4 — Παράρτημα Α.1: τρεις συντελεστές «× 0.1» δεν εφαρμόζονται από τον κώδικα

Ο οδηγός δηλώνει ότι «όλες οι πληροφορίες εδώ προέρχονται από τους πίνακες
`convert_parameters()`». Για τρεις γραμμές αυτό δεν ισχύει:

| Γραμμή | Ο οδηγός λέει | Τι κάνει ο κώδικας |
|---|---|---|
| `A-Parameter-Mapping.md:35` | `PSC_ACCZ_FF` → `PSC_D_ACC_FF` **× 0.1** | μεταφορά **χωρίς** κλιμάκωση |
| `A-Parameter-Mapping.md:41` | `PSC_ACCZ_PDMX` → `PSC_D_ACC_PDMX` **× 0.1** | μεταφορά **χωρίς** κλιμάκωση |
| `A-Parameter-Mapping.md:42` | `PSC_ACCZ_D_FF` → `PSC_D_ACC_D_FF` **× 0.1** | μεταφορά **χωρίς** κλιμάκωση |

Στο `AC_PosControl::convert_parameters()` οι τρεις αυτές παράμετροι βρίσκονται στον πίνακα
`conversion_info`, ο οποίος περνάει από την **ακλιμάκωτη** `AP_Param::convert_old_parameters()`.
Μόνο τα `PSC_D_ACC_P/I/D` είναι στον `conversion_info_01` με συντελεστή `0.1`, και το
`PSC_D_ACC_IMAX` περνάει ξεχωριστά με `0.001`.

**Προσοχή — δεν είναι απλό λάθος του οδηγού.** Η τεκμηρίωση των ίδιων των παραμέτρων λέει
το αντίθετο από τον κώδικα:

```
PSC_D_ACC_FF   -> "…If upgrading from 4.6 this is _ACCZ_FF * 0.1."
PSC_D_ACC_PDMX -> "…If upgrading from 4.6 this is _ACCZ_P * 0.1."
PSC_D_ACC_D_FF -> "…If upgrading from 4.6 this is _ACCZ_P * 0.1."
```

Δηλαδή υπάρχει **ασυμφωνία μέσα στον ίδιο τον ArduPilot**: το `@Description` λέει ×0.1, η
αυτόματη μετατροπή δεν το κάνει.

**Πρακτική συνέπεια:** η δήλωση του Παραρτήματος ότι «η μετατροπή γίνεται αυτόματα, δεν
χρειάζεται να ξανα-εισάγεις τιμές με το χέρι» **δεν ισχύει για αυτές τις τρεις**. Όποιος
είχε μη-μηδενικό `PSC_ACCZ_FF`, `PSC_ACCZ_PDMX` ή `PSC_ACCZ_D_FF` στην 4.6 θα βρεθεί μετά
την αναβάθμιση με τιμή **10× μεγαλύτερη** από τη σωστή. Ο οδηγός πρέπει να το προειδοποιεί
ρητά.

---

## Μικρότερα και ελλείψεις

### Μ1 — Παράρτημα Α.5: λείπει το × 0.01 στο `ANGLE_MAX`

`A-Parameter-Mapping.md:122` λέει μόνο «μετακινήθηκε στην ομάδα `ATC_`». Στην πραγματικότητα
το entry βρίσκεται στον πίνακα `conversion_info_001` που περνάει με συντελεστή `0.01`
(`AC_AttitudeControl.cpp:1193-1201`), γιατί η παλιά παράμετρος ήταν σε centidegrees:
`ANGLE_MAX = 4500` → `ATC_ANGLE_MAX = 45`.

### Μ2 — Κεφ. 5 §5.10: κενά defaults στον πίνακα FFT

Επτά γραμμές έχουν «—» ενώ υπάρχουν πραγματικά defaults
(`libraries/AP_GyroFFT/AP_GyroFFT.cpp`):

| Παράμετρος | Γραμμή οδηγού | Πραγματικό default |
|---|---|---|
| `FFT_WINDOW_SIZE` | 390 | 64 σε STM32H7, αλλιώς **32** |
| `FFT_WINDOW_OLAP` | 391 | 0.75 σε STM32H7, αλλιώς **0.5** |
| `FFT_SNR_REF` | 393 | **25** dB |
| `FFT_HMNC_FIT` | 395 | **10** |
| `FFT_FREQ_HOVER` | 399 | **80** Hz |
| `FFT_THR_REF` | 400 | **0.35** |
| `FFT_BW_HOVER` | 401 | **20** Hz |

Το πιο σημαντικό είναι το `FFT_HMNC_FIT`: ο οδηγός γράφει «0 = απενεργοποιημένο» χωρίς να
πει ότι το default (10) το αφήνει **ενεργό**.

### Μ3 — Κεφ. 7 §7.2: απλοποιημένος ο τύπος του sqrt controller

Η γραμμή 48 δίνει `rate = √(2 × accel_lim × σφάλμα)`. Ο κώδικας
(`libraries/AP_Math/control.cpp`, `sqrt_controller()`) υπολογίζει
`√(2 · accel_lim · (σφάλμα − linear_dist/2))`, όπου `linear_dist = accel_lim / P²` — το
σύνορο που ο οδηγός αναφέρει σωστά στη γρ. 59. Αποδεκτή απλοποίηση για διάγραμμα, αλλά
αξίζει υποσημείωση δεδομένου ότι ο οδηγός διαφημίζει επαλήθευση από τον κώδικα.

### Μ4 — Κεφ. 3 §3.10: λείπει η προειδοποίηση του `ESC_CALIBRATION`

`03-Initial-Configuration.md:250`. Το `@Description` του κώδικα λέει ρητά
«Do not adjust this parameter manually» — τη ρυθμίζει το Mission Planner κατά τη
διαδικασία calibration. Οι τιμές του πίνακα (0/1/2/3/9) είναι σωστές.

### Μ5 — Χαρτογράφηση lessons: το vertical kinematics του chapter 9

Τα lessons **9.13 «Tuning Vertical Kinematic Parameters»** και **9.14 (quiz)** ανήκουν στο
chapter 9 του course, αλλά ο οδηγός τα καλύπτει στο **§8.13**. Το περιεχόμενο υπάρχει, η
παραπομπή όχι — μία γραμμή στο κεφ. 9 («τα κάθετα kinematics είναι στο §8.13») κλείνει το κενό.

### Μ6 — README: η αρίθμηση ξεκινά από το 02 χωρίς εξήγηση

Ο πίνακας κεφαλαίων του `README.md` ξεκινά από το 02. Η εξήγηση (τα chapters 1 «Welcome and
Introduction» και 12 «Close» παραλείπονται σκόπιμα) υπάρχει μόνο στο
`00-Course-Structure.md`. Μία πρόταση στο README αποτρέπει την εντύπωση ότι λείπει αρχείο.

---

## Τι επαληθεύτηκε ως σωστό

Δείγμα από τα ~150 σημεία που ελέγχθηκαν ένα προς ένα και βρέθηκαν **ακριβή**:

**Κεφ. 3 — Configuration**
`FRAME_CLASS` / `FRAME_TYPE` (τιμές enum), `MOT_PWM_TYPE` 0–7,
`SERVO_BLH_POLES` default 14, `SERVO_BLH_BDMASK`, `SERVO_DSHOT_RATE`/`_ESC` (τιμές),
`SERVOn_FUNCTION` 33–40 = Motor1–8, `SERIALn_PROTOCOL` 1/2/5/16/23/28/33/42,
`SERIALn_BAUD` συντομεύσεις, `ESC_CALIBRATION` 0/1/2/3/9, `FLTMODE*` τιμές,
`FS_THR_ENABLE` τιμές, `BATT_MONITOR` 0/3/4, `MOT_SPIN_ARM` 0.10, `MOT_SPIN_MIN` 0.15,
`INS_FAST_SAMPLE` bitmask, `INS_GYRO_RATE` 0–3, `SCHED_LOOP_RATE` default 400 (Copter),
`FSTRATE_ENABLE`, **και τα 10 bits του `LOG_BITMASK`** που αναφέρονται,
`INS_LOG_BAT_CNT` 1024 / `_OPT` 0 / `_LGIN` 20 / `_LGCT` 32,
`ATC_RAT_RLL/PIT_P` 0.135, `_I` 0.135, `_D` 0.0036, `ATC_RAT_YAW_P` 0.180 / `_I` 0.018 /
`_D` 0, `ATC_ANG_*_P` 4.5, `MOT_THST_EXPO` 0.65, `MOT_THST_HOVER` 0.35.

**Κεφ. 4 — Maiden flight**
`ARMING_SKIPCHK` (νέο όνομα, default 0) και η αντιστροφή λογικής από `ARMING_CHECK`,
πεδία `PM` (`LR, NLon, NL, MaxT, Mem, Load, Ex`), πεδία `VIBE`
(`IMU, VibeX, VibeY, VibeZ, Clip`), `FS_VIBE_ENABLE` default 1, `CTUN.ThH`,
`MOT_HOVER_LEARN` default 2 («Learn and Save»).

**Κεφ. 5 — Filters**
Η **σειρά της αλυσίδας φιλτραρίσματος** (notch πρώτα, LPF τελευταίο) —
`AP_InertialSensor_Backend.cpp:254`: «apply the low pass filter last to attenuate any notch
induced noise». `INS_GYRO_FILTER` / `INS_ACCEL_FILTER` default 20 Hz (Copter),
`INS_HNTCH_FREQ` 80 / `_BW` 40 / `_ATT` 40 / `_HMNCS` 3 / `_REF` 0 / `_MODE` 1 (Throttle) /
`_FM_RAT` 1.0, **και τα 7 bits του `INS_HNTCH_OPTS`** μαζί με τη σημείωση ότι το double
υπερισχύει του triple, 2 harmonic notches σε boards ≤ 1 MB και 3 σε μεγαλύτερα,
οι έξι τιμές του `MODE`, η συμπεριφορά `REF = 0` (παγώνει το tracking — ακριβώς όπως το
`if (is_zero(ref))` του `AP_Vehicle::update_dynamic_notch()`), οι τύποι
`freq = RPM × REF / 60`, `freq = μέση_συχνότητα × REF`, `freq = FREQ × √(throttle / REF)`,
η συμπεριφορά Multi-Source ανά mode, `RPM1_TYPE` 2 = GPIO / 5 = ESC Telemetry,
`FFT_MINHZ` 50 / `FFT_MAXHZ` 450 / `FFT_ATT_REF` 15 / `FFT_HMNC_PEAK` 0–5 /
`FFT_OPTIONS` bits, **και τα 13 πεδία του `FTN1`**.

**Κεφ. 6 — Rate PID**
Η ροή του `AC_PID::update_all()` όπως αποτυπώνεται στο διάγραμμα: notch στόχου → `FLTT` →
σφάλμα → notch σφάλματος → `FLTE` → παράγωγος **του σφάλματος** → `FLTD`, DFF από την
παράγωγο του **φιλτραρισμένου** στόχου, `Dmod` σε P και D, όριο `PDMX` στο άθροισμα P+D.
`FLTT` 20 / `FLTE` 0 (R,P) και 2.5 (Yaw) / `FLTD` 20 / `SMAX` 0 / `PDMX` 0 / `FF` 0 /
`D_FF` 0 με εύρος 0–0.02, `NTF`/`NEF` εύρος 0–8, η μείωση κερδών «μέχρι το 10 %» του slew
limiter, **και τα 12 πεδία του `PIDR`** και **τα 14 του `RATE`**,
`MOT_YAW_HEADROOM` 200, `ATC_LAND_*_MULT` 1.0, `ATC_RATE_FF_ENAB` 1,
AutoTune = flight mode 15, `RCn_OPTION = 219`, `TUNE` / `TUNE2` / `TUNE_MIN` / `TUNE_MAX`.

**Κεφ. 7 — Angle**
`ATC_ANG_*_P` 4.5 με εύρος 3.0–12.0, `ATC_ACC_R/P_MAX` 1100 (εύρος 0–1800),
`ATC_ACC_Y_MAX` 270 (εύρος 0–720), **τα εσωτερικά όρια 40–720 deg/s² και 10–120 deg/s²**,
`ATC_ANGLE_MAX` default 30° με clamp 10°–80°, `ATC_ANGLE_BOOST` 1, `ATC_ANG_LIM_TC` 1.0,
`PSC_ANGLE_MAX` 0, `ATC_RATE_*_MAX` 0, `linear_dist = accel_lim / P²`.

**Κεφ. 8 & 9 — Position control**
`PSC_D_POS_P` 1.0, `PSC_D_VEL_P` 5.0 / `IMAX` 10 / `FLTE` 5 / `FLTD` 5,
`PSC_D_ACC_P` 0.05 / `I` 0.1 / `D` 0 / `IMAX` 0.8 / `FLTT` 0 / `FLTE` 20 / `FLTD` 0,
`PSC_NE_POS_P` 1.0, `PSC_NE_VEL_P` 2.0 / `I` 1.0 / `D` 0.25 / `IMAX` 10 / `FLTE` 5 / `FLTD` 5
(όλα Copter-specific — ο οδηγός δεν παρασύρθηκε από τα Plane/Sub/Heli defaults),
η παρατήρηση ότι το `PSC_D_VEL_*` είναι `AC_PID_Basic` και το `PSC_NE_VEL_*` `AC_PID_2D`
(χωρίς `FLTT`/`SMAX`/`DFF`) — επαληθεύτηκε από τους πίνακες παραμέτρων των δύο κλάσεων,
`PILOT_SPD_UP` 2.5 / `PILOT_SPD_DN` 0 / `PILOT_ACC_Z` 2.5,
`LOIT_SPEED_MS` 12.5 / `LOIT_ACC_MAX_M` 5.0 / `LOIT_BRK_ACC_M` 2.5 / `LOIT_BRK_JRK_M` 5.0 /
`LOIT_BRK_DELAY` 1.0 / `LOIT_ANG_MAX` 0 / `LOIT_OPTIONS` 1,
πεδία `PSCD`/`PSCN`/`PSCE`, `PIDA`/`PIDN`/`PIDE`, `XKF4.SV`/`SP`.
Σωστοί και οι αριθμοί: `atan(2/9.81) ≈ 11.5°`, `atan(5/9.81) ≈ 27°`.

**Κεφ. 10 & 11**
`WP_SPD` 10.0 / `WP_ACC` 2.5 / `WP_ACC_CNR` 0 (= 2× `WP_ACC`) / `WP_JERK` 1.0 /
`WP_RADIUS_M` 2.0 / `WP_SPD_UP` 2.5 / `WP_SPD_DN` 1.5 / `WP_ACC_Z` 1.0 /
`WP_RFND_USE` 1 / `WP_TER_MARGIN` 10, `ATC_RATE_WPY_MAX` 60 deg/s,
τιμές `WP_YAW_BEHAVIOR`, `ATC_THR_MIX_MIN` 0.1 (0.1–0.25) / `_MAX` 0.5 (0.5–0.9) /
`_MAN` 0.1 (0.1–0.9), `ATC_THR_G_BOOST` 0, `MOT_SLEW_UP/DN_TIME` 0, `MOT_SPOOL_TIME` 0.5,
`MOT_BAT_VOLT_MAX/MIN` 0, `MOT_BAT_CURR_MAX` 0, `MOT_BAT_IDX` 0.
Σωστοί οι υπολογισμοί απόστασης φρεναρίσματος (20 m από 10 m/s, 45 m από 15 m/s με 2.5 m/s²)
και το «41 % περισσότερο throttle στις 45°».

**Παράρτημα Α**
Επαληθεύτηκαν γραμμή προς γραμμή από τους πίνακες `convert_parameters()`:
όλες οι μετατροπές `WPNAV_*` → `WP_*` (× 0.01), `LOIT_*` (× 0.01), `PILOT_*` (× 0.01),
`ATC_ACCEL_*_MAX` και `ATC_SLEW_YAW` → `ATC_RATE_WPY_MAX` (× 0.01),
`PSC_VELZ/VELXY_IMAX` (× 0.01), `PSC_ACCZ_P/I/D` (× 0.1), `PSC_ACCZ_IMAX` (× 0.001),
και η αντιστροφή `ARMING_CHECK` → `ARMING_SKIPCHK` (0 → −1, ALL → 0). Εξαιρέσεις: Ε4 και Μ1.

**Σημείωση:** το `ACT_THR_MIX_MAX` του τίτλου του lesson 11.11 όντως **δεν υπάρχει** —
ο οδηγός το εντοπίζει σωστά και στο `00-Course-Structure.md` και στο §11.9.

---

## Πληρότητα

**Δομικά.** Και τα 13 αρχεία υπάρχουν και συνδέονται μεταξύ τους σωστά· κανένας από τους
εσωτερικούς συνδέσμους markdown δεν είναι σπασμένος. Και τα 46 διαγράμματα mermaid
περνούν από τον parser του mermaid 11 χωρίς σφάλμα.

**Ως προς το course.** Και τα 12 chapters είναι λογοδοτημένα: τα 2–11 καλύπτονται σε αντίστοιχα
αρχεία, τα 1 (Welcome) και 12 (Close) παραλείπονται σκόπιμα και τεκμηριωμένα. Δειγματοληπτικός
έλεγχος ανά lesson δεν βρήκε θέμα χωρίς κάλυψη· η μόνη ασυμφωνία θέσης είναι το Μ5.

**Ως προς την ύλη του tuning.** Η αλυσίδα cascade είναι πλήρης από τα filters ως το waypoint
navigation, με τη σωστή σειρά και με ρητή αιτιολόγηση του γιατί η σειρά είναι δεσμευτική.
Δεν λείπει στάδιο.

---

## Προτεινόμενη σειρά διόρθωσης

1. **Ε2, Ε3** — καθαρή αντικατάσταση ονομάτων (`PSC_JERK_D` → `PSC_D_JERK`,
   `PSC_JERK_NE` → `PSC_NE_JERK`) σε 9 σημεία. Μηδενικό ρίσκο.
2. **Ε1** — διόρθωση του default σε 3 σημεία + μία σημείωση στο Παράρτημα Α.7.
3. **Ε4** — διόρθωση των τριών γραμμών **και** προσθήκη προειδοποίησης ότι η αυτόματη
   μετατροπή δεν τις κλιμακώνει. Αυτό είναι το μόνο εύρημα με πρακτική συνέπεια σε
   πραγματικό drone.
4. **Μ1–Μ6** — όποτε βολεύει.
