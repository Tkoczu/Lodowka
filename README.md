# 🧊 Moja Lodówka

Progresywna aplikacja webowa (PWA) do zarządzania zawartością lodówki z synchronizacją w czasie rzeczywistym między użytkownikami.

## Funkcje

- 📦 Dodawanie, edytowanie i usuwanie produktów
- 📅 Śledzenie dat ważności (OK / Wkrótce / Przeterminowane)
- 🗂️ Kategorie produktów i filtrowanie
- 🔍 Wyszukiwanie produktów
- 📚 Baza produktów z autouzupełnianiem
- 🤝 Współdzielona lodówka — synchronizacja w czasie rzeczywistym przez Firebase
- 🌙 Tryb ciemny
- 📱 Instalowalna jako PWA (ekran główny iPhone / Android)
- 🔒 Logowanie przez Google — dostęp tylko dla zaproszonych

## Technologie

- Vanilla HTML / CSS / JavaScript (bez frameworków)
- [Firebase Realtime Database](https://firebase.google.com/products/realtime-database) — synchronizacja danych
- [Firebase Authentication](https://firebase.google.com/products/auth) — logowanie przez Google
- [Firebase App Check](https://firebase.google.com/products/app-check) — ochrona przed nieautoryzowanym dostępem
- PWA — manifest + Service Worker (offline)

## Bezpieczeństwo

Aplikacja używa trzech warstw ochrony:

1. **Google Auth** — tylko zalogowani użytkownicy mają dostęp do UI
2. **Firebase Security Rules** — tylko zatwierdzone konta mają dostęp do danych w bazie
3. **Firebase App Check** — blokuje zapytania spoza przeglądarki (skrypty, curl, Postman)

## Uruchomienie własnej instancji

### 1. Firebase

1. Utwórz projekt na [firebase.google.com](https://firebase.google.com)
2. Włącz **Realtime Database** (wybierz region Europe)
3. Włącz **Authentication → Sign-in method → Google**
4. Włącz **App Check → reCAPTCHA v3**
5. Ustaw reguły bazy danych:

```json
{
  "rules": {
    "fridges": {
      "$fridgeId": {
        ".read": "auth != null && (auth.token.email == 'email1@gmail.com' || auth.token.email == 'email2@gmail.com')",
        ".write": "auth != null && (auth.token.email == 'email1@gmail.com' || auth.token.email == 'email2@gmail.com')",
        "items": {
          "$itemId": {
            ".validate": "newData.hasChildren(['id','name','emoji','qty','unit','cat']) && newData.child('name').isString() && newData.child('name').val().length > 0 && newData.child('name').val().length <= 40 && newData.child('qty').isNumber()"
          }
        }
      }
    },
    "$other": { ".read": false, ".write": false }
  }
}
```

### 2. reCAPTCHA

1. Zarejestruj domenę na **https://www.google.com/recaptcha/admin**
2. Wybierz **reCAPTCHA**
3. Skopiuj **Site Key** (nie Secret Key)

### 3. Konfiguracja kodu

W pliku `index.html` uzupełnij:

```javascript
const FIREBASE_CONFIG = {
  apiKey:            "TWÓJ_API_KEY",
  authDomain:        "TWÓJ_PROJEKT.firebaseapp.com",
  databaseURL:       "https://TWÓJ_PROJEKT-default-rtdb.europe-west1.firebasedatabase.app",
  projectId:         "TWÓJ_PROJEKT",
  storageBucket:     "TWÓJ_PROJEKT.firebasestorage.app",
  messagingSenderId: "TWÓJ_SENDER_ID",
  appId:             "TWÓJ_APP_ID"
};
```

```javascript
initializeAppCheck(app, {
  provider: new ReCaptchaV3Provider('TWÓJ_RECAPTCHA_SITE_KEY'),
  isTokenAutoRefreshEnabled: true
});
```

### 4. GitHub Pages

1. Wgraj `index.html` i `sw.js` do repozytorium
2. **Settings → Pages → Source: Deploy from branch → main → / (root)**
3. Dodaj domenę GitHub Pages do Firebase Authentication:
   - Firebase Console → Authentication → Settings → Authorized domains → Add domain → `TWOJA_NAZWA.github.io`
4. Włącz App Check enforcement:
   - Firebase Console → App Check → APIs → Realtime Database → **Enforce**

## Struktura plików

```
├── index.html   # Cała aplikacja (single-file PWA)
└── sw.js        # Service Worker (cache offline)
```

## Licencja

Projekt prywatny — do użytku osobistego.
