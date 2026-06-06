# EduVoice AI

Coach vocal intelligent pour les étudiants togolais (BEPC · BAC 1 · BAC 2).  
L'IA transcrit, analyse et conseille chaque prestation orale en temps réel.

---

## Architecture

```
EduVoice/
├── eduvoice-backend/     Django REST API + pipeline IA
└── eduvoice-mobile/      React Native (Expo) — iOS / Android
```

```
Téléphone  ──audio base64──▶  Django  ──▶  Groq Whisper (STT)
                                       ──▶  Groq Llama 3  (analyse)
           ◀──JSON analyse──           ◀──  Fallback auto si erreur
```

### Stack technique

| Couche          | Technologie                                  |
| --------------- | -------------------------------------------- |
| Mobile          | React Native 0.74 · Expo SDK 51 · TypeScript |
| Navigation      | Expo Router v3                               |
| État            | Zustand · AsyncStorage                       |
| Audio           | expo-av · expo-file-system                   |
| Backend         | Django 4.2 · Django REST Framework           |
| Base de données | SQLite (dev) · PostgreSQL (prod)             |
| Cache           | RAM locale (dev) · Redis (prod)              |
| STT             | OpenAI Whisper via Groq                      |
| LLM             | Llama 3.1 8B via Groq                        |

---

## Installation — Backend

### Prérequis

- Python 3.10+ (testé sur 3.14)
- Clé API Groq gratuite → [console.groq.com](https://console.groq.com)

### Étapes

```bash
cd eduvoice-backend

# 1. Installer les dépendances
pip install -r requirements.txt

# 2. Configurer l'environnement
cp .env.example .env
# Ouvrir .env et renseigner GROQ_API_KEY=gsk_...

# 3. Créer la base de données
python manage.py makemigrations topics recordings
python manage.py migrate

# 4. Insérer les données de démo
python manage.py shell < scripts/seed.py

# 5. Lancer le serveur (accessible depuis le réseau local)
python manage.py runserver 0.0.0.0:8000
```

### Variables d'environnement (`eduvoice-backend/.env`)

| Variable       | Description            | Obligatoire                    |
| -------------- | ---------------------- | ------------------------------ |
| `GROQ_API_KEY` | Clé API Groq (gsk_...) | Oui (pour l'IA réelle)         |
| `SECRET_KEY`   | Clé secrète Django     | Non (défaut dev inclus)        |
| `DEBUG`        | `True` en dev          | Non                            |
| `DATABASE_URL` | URL PostgreSQL         | Non (SQLite par défaut)        |
| `REDIS_URL`    | URL Redis              | Non (cache mémoire par défaut) |

### Endpoints API

| Méthode | URL                 | Description                       |
| ------- | ------------------- | --------------------------------- |
| `GET`   | `/api/health/`      | État du serveur et de la clé Groq |
| `GET`   | `/api/topics/`      | Liste des sujets                  |
| `POST`  | `/api/topics/`      | Créer un sujet                    |
| `GET`   | `/api/topics/{id}/` | Détail d'un sujet                 |
| `POST`  | `/api/analyze/`     | Analyser un enregistrement audio  |
| `GET`   | `/api/recordings/`  | Historique des sessions           |
| `GET`   | `/api/docs/`        | Documentation Swagger interactive |

#### Exemple — POST `/api/analyze/`

```json
// Requête
{
  "audio_base64": "<fichier audio encodé en base64>",
  "topic": "La photosynthèse",
  "level": "lycee",
  "duration_seconds": 45
}

// Réponse
{
  "content_accuracy": 78,
  "structure_score": 7,
  "avg_pace_wpm": 124,
  "filler_count": 5,
  "missing_concepts": ["cycle de Calvin", "lumière visible"],
  "advice": [
    "Introduisez votre réponse avec une définition en 1 phrase.",
    "Réduisez les 'euh' en marquant des pauses silencieuses."
  ],
  "next_exercise": "Réenregistrez votre introduction en visant 0 tic sur 20 secondes.",
  "confidence": 0.87,
  "filler_words": [{"word": "euh", "count": 3}, {"word": "donc", "count": 2}],
  "from_fallback": false,
  "processing_time_ms": 2340
}
```

---

## Installation — Mobile

### Prérequis

- Node.js 18+
- Application **Expo Go** sur ton téléphone ([Android](https://play.google.com/store/apps/details?id=host.exp.exponent) · [iOS](https://apps.apple.com/app/expo-go/id982107779))
- Être sur le **même réseau Wi-Fi** que le PC qui fait tourner le backend

### Étapes

```bash
cd eduvoice-mobile

# 1. Installer les dépendances
npm install

# 2. Configurer l'URL du backend
cp .env.example .env.local
# Remplacer localhost par l'IP locale du PC (ex: 192.168.1.73)
# EXPO_PUBLIC_API_URL=http://192.168.1.73:8000

# 3. Lancer
npx expo start
```

Scanne le QR code affiché avec l'app Expo Go.

### Variables d'environnement (`eduvoice-mobile/.env.local`)

| Variable                | Description                      | Exemple                    |
| ----------------------- | -------------------------------- | -------------------------- |
| `EXPO_PUBLIC_API_URL`   | URL du backend Django            | `http://192.168.1.73:8000` |
| `EXPO_PUBLIC_DEMO_MODE` | Forcer le mode démo sans backend | `false`                    |

> **Trouver son IP locale (Windows) :** `ipconfig` → chercher "Adresse IPv4" sous "Wi-Fi"

---

## Fonctionnalités

### Opérationnelles

- **Planning** — liste des sujets à réviser avec deadlines et coefficients
- **Coach Vocal** — enregistrement audio, transcription Whisper, analyse Llama 3
  - Score de précision du contenu (0–100 %)
  - Score de structure (intro / développement / conclusion)
  - Débit mesuré (mots par minute)
  - Détection et comptage des tics de langage
  - 2 conseils actionnables + 1 exercice ciblé
  - Concepts clés non mentionnés
- **Dashboard** — graphiques de progression sur 7 jours, streak, stats globales
- **Mode démo** — fonctionne sans backend (toucher le logo 3× pour activer)
- **Fallback offline** — si le backend ou l'IA est indisponible, une analyse type est renvoyée automatiquement

### En cours / À venir

- Viewer 3D interactif (`@react-three/fiber` — placeholder 2D actuel)
- Module Quiz / EduTrack (génération de QCM après chaque session)
- Export PDF du bilan de session
- Authentification (login / profil étudiant)
- Analyse des matières faibles (algorithme de priorisation)

---

## Niveaux scolaires supportés

| Code            | Niveau        | Examen cible                      |
| --------------- | ------------- | --------------------------------- |
| `lycee`         | Lycée         | BEPC · BAC 1 (Probatoire) · BAC 2 |
| `superieur`     | Université    | Soutenances · exposés             |
| `professionnel` | Formation pro | Présentations métier              |

---

## Lancer les deux en parallèle (workflow développement)

```bash
# Terminal 1 — Backend
cd eduvoice-backend
python manage.py runserver 0.0.0.0:8000

# Terminal 2 — Mobile
cd eduvoice-mobile
npx expo start
```

Vérifier que le backend répond : `http://<IP>:8000/api/health/`

---

## Structure du code — Mobile

```
eduvoice-mobile/
├── app/                    Routes Expo Router
│   ├── _layout.tsx         Navigation tabs + sync backend au démarrage
│   ├── index.tsx           Écran Planning
│   ├── vocal.tsx           Écran Coach Vocal
│   ├── dashboard.tsx       Écran Progression
│   └── viewer.tsx          Écran Concepts
├── components/
│   ├── audio/VocalScreen   Interface enregistrement + résultats IA
│   ├── dashboard/          Graphiques et statistiques
│   ├── ui/ScoreComponents  Jauges circulaires, badges tics, skeletons
│   └── ui/Charts           LineChart et BarChart 100% SVG
├── hooks/useAudioRecorder  Logique d'enregistrement + appel API
├── lib/api.ts              Client HTTP (fetch + timeouts)
├── lib/theme.ts            Design system (couleurs, typographie)
├── store/index.ts          État global Zustand (topics, recordings, progress)
└── types/index.ts          Types TypeScript partagés
```

## Structure du code — Backend

```
eduvoice-backend/
├── config/
│   ├── settings.py         Configuration Django
│   └── urls.py             Routage principal
└── apps/
    ├── topics/             Modèle Topic (CRUD)
    ├── recordings/         Modèle Recording (historique)
    └── analysis/
        ├── pipeline.py     STT Whisper → LLM Llama 3 → fallback
        └── views.py        Endpoint POST /api/analyze/
```
