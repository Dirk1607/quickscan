# Technical Reference — Metroplan Quick Scan

## API Functions Reference

### Notification System

**Function:** `showNotification(msg, isError=false)`
- **Description:** Displays a toast notification
- **Parameters:**
  - `msg` (string) — Notification message
  - `isError` (boolean) — Red error style if true
- **Example:**
  ```javascript
  showNotification("Rapport verstuurd!", false);
  showNotification("Fout bij verzending", true);
  ```

### Metro Map Generation

**Function:** `generateMetroMap(scores)`
- **Description:** Creates interactive SVG metro visualization
- **Parameters:**
  - `scores` (object) — Domain scores object
- **Returns:** SVG markup string
- **Used in:** Report generation
- **Domains included:**
  - 11 Metro Lines: STR, PROD, MKT, SAL, OPS, AFS, HR, FIN, IT, QUA, SAF
  - 5 Accelerators: PM, PROC, LEAD, CHG, OWN

### Email Functionality

**Function:** `sendEmailReport(naam, email, bedrijf, rapportContent, scores)`
- **Description:** Sends personalized report to customer
- **Parameters:**
  - `naam` (string) — Customer name
  - `email` (string) — Customer email
  - `bedrijf` (string) — Company name
  - `rapportContent` (string) — HTML report
  - `scores` (object) — All domain scores
- **Requires:** EmailJS configured with public key
- **Side effect:** Calls `notifyAdmin()` on success

**Function:** `notifyAdmin(naam, bedrijf, email, scores)`
- **Description:** Triggers admin notification
- **Parameters:** Same as above
- **Storage:** localStorage `tnl_notifications` (max 50 entries)
- **Webhook:** Can be enabled for production

## Data Structures

### DOMAINS Object
```javascript
{
  STR: "Strategie",
  PROD: "Product / R&D",
  MKT: "Marketing",
  SAL: "Sales",
  OPS: "Operations",
  AFS: "After Sales",
  HR: "HR",
  FIN: "Finance",
  IT: "IT",
  QUA: "Kwaliteit",
  SAF: "Veiligheid",
  PM: "Project Management",
  PROC: "Proces Management",
  LEAD: "Leiderschap",
  CHG: "Change Management",
  OWN: "Eigenaarschap"
}
```

### PROFIEL Structure (for each domain)
```javascript
{
  1: { // Zone 1 - Kritische groeipijn
    badge: "badge-r",
    bar: "bar-r",
    barlbl: "🔴 Kritische groeipijn",
    wat: "Description of situation",
    herk: "How to recognize this",
    impact: "Business impact",
    nieuws: "Positive news",
    visie: "Strategic vision"
  },
  2: { /* Zone 2 - Aandachtspunt */ },
  3: { /* Zone 3 - Goed ontwikkeld */ },
  4: { /* Zone 4 - Sterke troef */ }
}
```

### Notification Structure (localStorage)
```javascript
{
  timestamp: "2026-06-05T10:30:00Z",
  naam: "John Doe",
  bedrijf: "Company Name",
  email: "john@company.be",
  avgScore: "2.5"
}
```

## Score Calculation

**Function:** `calcScores()` → Returns object with domain scores

Scoring logic:
1. For each question, assigns weighted domain contributions
2. Question 1 domain (weight: 3) → Most important
3. Question 2 domain (weight: 2) → Medium
4. Question 3 domain (weight: 1) → Supporting
5. Reverse scoring applied for `inv:true` questions
6. Scores normalized: (total weight points) / (total weight count)
7. Result range: 1.0 - 4.0

## Zone Classification

```javascript
function getZone(score) {
  if (score <= 1.75) return 1;      // 🔴 Kritische
  if (score <= 2.5)  return 2;      // 🟠 Aandachtspunt
  if (score <= 3.25) return 3;      // 🟢 Goed
  return 4;                          // 💙 Sterke troef
}
```

## State Management

### Global Variables
```javascript
rawAns[]      // Raw 1-4 answers per question
scoreVals[]   // Processed scores per question (1-4 or reversed)
currentStep   // Current step (0-7)
```

### Step Flow
- **Step 0:** Introduction & personal info
- **Steps 1-6:** Question blocks (5 questions each)
- **Step 7:** Report & recommendations

## Color System

```css
--red-bar:    #D85A30  /* Zone 1 */
--ora-bar:    #EF9F27  /* Zone 2 */
--grn-bar:    #1D9E75  /* Zone 3 */
--blu-bar:    #378ADD  /* Zone 4 */
```

## Email Template Variables (EmailJS)

```
to_email          → ${email}
customer_name     → ${naam}
company_name      → ${bedrijf}
report_content    → ${rapportContent} (full HTML)
average_score     → ${avgScore}
```

## LocalStorage Keys

```javascript
// Admin notifications
localStorage.getItem('tnl_notifications')

// Retrieve all notifications:
JSON.parse(localStorage.getItem('tnl_notifications') || '[]')

// Clear notifications:
localStorage.removeItem('tnl_notifications')
```

## Debugging

**Enable detailed logging:**
```javascript
// Console output for score calculation
console.log('Scores:', calcScores());

// View all notifications
console.log(JSON.parse(localStorage.getItem('tnl_notifications')));

// Test email sending
sendEmailReport('Test Name', 'test@example.com', 'Test Co', '<p>test</p>', calcScores());
```

## Performance Notes

- 30 questions × 2 domains per question = 60+ domain contributions
- Metro map SVG with 11 lines + 5 accelerators = 16 stations
- ~2KB estimated report size per submission
- localStorage limited to ~5MB (fits ~1000+ submissions)

## Browser Compatibility

| Feature | Chrome | Firefox | Safari | Edge |
|---------|--------|---------|--------|------|
| SVG Logo | ✅ | ✅ | ✅ | ✅ |
| Metro Map | ✅ | ✅ | ✅ | ✅ |
| EmailJS | ✅ | ✅ | ✅ | ✅ |
| Notifications | ✅ | ✅ | ✅ | ✅ |
| localStorage | ✅ | ✅ | ✅ | ✅ |

---

Last Updated: 2026-06-05
