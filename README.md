# GB SRSB Vivaha Seva — Admin Portal

**GB SRS Brundavan Matrimonial Service**
Registered Charity No. 1150660 · Cowley, Uxbridge

---

## Overview

A secure, web-based admin portal for managing matrimonial profiles collected at GB SRS Brundavan's Vivaha Seva events. The system allows trustees and coordinators to search, filter, view, edit and manage profiles, and to import data from past event spreadsheets.

**Live URL:** `https://gbsrsbrundavan.github.io/gbsrsb-matrimony`

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | Single-file HTML/CSS/JS (vanilla) |
| Database | Firebase Firestore (NoSQL) |
| Authentication | Firebase Auth (email + password) |
| Hosting | GitHub Pages |
| Project | `gbsrs-matrimony` (Firebase) |

---

## Features

### Dashboard
- Live stat cards — Total profiles, Brides, Grooms, Matched, Needs Review
- Click the ⚠ Needs Review card to instantly filter flagged profiles

### Profile Management
- Add, edit, view and withdraw profiles
- Full profile schema covering identity, contact, community/astrology, personal, location, education, family background, partner preferences and coordinator notes
- Soft delete (withdraw) — data is retained, not permanently deleted

### Search & Filter
- Free-text search across name, community, gotra, profession, city, languages, email and notes
- Filter dropdowns: Gender, Age Group, Status
- Review flag filter — shows only profiles needing coordinator attention
- Pagination: 25 / 50 / 100 / Show all rows per page
- Export visible/filtered profiles to CSV

### Import from Spreadsheet
- Claude.ai processes Excel files and extracts structured data into a JSON file
- Upload the JSON file to import profiles in bulk
- Profiles registered by parents/siblings are automatically flagged ⚠ for review
- Cross-file deduplication by email address

### Security
- Firebase Authentication — email + password login required
- Firestore Security Rules — only verified admin accounts can read or write data
- Three-tier admin structure: superadmin / admin roles stored in Firestore `admins` collection
- No data stored on GitHub — all profiles live in Firestore

---

## Data Structure

### Firestore Collections

```
profiles/           — All matrimonial profiles
  {auto-id}/
    candidateFirstName, candidateLastName, title
    gender (Bride | Groom)
    registeredBy (Self | Parent | Sibling | Other)
    registrantName, registrantContact
    email, whatsapp, preferredContact
    dob, timeOfBirth, placeOfBirth, ageGroup
    community, subSect, gotra, nakshatra, rashi, gana, nadi, mutt
    motherTongue, religion, languages, nativePlace
    height, dietary, smokes, drinks, maritalStatus, horoscopeRequired
    country, city, visaStatus, willingToRelocate
    education, educationField, profession, employerType
    fatherName, fatherOccupation, motherName, motherOccupation
    numBrothers, numSisters, familyType
    aboutMe, hobbies
    preferredAgeMin, preferredAgeMax, preferredCommunity
    preferredLocation, preferredSmokes, preferredDrinks
    attendedEvent, registeredEvent
    status (active | matched | withdrawn | pending)
    needsReview (boolean), needsReviewReason
    coordinatorNotes
    createdAt, updatedAt (server timestamps)

admins/             — Admin user registry
  {uid}/            — Document ID must match Firebase Auth UID exactly
    email
    name
    role (superadmin | admin)
```

---

## Events Imported

| Event | Profiles |
|---|---|
| Matrimonial Meet May 2026 | 80 |
| Matrimonial Meet (earlier) | 103 (after deduplication) |
| Matrimonial Meet March 2025 | 84 (after deduplication) |
| **Total** | **235** |

Duplicates (same email across events) are removed automatically during import — the earliest registration is kept.

---

## Admin Setup

### Adding a New Admin

1. Go to **Firebase Console → Authentication → Users → Add user**
2. Enter their email and a temporary password
3. Copy the User UID shown in the users list
4. Go to **Firestore → admins collection → Add document**
5. Set Document ID = User UID (exact match)
6. Add fields: `email`, `name`, `role` (`admin` or `superadmin`)
7. Share the temporary password with the new admin securely

### Removing an Admin

1. Delete their document from the `admins` Firestore collection
2. Optionally disable or delete their account in Firebase Authentication

---

## Firestore Security Rules

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /profiles/{profileId} {
      allow read, write: if request.auth != null &&
        exists(/databases/$(database)/documents/admins/$(request.auth.uid));
    }
    match /admins/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if false;
    }
  }
}
```

---

## Importing New Event Data

When a new Matrimonial Meet is held:

1. Export registrations from your form tool as an Excel (.xlsx) file
2. Upload the file(s) to **Claude.ai** and ask:
   > *"Please extract all matrimonial profiles from this spreadsheet and produce a structured JSON file ready for import into the Vivaha Seva app."*
3. Download the JSON file Claude produces
4. Log in to the admin portal → **Import from Spreadsheet** tab
5. Upload the JSON file → review the profile cards
6. Click **Import All to Firestore** and confirm

---

## GDPR Notes

This system stores special category personal data under UK GDPR (community, religion, personal details). Responsibilities:

- Data is held under legitimate interests as a registered charity facilitating a community service
- Registrants consented at the point of registration (event sign-up forms)
- Data should not be retained beyond **2 years** without re-consent
- Any deletion request from a registrant should be actioned by withdrawing the profile and then deleting the Firestore document
- Only named admin users should have access — review the admins collection periodically
- CSV exports contain full personal data and must not be shared outside the admin team
- A formal privacy notice should be provided to registrants at future events

---

## Planned Future Development

| Phase | Feature |
|---|---|
| Phase 2 | Registrant self-service login (email + password) |
| Phase 2 | Candidate profile creation and editing |
| Phase 2 | "Compatible profiles" view — limited info, no contact details |
| Phase 2 | "Express Interest" button — coordinator-mediated introduction |
| Phase 3 | Public registration form for future events (feeds directly into Firestore) |
| Phase 3 | Custom domain: `vivaha.gb-srsbrundavan.org` |
| Phase 3 | Collaborating organisation data-sharing (federated model) |
| Phase 3 | Match outcome tracking and anonymised reporting |

---

## Repository Structure

```
gbsrsb-matrimony/
  index.html          — Complete single-file application
  README.md           — This file
```

JSON import files are not committed to the repository as they contain personal data.

---

## Contact

**GB SRS Brundavan**
55 High Street, Cowley, Uxbridge, UB8 2DZ
[gb-srsbrundavan.org](https://gb-srsbrundavan.org)
Registered Charity No. 1150660
