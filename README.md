# Kiri Cars Website

Kiri Cars is a car marketplace website built with HTML, CSS, JavaScript, Firebase Authentication, and Cloud Firestore.

> **Project status:** This documentation describes the intended structure and behaviour of the project. The website source files were not present in the repository when this README was checked, so the implementation and current runtime behaviour still need to be verified against the actual files.

## Main features

- Public homepage for browsing cars.
- Search/filter cars by available vehicle details.
- Firebase email/password login.
- Owner dashboard for managing admin accounts and viewing car totals.
- Admin dashboard for adding vehicles and managing vehicles added by that admin.
- Firestore-backed vehicle listings.

## Source files

The intended project contains these nine files:

| File | Purpose |
| --- | --- |
| `index.html` | Public homepage and vehicle search interface |
| `app.js` | Loads and filters public vehicle listings |
| `style.css` | Shared page styling and responsive layout |
| `firebase-config.js` | Firebase app and service initialization |
| `login.html` | Login page |
| `owner.html` | Owner dashboard interface |
| `owner.js` | Owner-only admin management and dashboard logic |
| `admin.html` | Admin dashboard and vehicle entry form |
| `admin.js` | Admin authentication checks and vehicle management |

## Technology

- HTML5
- CSS
- JavaScript ES modules
- Firebase Authentication
- Cloud Firestore

## Firebase data model

### `admins/{uid}`

Each account is expected to have a Firestore document whose document ID matches that user's Firebase Authentication UID.

Expected fields:

| Field | Type | Meaning |
| --- | --- | --- |
| `name` | String | Display name |
| `email` | String | Account email |
| `role` | String | `owner` or `admin` |
| `active` | Boolean | Must be the Boolean value `true` for an enabled account |
| `createdAt` | Timestamp | Account creation time, when recorded |
| `createdBy` | String | UID of the owner who created an admin, when recorded |

Important: `active: true` must be a Boolean, not the text string `"true"`. The document ID must exactly match the signed-in user's UID.

### `cars/{carId}`

A vehicle listing may contain fields such as:

- `manufacturer`
- `carType`
- `model`
- `year`
- `price`
- `status` (for example, locally used, fresh import, or brand new)
- `mileage`
- `fuel`
- `transmission`
- `description`
- `imageUrl` (optional)
- `addedBy` (the creator's Firebase Authentication UID)
- `createdAt` (Firestore timestamp)

Field names in this section describe the intended model; verify them against the source code before changing database data.

## Intended access model

- Public visitors can read car listings.
- The owner can manage admin profiles and all vehicle listings.
- An enabled admin can add vehicle listings.
- An admin can update or delete only listings whose `addedBy` value matches that admin's UID.
- An account must be enabled and have the expected role before opening the relevant dashboard.

These permissions must be enforced by Firestore Security Rules as well as by the front-end code. Hiding a button or redirecting a page is not a security boundary by itself.

## Setup checklist

1. Ensure the nine source files are in the repository root, with the exact filenames listed above.
2. Check `firebase-config.js` and confirm it points to the intended Firebase project.
3. In Firebase Authentication, enable the sign-in method used by the login page (expected: Email/Password).
4. Create the owner account in Firebase Authentication.
5. Create `admins/{ownerUid}` in Firestore using the exact Authentication UID as the document ID, and set `role` to `owner` and `active` to Boolean `true`.
6. Publish Firestore Security Rules that match the intended owner/admin access model.
7. Test owner login, admin creation, admin login, vehicle creation, public listing, and admin ownership restrictions.
8. Deploy the static website to a hosting service and configure the relevant domain and Firebase authorized domains.

Never commit private service-account keys, passwords, or other server-side secrets. Firebase web configuration is designed to be included in a web client, but access to data must be protected by Security Rules.

## Troubleshooting: login says the account is inactive

If the page reports that the profile was found but the `active` field is undefined or is not Boolean `true`, check all of the following:

1. Open Firebase Console and select the correct Firebase project.
2. Open Firestore Database and locate the `admins` collection.
3. Open the document whose ID exactly matches the UID shown in Firebase Authentication for the account being used.
4. Confirm there is a field named exactly `active`.
5. Set its field type to **boolean** and its value to **true**. Do not enter the string `"true"`.
6. Confirm `role` is exactly `owner` for the owner account or `admin` for an admin account.
7. Save and try logging in again.

If login still fails, capture the exact on-screen error and inspect the current contents of `login.html` and `firebase-config.js`. Confirm that Authentication and Firestore are using the same Firebase project.

## Debugging checklist

- Confirm all HTML pages import the correct JavaScript filenames and relative paths.
- Check the browser developer console for module import errors and Firebase errors.
- Confirm Firebase Authentication returns a UID after successful sign-in.
- Confirm the corresponding Firestore profile document exists at `admins/{uid}`.
- Check exact field names, types, and role values.
- Review Firestore Security Rules if a read or write returns `permission-denied`.
- Do not weaken database rules to public read/write as a troubleshooting shortcut.

## Current verification status

The repository was checked while preparing this document and contained no source files. This README was added as the first project file. The actual website code must be committed before a reliable code review or diagnosis of the remaining issue can be completed.
