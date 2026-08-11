# DJ-læringsprogram — Setup Guide

## Hvad du får

Et selvstændigt HTML-program med:
- 8 ugers struktureret DJ-læringsprogram
- YouTube-links direkte ved hver relevante opgave
- Automatisk synkronisering af dine checks via Firebase
- Fallback til browserens localStorage hvis Firebase ikke er sat op
- Virker på tværs af enheder (samme URL = samme fremskridt)

## Trin 1: Firebase-projekt (5-10 minutter)

### 1.1 Opret projekt

1. Gå til [console.firebase.google.com](https://console.firebase.google.com/)
2. Log ind med din Google-konto
3. Klik **"Add project"** (eller "Tilføj projekt")
4. Navn: fx `dj-program-lars`
5. Google Analytics: **fravælg** (ikke nødvendigt)
6. Klik **Create project**

### 1.2 Aktivér Anonymous Authentication

1. I venstre menu: **Build → Authentication**
2. Klik **Get started**
3. Under **Sign-in method**, klik på **Anonymous**
4. Slå den **Enable** til
5. Klik **Save**

### 1.3 Opret Firestore Database

1. I venstre menu: **Build → Firestore Database**
2. Klik **Create database**
3. Vælg **Start in production mode**
4. Location: **eur3 (europe-west)** — vælg en europæisk region
5. Klik **Enable**

### 1.4 Konfigurer sikkerhedsregler

Når databasen er oprettet:

1. Gå til fanen **Rules**
2. Erstat indholdet med:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /dj-progress/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

3. Klik **Publish**

Dette sikrer at kun du kan læse/skrive dine egne data.

### 1.5 Hent din config

1. I venstre menu, klik på **⚙️ (tandhjul) → Project settings**
2. Scroll ned til **Your apps**
3. Klik på **web-ikonet (`</>`)** for at oprette en web-app
4. Navn: fx `dj-program-web`
5. **Undlad** at hakke i "Firebase Hosting"
6. Klik **Register app**
7. Kopiér `firebaseConfig`-objektet — det ser sådan ud:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyABC123...",
  authDomain: "dj-program-lars.firebaseapp.com",
  projectId: "dj-program-lars",
  storageBucket: "dj-program-lars.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abc123"
};
```

## Trin 2: Indsæt config i HTML-filen

1. Åbn `dj-laeringsprogram-v2.html` i en teksteditor
2. Find blokken der starter med `// FIREBASE CONFIG` (ca. linje 750)
3. Erstat placeholder-værdierne med dine rigtige værdier fra Firebase
4. Gem filen

## Trin 3: GitHub Pages-opsætning

### 3.1 Opret repository

1. Gå til [github.com](https://github.com) og opret et nyt repo, fx `dj-program`
2. Sæt det til **Public** (kræves for gratis GitHub Pages)

### 3.2 Upload filen

**Nemmeste vej:**

1. Omdøb `dj-laeringsprogram-v2.html` til `index.html`
2. På GitHub-siden af dit nye repo, klik **Add file → Upload files**
3. Træk `index.html` ind
4. Klik **Commit changes**

### 3.3 Aktivér GitHub Pages

1. Gå til **Settings** på dit repo
2. Venstre menu: **Pages**
3. Under **Source**, vælg **Deploy from a branch**
4. Branch: **main** / **root**
5. Klik **Save**
6. Vent 1-2 minutter

Din side er nu tilgængelig på:
`https://[dit-username].github.io/dj-program/`

### 3.4 Autoriser din GitHub Pages-URL i Firebase

Vigtigt sidste trin:

1. Tilbage i Firebase Console → **Authentication → Settings**
2. Fanen **Authorized domains**
3. Klik **Add domain**
4. Tilføj: `[dit-username].github.io` (uden https:// og uden path)
5. Klik **Add**

Uden dette vil anonym login fejle på GitHub Pages, og programmet vil falde tilbage til localStorage.

## Sådan virker synkronisering

- Programmet logger dig automatisk ind som anonym bruger første gang
- Din unikke ID gemmes i browseren
- Alle dine checks synkroniseres real-time til Firebase
- Åbner du samme URL i en anden browser eller enhed, får du **et nyt anonymt ID** — så din progression synkroniserer *ikke* automatisk på tværs af enheder med denne opsætning

### Hvis du vil have adgang fra flere enheder

Du har to muligheder:

**A) Manuel export/import** — kopier localStorage mellem browsere (kompliceret)

**B) Skift til Google-login** — jeg kan lave den om til at bruge din Google-konto som identifikator, så dine data følger dig. Sig til hvis du vil have det.

## Fejlfinding

**Ingen synkronisering / "Lokal" i toppen:**
- Tjek at du har indsat den korrekte config
- Tjek at din GitHub Pages-URL er tilføjet under Authorized domains
- Åbn browserens konsol (F12) og se om der er fejlbeskeder

**Checks forsvinder ikke:**
- Programmet gemmer altid lokalt som backup — så selv uden Firebase virker det
- Ryd browserens localStorage hvis du vil starte forfra: F12 → Application → Local Storage → slet nøglen `dj-program-progress-v2`

**Firestore-omkostninger:**
- Gratis tier: 50.000 læsninger og 20.000 skrivninger om dagen. Du vil aldrig komme i nærheden af dette.

## Gratis grænser (Firebase Spark plan)

- Firestore: 1 GB storage, 50k reads/day, 20k writes/day
- Authentication: ubegrænset anonyme brugere
- **Du kommer aldrig til at betale noget** til dette projekt
