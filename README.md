# DJ-læringsprogram v3 — Setup Guide (Google-login)

## Hvad du får

Et selvstændigt HTML-program med:
- 8 ugers struktureret DJ-læringsprogram
- YouTube-links direkte ved hver relevante opgave
- **Google-login** — dine data følger med på tværs af PC, iPad og telefon
- Real-time synkronisering: check noget på PC'en, og det opdateres på iPad'en med det samme
- Fallback til localStorage hvis Firebase ikke er tilgængelig

## Din config er allerede indsat

Din Firebase-config står allerede i filen:
- Projekt: `dj-ddj-11bfd`
- Auth domain: `dj-ddj-11bfd.firebaseapp.com`

Så du skal ikke redigere HTML-filen. Der er tre ting du skal gøre i Firebase Console, og så skal filen op på GitHub Pages.

---

## Trin 1: Aktivér Google-login i Firebase (2 min)

1. Gå til [console.firebase.google.com](https://console.firebase.google.com/)
2. Vælg dit projekt **dj-ddj-11bfd**
3. Venstre menu: **Build → Authentication**
4. Fanen **Sign-in method**
5. Hvis **Anonymous** stadig er slået til fra sidste version, kan du lade den være (det gør ingen skade) eller slå den fra
6. Klik på **Google** i listen
7. Slå **Enable** til
8. **Support email:** vælg din egen email
9. Klik **Save**

## Trin 2: Firestore-regler (2 min)

Sikkerhedsreglerne skal justeres — nu bruger vi den fulde Google UID i stedet for anonym UID (samme regel virker faktisk, men lad os bekræfte):

1. Venstre menu: **Firestore Database**
2. Fanen **Rules**
3. Sørg for at der står:

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

4. Klik **Publish**

Reglen siger: kun logget-ind brugere kan læse/skrive, og kun deres eget dokument.

## Trin 3: Upload til GitHub

1. Omdøb `dj-laeringsprogram-v3.html` til `index.html`
2. Upload til dit `dj-program`-repo (eller opret et nyt)
3. Aktivér GitHub Pages: **Settings → Pages → Deploy from branch → main / root**
4. Vent 1-2 minutter

Din side er nu på:
`https://[dit-username].github.io/dj-program/`

## Trin 4: Autoriser dit domæne i Firebase (kritisk!)

**Uden dette trin virker Google-login IKKE på GitHub Pages.**

1. Firebase Console → **Authentication → Settings**
2. Fanen **Authorized domains**
3. Du bør allerede se `localhost` og `dj-ddj-11bfd.firebaseapp.com`
4. Klik **Add domain**
5. Skriv: `[dit-username].github.io` (kun domænet, uden `https://` og uden `/dj-program`)
6. Klik **Add**

---

## Sådan virker det

**Første besøg:**
- Login-vindue vises
- Klik "Log ind med Google" → Google-popup → vælg konto
- Popup lukker → du er inde

**Efter login:**
- Din Google-avatar og navn vises øverst i venstre hjørne
- "Synkroniseret" med grøn prik øverst i højre hjørne
- Alle checks gemmes automatisk i Firebase

**Fra anden enhed (fx iPad):**
- Åbn samme URL
- Log ind med samme Google-konto
- Dit fremskridt vises med det samme
- Checker du noget på PC'en, opdateres iPad'en real-time

**Log ud:**
- Klik "Log ud"-knappen i header
- Login-vindue vises igen

---

## Fejlfinding

### "Login fejlede: unauthorized domain"

Du har glemt trin 4. Tilføj din GitHub Pages-URL under Authorized domains.

### Popup lukker med det samme uden at logge mig ind

- Tjek at popups ikke er blokeret af browseren
- Filen falder automatisk tilbage til redirect-flow hvis popup blokeres
- Ved redirect skal du klikke Google-knappen én ekstra gang efter du kommer tilbage til siden

### Jeg kan ikke se dine data fra en anden enhed

- Er du logget ind med **samme** Google-konto på begge enheder?
- Tjek "Synkroniseret" med grøn prik er synlig
- F12 → Console for at se om der er fejl

### Vil ikke logge mig ind på iPad Safari

Safari på iOS har strikse regler om cross-site cookies. Hvis popup fejler:
- Tillad "Cross-Site Tracking" for `firebaseapp.com` i iOS Settings → Safari → Advanced
- Eller brug Chrome på iPad

### "Firebase: Error (auth/operation-not-allowed)"

Google-login er ikke aktiveret. Gå tilbage til trin 1.

### Data forsvinder ikke fra localStorage

localStorage bruges nu kun som midlertidig cache før login. Efter login synkroniseres alt fra Firebase. Hvis du vil rydde: F12 → Application → Local Storage → slet nøglen `dj-program-progress-v2`.

---

## Sikkerhed

- **API-key i HTML:** Fint. Firebase API-keys er offentlige — de identificerer projektet, ikke brugeren
- **Din sikkerhed:** Kommer 100% fra Firestore Rules i trin 2
- **Data i cloud:** Kun du (via din Google-konto) kan læse/skrive dit dokument
- **Ingen andre kan se dit fremskridt** — reglerne blokerer det

## Gratis grænser

Firebase Spark plan (gratis):
- Authentication: ubegrænset Google-logins
- Firestore: 50k reads, 20k writes, 1 GB storage per dag
- **Du kommer aldrig i nærheden af grænserne** med dette program

---

## Overgang fra v2 (anonym)

Hvis du har brugt v2 (anonym auth) og har progression du vil beholde:

1. Åbn v2 i din browser og notér hvilke opgaver du har checket
2. Skift til v3
3. Log ind med Google
4. Check opgaverne manuelt igen

Der er desværre ingen automatisk migration fordi anonym auth og Google auth giver forskellige user IDs. Beklager — men det er første og eneste gang du skal gøre det.
