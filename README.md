# Shelfy

> Gestionnaire de bibliothèque personnelle hors ligne, construit avec Tauri 2 et Rust.

Cataloguez vos livres, suivez vos lectures et gérez vos prêts, le tout dans une application desktop légère qui stocke vos données localement en SQLite — aucun compte, aucun serveur.

## Fonctionnalités

// - Ajout, modification et suppression de livres (titre, auteur, année, ISBN)
// - Statut de lecture (à lire, en cours, lu) et note sur 5
// - Étagères pour organiser sa collection
// - Prêts : à qui, depuis quand, retour
// - Recherche plein texte sur le titre et l'auteur
// - Export et import JSON et CSV
// - Statistiques de lecture (livres lus par année, note moyenne, répartition)

## Installation

// fTéléchargez l'installeur correspondant à votre système sur la [page des releases](https://github.com/ezanka/shelfy/releases) :

| Système | Fichier |
| --- | --- |
| Windows | `Shelfy_x.y.z_x64-setup.exe` ou `.msi` |
| macOS | `Shelfy_x.y.z_x64.dmg` |
| Linux | `shelfy_x.y.z_amd64.AppImage` ou `.deb` |

## Développement

### Prérequis

- [Rust](https://rustup.rs/) stable (toolchain installée via `rustup`)
- [Node.js](https://nodejs.org/) LTS et [pnpm](https://pnpm.io/)
- Dépendances système Tauri selon votre OS : voir le [guide officiel](https://v2.tauri.app/start/prerequisites/) (WebView2 sous Windows, `webkit2gtk` sous Linux, Xcode Command Line Tools sous macOS)

### Démarrer

```bash
git clone git@github.com:ezanka/shelfy.git
cd shelfy
pnpm install
pnpm tauri dev
```

Les migrations de base de données s'appliquent automatiquement au premier lancement.

### Commandes utiles

| Commande | Rôle |
| --- | --- |
| `pnpm tauri dev` | Lance l'application en mode développement |
| `pnpm tauri build` | Construit les installeurs de production |
| `pnpm lint` | Lint du front (ESLint + Prettier) |
| `cargo fmt --check` | Vérifie le formatage Rust |
| `cargo clippy --all-targets -- -D warnings` | Lint Rust strict |
| `cargo test` | Tests Rust (exécuter depuis `src-tauri/`) |
| `cargo sqlx prepare` | Regénère le cache `.sqlx/` pour les requêtes vérifiées à la compilation |

## Architecture

Le front (webview React) ne touche jamais directement au disque ni à la base : il appelle des commandes Tauri, qui délèguent à des services, qui passent par une couche d'accès aux données.

```text
src-tauri/
├── migrations/              # fichiers SQL versionnés
├── capabilities/            # permissions accordées à la webview
└── src/
    ├── main.rs              # point d'entrée
    ├── lib.rs               # Builder Tauri, setup, enregistrement des commandes
    ├── error.rs             # AppError + sérialisation vers le front
    ├── state.rs             # AppState { db: SqlitePool }
    ├── commands/            # commandes exposées au front (fines)
    ├── services/            # logique métier et validation
    └── db/                  # modèles et repositories sqlx
```

Flux d'un appel : `composant React → src/api/*.ts → invoke → commande → service → repository → SQLite`.

## Base de données

Shelfy utilise **SQLite** via [sqlx](https://docs.rs/sqlx/). Le fichier `shelfy.db` est créé dans le dossier de données de l'application :

| Système | Emplacement |
| --- | --- |
| Windows | `%APPDATA%\com.shelfy.app\` |
| macOS | `~/Library/Application Support/com.shelfy.app/` |
| Linux | `~/.local/share/com.shelfy.app/` |

Le schéma est géré par des migrations numérotées dans `src-tauri/migrations/`, appliquées au démarrage. **Une migration déjà commitée n'est jamais modifiée** : toute évolution du schéma passe par une nouvelle migration.

## Stack technique

| Couche | Technologie |
| --- | --- |
| Shell desktop | Tauri 2 |
| Backend | Rust, tokio |
| Base de données | SQLite, sqlx |
| Erreurs | thiserror |
| Sérialisation | serde |
| Frontend | React, TypeScript, Vite |

## Licence

Distribué sous licence MIT. Voir [`LICENSE`](LICENSE).