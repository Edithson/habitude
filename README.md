# HabitFlow v3.0 — Suivi des Habitudes avec Persistance SQLite (100% Frontend)

HabitFlow est une application moderne de suivi des habitudes et de renforcement de la discipline, fonctionnant **entièrement dans le navigateur sans nécessiter de serveur**, tout en utilisant une authentique base de données relationnelle **SQLite 3** (`habits.db`) stockée directement sur votre disque physique.

---

## 🚀 Comment ça fonctionne ?

1. **Moteur SQLite WebAssembly (`sql.js`)** :
   Le moteur C officiel de SQLite est exécuté directement dans votre navigateur via WebAssembly. Les performances sont quasi-instantanées et conformes au standard ACID.
   
2. **Accès direct au disque (File System Access API)** :
   Grâce aux APIs natives du navigateur (`window.showOpenFilePicker` / `window.showSaveFilePicker`), l'application lit et écrit directement dans votre fichier `habits.db` sur votre disque dur.

3. **Reconnexion transparente (IndexedDB)** :
   La poignée du fichier physique ainsi qu'une copie miroir en mémoire sont persistées localement dans IndexedDB. Vos données restent accessibles instantanément même après un rechargement de page.

---

## 🗄️ Schéma relationnel de la base SQLite

Votre fichier `habits.db` est un **vrai fichier SQLite 3**. Vous pouvez l'ouvrir à tout moment avec `sqlite3 habits.db`, avec *DB Browser for SQLite*, *DBeaver* ou une extension VS Code.

```sql
-- 1. Catégories d'habitudes
CREATE TABLE categories (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  color TEXT NOT NULL,
  icon TEXT NOT NULL,
  position INTEGER DEFAULT 0
);

-- 2. Habitudes (Action positive ou Abstinence)
CREATE TABLE habits (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  type TEXT NOT NULL CHECK(type IN ('action', 'abstinence')),
  category_id TEXT REFERENCES categories(id) ON DELETE SET NULL,
  target INTEGER NOT NULL DEFAULT 7,
  color TEXT NOT NULL,
  icon TEXT NOT NULL,
  created_at TEXT NOT NULL,
  archived INTEGER NOT NULL DEFAULT 0
);

-- 3. Logs quotidiens d'accomplissement
CREATE TABLE logs (
  habit_id TEXT NOT NULL REFERENCES habits(id) ON DELETE CASCADE,
  date TEXT NOT NULL, -- Format 'YYYY-MM-DD'
  completed INTEGER NOT NULL CHECK(completed IN (0, 1)),
  PRIMARY KEY (habit_id, date)
);

-- 4. Paramètres de l'application
CREATE TABLE settings (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL
);
```

---

## 🛠️ Fonctionnalités intégrées

- **Liaison directe au fichier `habits.db`** : cliquez sur *Lier habits.db* dans la barre de navigation pour lier le fichier physique de votre projet.
- **Création d'une nouvelle base** : bouton *Nouveau .db* pour créer et initialiser une nouvelle base vierge sur votre disque.
- **Migration automatique depuis JSON** : bouton *Migrer JSON* pour importer à tout moment un ancien fichier `habitflow-data.json`.
- **Console & Explorateur SQLite** :
  - **Tables & Données** : inspectez le contenu brut des tables `habits`, `logs`, `categories` et `settings`.
  - **Console SQL** : exécutez n'importe quelle requête SQL personnalisée en direct (`Ctrl + Entrée`).
  - **Dump SQL** : exportez la structure et les données au format script `.sql`.
- **Téléchargement direct (.db)** : bouton *Exporter .db* pour sauvegarder une copie binaire à la volée.

---

## 📂 Arborescence du projet

```
habitude/
├── index.html                   # Application web complète (v3.0 SQLite)
├── habits.db                    # Base de données SQLite 3 physique active
├── README.md                    # Documentation complète du projet
├── assets/                      # Ressources locales & autonomes
│   ├── css/
│   │   ├── fontawesome.min.css  # Icônes FontAwesome
│   │   └── fonts.css            # Polices Inter & Plus Jakarta Sans
│   └── js/
│       ├── tailwind.min.js      # Framework CSS Tailwind
│       ├── chart.min.js         # Moteur de graphiques Chart.js
│       ├── sql-wasm.js          # Moteur WebAssembly sql.js
│       ├── sql-wasm.wasm        # Binaire compilé SQLite
│       └── sql-wasm-data.js     # Chargeur WASM autonome (zéro fetch, zéro restriction CORS file://)
└── backups/                     # Archives et sauvegardes
    ├── habitflow-data.json      # Données JSON d'origine
    └── index.json.backup.html   # Ancienne version de l'application (v2.0 JSON)
```
