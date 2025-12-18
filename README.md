## תיעוד API ציבורי – Leads (לידים)

**Base URL**: `https://api.reshetch.org/api/pub`

המסמך מתאר אך ורק את מה שמוגדר תחת: `apps/backend-reshet/src/api/public/leads`.

---

## Endpoints

### 1) יצירת ליד

**POST** `/leads/`

- **Middlewares**: `leadsMiddleware` + ולידציה ע"י `CreateLeadSchema`
- **תשובה**:
  - **201**: `{ "message": "Lead created successfully" }`
  - **500**: `{ "message": "Failed to create lead", "error": "..." }`

#### Body (JSON)

##### שדות חובה

- **institutionId** _(number)_: מזהה מוסד.
- **operationId** _(number)_: מזהה אופרציה/קמפיין.
- **institutionType** _(string)_: סוג מוסד. ערכים מותרים:
  - `"גן"`
  - `"בית ספר"`
  - `"על יסודי"`
  - `"בית ספר שלהבות"`

##### שדות רשות

> ברוב השדות למטה מותר לשלוח `null` או מחרוזת ריקה `""`.

- **childFirstName** _(string | null)_: שם פרטי ילד.
- **parentFirstName** _(string | null)_: שם פרטי הורה.
- **parentLastName** _(string | null)_: שם משפחה הורה.
- **parentPhone** _(string | null)_: טלפון הורה.
- **parentEmail** _(string | null)_: אימייל הורה (אם נשלח – חייב להיות בפורמט אימייל תקין).
- **address** _(string | null)_: כתובת.
- **comment** _(string | null)_: הערה.
- **arrivalSource** _(string | null)_: מקור הגעה. ערכים מותרים:
  - `"פייסבוק"`
  - `"אינסטגרם"`
  - `"טיקטוק"`
  - `"גוגל"`
  - `"חבר מביא חבר"`
  - `"אחר"`
- **religiousStatus** _(string | null)_: סטטוס דתי. ערכים מותרים:
  - `"חילוני"`
  - `"מסורתי"`
  - `"דתי"`
  - `"חרדי"`
- **gradeLevel** _(string | null)_: שכבת גיל / כיתה.
  - הערה: לפי הסכמה כאן זה _string חופשי_ (לא מוגבל לערכי enum).
- **upgradeToGrade** _(string | null)_: עולה לכיתה. ערכים מותרים:
  - `"טרום טרום"`, `"טרום"`, `"חובה"`, `"א"`, `"ב"`, `"ג"`, `"ד"`, `"ה"`, `"ו"`
- **hasSiblingShalhavot** _(string | null)_: אח/ות שלמד/לומד בשלהבות. ערכים מותרים:
  - `"כן"`
  - `"לא"`
- **hasSiblingInChabad** _(string | null)_: אח/ות בגן חב"ד. ערכים מותרים:
  - `"כן"`
  - `"לא"`
- **specialNeeds** _(string | null)_: צרכים מיוחדים.
- **academicYear** _(string | null)_: שנת לימודים.
- **gender** _(string | null)_: מגדר. ערכים מותרים:
  - `"זכר"`
  - `"נקבה"`
- **check** _(boolean)_: האם המשתמש אישר דיוור

#### הערות התנהגות (לפי ה־controller)

- בעת יצירה השרת קובע אוטומטית:
  - `isNew = true`
  - `isTransfer = false`
  - `status = "חדש"`
- אם נשלח `position`, השרת שומר `geom` כ־Point עם קואורדינטות `[lng, lat]`.
- לאחר יצירה:
  - אם קיים `institutionId`: נשלחת הודעת וואטסאפ לכל אנשי צוות עם תפקיד `REGISTRATION_MANAGER` במוסד.
  - אחרת, אם נשלח `staffMemberId`: נשלחת הודעה לאיש צוות יחיד (fallback).

---

### 2) עדכון ליד לפי טלפון

**PUT** `/leads/:phone`

- **Body**: לא קיימת ולידציה בסכמה/ראוט עבור עדכון (השרת מעביר את `req.body` ישירות ל־update).
- **תשובה**:
  - **200**: `{ "message": "Lead updated successfully" }`
  - **500**: `{ "message": "Failed to update lead", "error": "..." }`

#### Path params

- **phone** _(string)_: טלפון לזיהוי הליד.

#### איך מתבצע ה־match לטלפון (חשוב)

השרת מנרמל את `:phone` לפורמט ישראל (`IL`) ומשווה מול `parentPhone` כשהוא **national** (מספר “לאומי” ללא `+`). כלומר, ניתן לשלוח פורמטים שונים כל עוד הנרמול מצליח.

---

### 3) שליפת ליד לפי טלפון

**GET** `/leads/:phone`

- **תשובה**:
  - **200**: אובייקט הליד שנמצא (או `null` אם לא נמצא).
  - **500**: `{ "message": "Failed to get lead", "error": "..." }`

#### Path params

- **phone** _(string)_: טלפון לזיהוי הליד (מנורמל ל־IL ומשווה ל־`parentPhone` הלאומי).




## תיעוד API ציבורי – Institutions (מוסדות)

**Base URL**: `https://api.reshetch.org/api/pub`

המסמך מתאר אך ורק את מה שמוגדר תחת: `apps/backend-reshet/src/api/public/institutions`.

---

## Endpoints

### 1) רשימת מוסדות (לשימוש כבחירה / dropdown)

**GET** `/institutions/all`

- **Query params**: אין.
- **תשובה**:
  - **200**: מערך אובייקטים בפורמט:
    - **label** _(string)_: טקסט תצוגה בפורמט: `{institutionType} - {institutionName} - {city}` (החלק של העיר מתווסף רק אם יש עיר לא ריקה)
    - **value** _(number)_: `id` של המוסד
  - **500**: `{ "error": "..." }`

#### הערות

- הנתונים נשלפים עם השדות: `id`, `institutionName`, `institutionType`, `city`.
- המיון: לפי `institutionName` בסדר עולה.

---

### 2) רשימת מוסדות "שלהבות"

**GET** `/institutions/shalavot`

- **Query params**: אין.
- **תשובה**:
  - **200**: מערך של רשומות מוסד (כפי שמוחזר מהמודל `Institutions`).
  - **500**: `{ "error": "..." }`

#### סינון

- מוחזרות רק רשומות שבהן: `institutionType = "בית ספר שלהבות"`.

---

### 3) רשימת גנים + פרטי מנהל/ת רישום

**GET** `/institutions/kindergartens`

- **Query params**: אין.
- **תשובה**:
  - **200**: מערך של רשומות גן (כפי שמוחזר מהמודל `Institutions`) כולל `registrationManagerStaff`.
  - **500**: `{ "error": "..." }`

#### סינון

- מוחזרות רק רשומות שבהן: `institutionType = "גן"`.

#### הרחבות (include)

מצורף אובייקט `registrationManagerStaff` (alias) מהמודל `StaffMembers` עם השדות:

- **id**
- **firstName**
- **lastName**
- **nickName**
- **phoneWa**
- **phone**
- **mail**
- **tags**

---

## ערכים אפשריים

### InstitutionType

השדה `institutionType` במערכת משתמש בערכים:

- `"גן"`
- `"בית ספר"`
- `"על יסודי"`
- `"בית ספר שלהבות"`

