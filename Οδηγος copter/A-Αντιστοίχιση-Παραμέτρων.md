# Παράρτημα Α — Αντιστοίχιση Παραμέτρων 4.6 → 4.7 / 4.8

> Στην **ArduCopter 4.7** έγινε εκτενής αναδιοργάνωση παραμέτρων: μετονομασίες και μετάβαση
> από **cm σε m** (SI units). Όλες οι πληροφορίες εδώ προέρχονται από τους πίνακες
> `convert_parameters()` του πηγαίου κώδικα.
>
> **Ο οδηγός χρησιμοποιεί τα νέα ονόματα.** Αυτός ο πίνακας σε βοηθά αν πετάς 4.6 ή
> παλιότερα, ή αν διαβάζεις παλιότερα tutorials.

**Καλά νέα:** η μετατροπή γίνεται **αυτόματα** κατά την αναβάθμιση. Σχεδόν πάντα δεν
χρειάζεται να ξανα-εισάγεις τιμές με το χέρι.

**Οι δύο εξαιρέσεις**, και οι δύο σημειωμένες παρακάτω με ⚠:

- τρεις παράμετροι του κάθετου acceleration controller που η αυτόματη μετατροπή **δεν
  κλιμακώνει**, ενώ η τεκμηρίωσή τους λέει ότι θα έπρεπε (§Α.1)·
- παράμετροι που **κράτησαν το όνομά τους αλλά άλλαξαν default** — η αναβάθμιση δεν τις
  αγγίζει, αλλά μια «καθαρή» εγκατάσταση συμπεριφέρεται διαφορετικά (§Α.9).

---

## Α.1 Position Controller — `PSC_*`

Η μεγαλύτερη αλλαγή. Ο άξονας `Z` έγινε `D` (Down) και το `XY` έγινε `NE` (North/East),
ώστε να ταιριάζει με το σύστημα συντεταγμένων **NED** που χρησιμοποιείται εσωτερικά.

### Κάθετος άξονας

| 4.6 και παλιότερα | 4.7 / 4.8 | Μετατροπή τιμής |
|---|---|---|
| `PSC_POSZ_P` | `PSC_D_POS_P` | ίδια |
| `PSC_VELZ_P` | `PSC_D_VEL_P` | ίδια |
| `PSC_VELZ_I` | `PSC_D_VEL_I` | ίδια |
| `PSC_VELZ_D` | `PSC_D_VEL_D` | ίδια |
| `PSC_VELZ_FF` | `PSC_D_VEL_FF` | ίδια |
| `PSC_VELZ_IMAX` | `PSC_D_VEL_IMAX` | **× 0.01** |
| `PSC_VELZ_FLTE` | `PSC_D_VEL_FLTE` | ίδια |
| `PSC_VELZ_FLTD` | `PSC_D_VEL_FLTD` | ίδια |
| `PSC_ACCZ_P` | `PSC_D_ACC_P` | **× 0.1** |
| `PSC_ACCZ_I` | `PSC_D_ACC_I` | **× 0.1** |
| `PSC_ACCZ_D` | `PSC_D_ACC_D` | **× 0.1** |
| `PSC_ACCZ_FF` | `PSC_D_ACC_FF` | ίδια ⚠ — δες παρακάτω |
| `PSC_ACCZ_IMAX` | `PSC_D_ACC_IMAX` | **× 0.001** |
| `PSC_ACCZ_FLTT` | `PSC_D_ACC_FLTT` | ίδια |
| `PSC_ACCZ_FLTE` | `PSC_D_ACC_FLTE` | ίδια |
| `PSC_ACCZ_FLTD` | `PSC_D_ACC_FLTD` | ίδια |
| `PSC_ACCZ_SMAX` | `PSC_D_ACC_SMAX` | ίδια |
| `PSC_ACCZ_PDMX` | `PSC_D_ACC_PDMX` | ίδια ⚠ — δες παρακάτω |
| `PSC_ACCZ_D_FF` | `PSC_D_ACC_D_FF` | ίδια ⚠ — δες παρακάτω |
| `PSC_ACCZ_NTF` / `NEF` | `PSC_D_ACC_NTF` / `NEF` | ίδια |
| `PSC_JERK_Z` | `PSC_D_JERK` | ίδια (ήδη m/s³) |

> **⚠ Τρεις παράμετροι που η αυτόματη μετατροπή ΔΕΝ κλιμακώνει — έλεγξέ τις με το χέρι**
>
> Οι `PSC_ACCZ_FF`, `PSC_ACCZ_PDMX` και `PSC_ACCZ_D_FF` **μεταφέρονται αυτούσιες**: στο
> `AC_PosControl::convert_parameters()` βρίσκονται στον πίνακα `conversion_info`, που περνάει
> από την **ακλιμάκωτη** `AP_Param::convert_old_parameters()`. Μόνο τα `P`, `I`, `D` είναι
> στον `conversion_info_01` με συντελεστή `0.1`.
>
> Η **τεκμηρίωση** όμως των ίδιων παραμέτρων λέει το αντίθετο — π.χ. το `@Description` του
> `PSC_D_ACC_FF` γράφει *"If upgrading from 4.6 this is _ACCZ_FF * 0.1"*. Είναι δηλαδή
> **ασυμφωνία μέσα στον ίδιο τον ArduPilot**, όχι δική σου ρύθμιση.
>
> **Τι σημαίνει πρακτικά:** αν είχες μη-μηδενική τιμή σε κάποια από τις τρεις στην 4.6, μετά
> την αναβάθμιση θα βρεθείς με τιμή **10× μεγαλύτερη** από τη σωστή. Διαίρεσέ τη με το 10
> χειροκίνητα. Στα περισσότερα drones και οι τρεις είναι 0 (default), οπότε δεν σε αφορά —
> αλλά **έλεγξέ το** πριν πετάξεις.

### Οριζόντιο επίπεδο

| 4.6 και παλιότερα | 4.7 / 4.8 | Μετατροπή τιμής |
|---|---|---|
| `PSC_POSXY_P` | `PSC_NE_POS_P` | ίδια |
| `PSC_VELXY_P` | `PSC_NE_VEL_P` | ίδια |
| `PSC_VELXY_I` | `PSC_NE_VEL_I` | ίδια |
| `PSC_VELXY_D` | `PSC_NE_VEL_D` | ίδια |
| `PSC_VELXY_FF` | `PSC_NE_VEL_FF` | ίδια |
| `PSC_VELXY_IMAX` | `PSC_NE_VEL_IMAX` | **× 0.01** |
| `PSC_VELXY_FLTE` | `PSC_NE_VEL_FLTE` | ίδια |
| `PSC_VELXY_FLTD` | `PSC_NE_VEL_FLTD` | ίδια |
| `PSC_JERK_XY` | `PSC_NE_JERK` | ίδια (ήδη m/s³) |
| `PSC_ANGLE_MAX` | `PSC_ANGLE_MAX` | αμετάβλητη |

---

## Α.2 Waypoint Navigation — `WPNAV_*` → `WP_*`

Άλλαξε το πρόθεμα **και** οι μονάδες (cm → m, δηλαδή **× 0.01**).

| 4.6 και παλιότερα | 4.7 / 4.8 | Μετατροπή |
|---|---|---|
| `WPNAV_SPEED` (cm/s) | `WP_SPD` (m/s) | **× 0.01** |
| `WPNAV_RADIUS` (cm) | `WP_RADIUS_M` (m) | **× 0.01** |
| `WPNAV_SPEED_UP` (cm/s) | `WP_SPD_UP` (m/s) | **× 0.01** |
| `WPNAV_SPEED_DN` (cm/s) | `WP_SPD_DN` (m/s) | **× 0.01** |
| `WPNAV_ACCEL` (cm/s²) | `WP_ACC` (m/s²) | **× 0.01** |
| `WPNAV_ACCEL_Z` (cm/s²) | `WP_ACC_Z` (m/s²) | **× 0.01** |
| `WPNAV_ACCEL_C` (cm/s²) | `WP_ACC_CNR` (m/s²) | **× 0.01** |
| `WPNAV_JERK` | `WP_JERK` | ίδια (ήδη m/s³) |
| `WPNAV_RFND_USE` | `WP_RFND_USE` | ίδια |
| `WPNAV_TER_MARGIN` | `WP_TER_MARGIN` | ίδια |

> **Παράδειγμα:** `WPNAV_SPEED = 500` (5 m/s) γίνεται `WP_SPD = 5.0`.

---

## Α.3 Loiter — `LOIT_*`

| 4.6 και παλιότερα | 4.7 / 4.8 | Μετατροπή |
|---|---|---|
| `LOIT_SPEED` (cm/s) | `LOIT_SPEED_MS` (m/s) | **× 0.01** |
| `LOIT_ACC_MAX` (cm/s²) | `LOIT_ACC_MAX_M` (m/s²) | **× 0.01** |
| `LOIT_BRK_ACCEL` (cm/s²) | `LOIT_BRK_ACC_M` (m/s²) | **× 0.01** |
| `LOIT_BRK_JERK` (cm/s³) | `LOIT_BRK_JRK_M` (m/s³) | **× 0.01** |
| `LOIT_BRK_DELAY` | `LOIT_BRK_DELAY` | αμετάβλητη |
| `LOIT_ANG_MAX` | `LOIT_ANG_MAX` | αμετάβλητη |
| `LOIT_OPTIONS` | `LOIT_OPTIONS` | αμετάβλητη |

---

## Α.4 Pilot input — `PILOT_*`

| 4.6 και παλιότερα | 4.7 / 4.8 | Μετατροπή |
|---|---|---|
| `PILOT_SPEED_UP` (cm/s) | `PILOT_SPD_UP` (m/s) | **× 0.01** |
| `PILOT_SPEED_DN` (cm/s) | `PILOT_SPD_DN` (m/s) | **× 0.01** |
| `PILOT_ACCEL_Z` (cm/s²) | `PILOT_ACC_Z` (m/s²) | **× 0.01** |
| `PILOT_TKOFF_ALT` (cm) | `PILOT_TKO_ALT_M` (m) | **× 0.01** |
| `PILOT_THR_FILT` | `PILOT_THR_FILT` | αμετάβλητη |
| `PILOT_THR_BHV` | `PILOT_THR_BHV` | αμετάβλητη |
| `PILOT_Y_RATE`, `PILOT_Y_EXPO`, `PILOT_Y_RATE_TC` | ίδια | αμετάβλητες |

---

## Α.5 Attitude Controller — γωνιακές επιταχύνσεις

Μετονομάστηκαν και άλλαξαν μονάδες από **centidegrees** σε **degrees** (× 0.01).

| 4.6 και παλιότερα | 4.7 / 4.8 | Μετατροπή |
|---|---|---|
| `ATC_ACCEL_R_MAX` (cdeg/s²) | `ATC_ACC_R_MAX` (deg/s²) | **× 0.01** |
| `ATC_ACCEL_P_MAX` (cdeg/s²) | `ATC_ACC_P_MAX` (deg/s²) | **× 0.01** |
| `ATC_ACCEL_Y_MAX` (cdeg/s²) | `ATC_ACC_Y_MAX` (deg/s²) | **× 0.01** |
| `ATC_SLEW_YAW` (cdeg/s) | `ATC_RATE_WPY_MAX` (deg/s) | **× 0.01** |
| `ANGLE_MAX` (cdeg, παράμετρος Copter) | `ATC_ANGLE_MAX` (deg) | **× 0.01** — και μετακινήθηκε στην ομάδα `ATC_` |

> **Παραδείγματα:** `ATC_ACCEL_R_MAX = 110000` γίνεται `ATC_ACC_R_MAX = 1100`·
> `ANGLE_MAX = 4500` γίνεται `ATC_ANGLE_MAX = 45`.

---

## Α.6 Arming checks

Άλλαξε η **λογική**, όχι μόνο το όνομα.

| 4.6 και παλιότερα | 4.7 / 4.8 |
|---|---|
| `ARMING_CHECK` — bitmask των ελέγχων που **εκτελούνται** | `ARMING_SKIPCHK` — bitmask των ελέγχων που **παρακάμπτονται** |

| Πρόθεση | 4.6 | 4.7+ |
|---|---|---|
| Όλοι οι έλεγχοι ενεργοί | `ARMING_CHECK = 1` (ALL) | `ARMING_SKIPCHK = 0` |
| Όλοι απενεργοποιημένοι | `ARMING_CHECK = 0` | `ARMING_SKIPCHK = -1` |

Ο κώδικας κάνει την **αντιστροφή αυτόματα** κατά την αναβάθμιση.

> **Και στις δύο εκδόσεις:** η σωστή τιμή για κανονική χρήση είναι "όλοι οι έλεγχοι ενεργοί".

---

## Α.7 Τι **δεν** άλλαξε

Οι παρακάτω ομάδες, που είναι και οι πιο κρίσιμες για το tuning των κεφαλαίων 5–7,
**παρέμειναν ίδιες**:

| Ομάδα | Παραδείγματα |
|---|---|
| `ATC_RAT_*` | `ATC_RAT_RLL_P`, `ATC_RAT_PIT_D`, `ATC_RAT_YAW_I`, `ATC_RAT_*_D_FF` |
| `ATC_ANG_*` | `ATC_ANG_RLL_P`, `ATC_ANG_PIT_P`, `ATC_ANG_YAW_P` |
| `ATC_ACC_*` | `ATC_ACC_R_MAX`, `ATC_ACC_P_MAX`, `ATC_ACC_Y_MAX` |
| `ATC_THR_MIX_*` | `ATC_THR_MIX_MIN`, `ATC_THR_MIX_MAX`, `ATC_THR_MIX_MAN` |
| `ATC_INPUT_TC`, `ATC_ANG_LIM_TC` | ίδια ονόματα — αλλά **το default του `ATC_INPUT_TC` άλλαξε** (§Α.9) |
| `INS_HNTCH_*`, `INS_HNTC2_*` | Όλες οι παράμετροι harmonic notch |
| `INS_GYRO_FILTER`, `INS_FAST_SAMPLE`, `INS_GYRO_RATE` | — |
| `INS_LOG_BAT_*` | — |
| `FFT_*` | Όλες |
| `MOT_*` | `MOT_THST_HOVER`, `MOT_THST_EXPO`, `MOT_SPIN_MIN` κ.λπ. |
| `SERVO_BLH_*`, `SERVO_DSHOT_*` | — |

> **Πρακτικά:** τα κεφάλαια **5 και 6** — η καρδιά του tuning — ισχύουν αυτούσια και για
> 4.6. Οι μετονομασίες αφορούν κυρίως τα κεφάλαια **7 έως 10**.

---

## Α.8 Γρήγορος κανόνας μετατροπής

Αν διαβάζεις παλιότερο tutorial με τιμές σε cm:

```
τιμή_σε_m = τιμή_σε_cm / 100
```

| Παλιά τιμή | Νέα τιμή |
|---|---|
| `WPNAV_SPEED = 1000` | `WP_SPD = 10.0` |
| `LOIT_SPEED = 1250` | `LOIT_SPEED_MS = 12.5` |
| `PILOT_SPEED_UP = 250` | `PILOT_SPD_UP = 2.5` |
| `WPNAV_RADIUS = 200` | `WP_RADIUS_M = 2.0` |

Για τα κέρδη του κάθετου controller η μετατροπή **δεν είναι ×0.01** — δες τον πίνακα Α.1.

---

## Α.9 Ίδιο όνομα, διαφορετικό default

Αυτές οι παράμετροι **δεν μετονομάστηκαν**, οπότε δεν εμφανίζονται σε κανέναν πίνακα
μετατροπής — αλλά η εργοστασιακή τους τιμή άλλαξε. Αν αναβαθμίσεις, η δική σου
αποθηκευμένη τιμή διατηρείται· αν όμως κάνεις reset ή διαβάζεις παλιότερο tutorial, θα δεις
διαφορά.

| Παράμετρος | Default 4.6 | Default 4.7 / 4.8 | Σχόλιο |
|---|---|---|---|
| `ATC_INPUT_TC` | 0.15 (Medium) | **0.10 (Crisp)** | PR 32643. Πιο άμεσο αίσθημα stick. Δες [κεφ. 7 §7.6](07-Stabilisation-Mode-Tuning.md) |

> Το 0.15 εξακολουθεί να είναι το default του **Plane** — γι' αυτό πολλά κείμενα το
> αναφέρουν ακόμα ως «η εργοστασιακή τιμή».

---

**Επιστροφή:** [README — Ευρετήριο](README.md)
