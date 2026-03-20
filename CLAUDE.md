# Schulte Tracker — Contexte projet

## Vue d'ensemble
Application web single-page (un seul `index.html`) de suivi d'entraînement aux tables de Schulte. Hébergée sur GitHub Pages, avec Firebase (Firestore + Google Auth) pour la persistence et l'authentification.

**URL live** : `https://li0re.github.io/schulte-tracker/`
**Repo** : `https://github.com/Li0re/schulte-tracker`

## Stack technique
- **Un seul fichier** `index.html` (~870 lignes) — tout est inline (CSS + JS vanilla)
- **Chart.js 4.4.1** via CDN — graphiques du dashboard
- **Firebase 10.12.0** (compat) — Auth Google + Firestore
- **Pas de framework** — vanilla JS, rendu par innerHTML, state global `S`
- **Fonts** : DM Sans + Space Mono (Google Fonts)
- **Thème** : dark par défaut, toggle clair/sombre (persiste dans localStorage)

## Architecture du code

### State global
```javascript
let S = {
  data: [],        // Array de {date: "2026-03-19", "3x3": [2.4, 3.1, ...], "4x4": [...], ...}
  grids: [],       // Array de {key: "3x3", label: "3×3", defaultAttempts: 5, color: "#22d3ee"}
  view: "dashboard", // "dashboard" | "play" | "entry" | "settings"
  ed: "2026-03-19",  // date sélectionnée dans l'onglet Saisie
  ev: {},            // valeurs des inputs de saisie en cours
  sf: "all",         // filtre grille actif ("all" | "3x3" | "4x4" ...)
  dr: "all",         // filtre date ("all" | "7" | "30" | "90" | "custom")
  drFrom: null,      // date début custom
  drTo: null,        // date fin custom
  calOpen: false,    // calendrier ouvert/fermé
  theme: "dark",     // "dark" | "light"
  objectives: {},    // {"3x3": 2.0, "4x4": 6.0, ...} — objectifs par grille
  // + états UI temporaires (confirmations, formulaires)
}
```

### Fonctions clés
- `R()` — Render principal, reconstruit tout le innerHTML selon `S.view`
- `rD()` — Render dashboard (stats, charts, heatmap)
- `rG()` — Render jeu (table de Schulte interactive)
- `rE()` — Render saisie manuelle
- `rS()` — Render settings (grilles, objectifs, export, reset)
- `iC()` — Initialise les Chart.js après render dashboard
- `fg()` — Retourne les grilles filtrées selon `S.sf`
- `fdData()` — Retourne les données filtrées selon la date range
- `ps()` — Persist (sauvegarde vers Firestore)
- `lev()` — Load entry values (charge les inputs pour la date sélectionnée)

### Firebase
- Config en dur dans le HTML (apiKey, projectId, etc.)
- `auth.onAuthStateChanged` gère login/logout
- `loadFromFirestore()` / `saveToFirestore()` avec debounce 600ms
- Offline persistence activée (`db.enablePersistence()`)
- Structure Firestore : `users/{uid}` → `{data: [...], grids: [...], updatedAt}`
- Règles de sécurité : chaque user ne lit/écrit que ses données

## Les 4 onglets

### 📊 Dashboard
- Barre de filtre sticky (filtre par grille + date range picker avec calendrier)
- Stat cards : moyenne + record par grille, avec % vs veille
- Streak badge (🔥 X jours de suite)
- 4 graphiques Chart.js : moyennes (avec MA7 en pointillé), meilleurs temps, progression %, régularité
- Heatmap de tous les essais

### 🎮 Jouer
- Table de Schulte interactive (3×3 à 10×10)
- Chrono démarre au premier clic correct (sur le 1)
- Les cases NE CHANGENT PAS de couleur après clic correct (volontairement, pour la difficulté)
- Erreur = bref flash rouge
- Résultat → "Enregistrer & Rejouer" ou "Annuler & Rejouer"
- Enregistrement auto dans S.data à la date du jour
- Auto-création de la grille dans la config si elle n'existe pas
- Compteur de parties avec tooltip détaillé par grille
- Inline onclick (pas d'addEventListener) pour zéro délai
- `touch-action: manipulation` pour mobile

### ✏️ Saisie
- Saisie manuelle des temps par date
- Navigation date (veille/aujourd'hui/lendemain)
- Auto-save : debounce 1.5s après frappe + sauvegarde quand on quitte l'onglet ou change de date
- Coloration meilleur (🟢 vert) / pire (🔴 rouge) quand 2+ temps
- +/- pour ajouter/retirer des champs d'essai
- Liste des sessions avec suppression

### ⚙️ Config
- Objectifs par grille (temps cible + barre de progression)
- Gestion des grilles (ajouter, supprimer, modifier essais par défaut)
- Export JSON / CSV
- Réinitialisation avec double confirmation

## Features implémentées
- ✅ Firebase Auth (Google) + Firestore sync
- ✅ Offline persistence
- ✅ Thème clair/sombre (localStorage)
- ✅ Filtre universel par grille
- ✅ Date range picker (All time, 7j, 30j, 90j, custom calendar)
- ✅ Streak (jours consécutifs)
- ✅ Moyenne mobile 7 jours sur graphique
- ✅ Objectifs par grille avec barre de progression
- ✅ Export JSON/CSV
- ✅ Auto-save en saisie
- ✅ Coloration meilleur/pire en saisie
- ✅ Transitions animées entre onglets
- ✅ Table de Schulte jouable (3×3 à 10×10)
- ✅ Header sticky + filtre sticky
- ✅ Onglets animés (slider cyan)
- ✅ Double confirmation pour reset
- ✅ Descriptions sous chaque section
- ✅ Responsive (media query 600px)

## Couleurs des grilles
```javascript
const GC = ["#22d3ee", "#a78bfa", "#fb923c", "#f472b6", "#4ade80", "#facc15", "#38bdf8", "#c084fc"];
// cyan, violet, orange, pink, green, yellow, blue, purple
```

## Conventions
- Code très compact (variables courtes : `S`, `g`, `d`, `gr`, `h`)
- Fonctions courtes : `av()` (average), `bt()` (best/min), `sd()` (std dev), `fd()` (format date), `ti()` (today ISO)
- Tout le state UI est dans `S`, le jeu dans `GM`
- Les grids dynamiques : l'utilisateur peut ajouter/supprimer des tailles
- Les données initiales (ID) sont des données de démo du 2-9 février 2026

## Points d'attention
- Le fichier est un single HTML, pas de build, pas de bundler
- Chaque modification = remplacer index.html sur GitHub → auto-deploy via GitHub Pages
- Firebase config est en clair dans le HTML (c'est normal pour un projet client-side)
- Le calendrier utilise stopPropagation pour ne pas se fermer au clic interne
- Les charts sont détruits et recréés à chaque render (`dC()`)

## Idées futures non implémentées
- Raccourcis clavier pour le jeu
- Mode compétitif / classement
- PWA (manifest + service worker pour install sur mobile)
- Notifications de rappel d'entraînement
- Import de données (JSON)
