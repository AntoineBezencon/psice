# Paprika *(nom provisoire)*

Simulateur de circuits électroniques inspiré de LTspice, avec une interface moderne
et ngspice comme moteur de simulation.

Projet de groupe de 2ᵉ année — HEIG-VD.

> ⚠️ Projet en cours de démarrage : ce README décrit les objectifs et l'architecture prévue.

---

## Objectifs

- Dessiner un schéma électronique dans un éditeur graphique (composants, fils, masse).
- Simuler le circuit avec **ngspice** : point de fonctionnement (`.op`), temporel (`.tran`),
  fréquentiel (`.ac`), balayage DC (`.dc`).
- Afficher les résultats avec une **sonde interactive** : cliquer sur un fil affiche sa tension,
  cliquer sur un composant affiche son courant.
- Proposer un **solveur maison** (analyse nodale modifiée, circuits linéaires en DC et en temporel)
  sélectionnable dans l'interface, pour comparer ses résultats à ceux de ngspice.
- Livrer une **application Windows** lançable d'un double-clic, sans installation.

### Fonctionnalités prévues

| Priorité | Fonctionnalité |
|---|---|
| MVP | Éditeur de schéma : placement, rotation, fils, grille, zoom |
| MVP | Composants de base : R, C, L, sources de tension et de courant (DC, PULSE, SIN), masse |
| MVP | Génération de netlist SPICE et simulation via ngspice |
| MVP | Visualiseur de courbes avec sonde interactive |
| MVP | Sauvegarde et ouverture des schémas |
| Ensuite | Diodes, transistors (BJT, MOSFET), modèles `.model` / `.subckt` |
| Ensuite | Analyses AC (diagramme de Bode) et DC, balayage de paramètres |
| Ensuite | Annuler / rétablir, copier-coller |
| Ensuite | Solveur maison en mode pédagogique |
| Bonus | Import des schémas LTspice (`.asc`) |
| Bonus | Mesures sur les courbes (crête, moyenne, temps de montée), FFT, export |

---

## Architecture

```
┌──────────────────────────── Fenêtre pywebview ─────────────────────────────┐
│  Frontend React + TypeScript                                                │
│  ┌──────────────────────┐   ┌────────────────────────┐                      │
│  │ Éditeur de schéma    │   │ Courbes + sondes       │                      │
│  │ (Konva / SVG)        │   │ (uPlot)                │                      │
│  └──────────┬───────────┘   └───────────▲────────────┘                      │
└─────────────┼───────────────────────────┼───────────────────────────────────┘
              │  schéma (JSON)            │  résultats (JSON)
              ▼                           │
┌─────────────────────────── Backend Python (FastAPI) ────────────────────────┐
│  netlist/     schéma JSON → netlist SPICE                                    │
│  importers/   import .asc LTspice                                            │
│  engines/     ngspice  │  solveur maison (MNA)                               │
└──────────────────────────────────────────────────────────────────────────────┘
```

- Le **frontend** ne fait que l'affichage et l'édition. Toute la logique de simulation est dans le backend.
- Le **contrat d'API** (format du schéma et des résultats) est défini en Python avec Pydantic.
  Les types TypeScript du front sont générés à partir du schéma OpenAPI de FastAPI
  (`openapi-typescript`), pour que les deux côtés restent synchronisés.
- Tout tourne **en local** : aucun hébergement nécessaire.

### Stack technique

| Partie | Technologies |
|---|---|
| Frontend | React, TypeScript, Vite, Konva (ou SVG), uPlot, Zustand |
| Backend | Python 3, FastAPI, Pydantic, NumPy / SciPy |
| Simulation | ngspice (binaires Windows inclus dans le projet) |
| Application | pywebview (fenêtre native), PyInstaller (packaging) |
| Qualité | pytest, ESLint, GitHub Actions |

---

## Structure du repo

```
paprika/
├── backend/
│   ├── app/
│   │   ├── api/          routes FastAPI
│   │   ├── models/       modèles Pydantic (contrat d'API)
│   │   ├── netlist/      génération de netlist SPICE
│   │   ├── importers/    import .asc LTspice
│   │   └── engines/      ngspice + solveur maison
│   ├── vendor/ngspice/   binaires ngspice Windows
│   ├── launcher.py       démarre le serveur et ouvre la fenêtre
│   ├── tests/
│   └── pyproject.toml
├── frontend/
│   ├── src/
│   │   ├── editor/       éditeur de schéma
│   │   ├── plot/         courbes et sondes
│   │   ├── store/        état de l'application, annuler/rétablir
│   │   └── api/          client vers le backend
│   └── package.json
├── build.ps1             build complet → application Windows
└── README.md
```

---

## Développement

### Prérequis

- Windows 10 ou 11
- Python 3.12+
- Node.js 20+
- ngspice placé dans `backend/vendor/ngspice/` *(procédure à compléter)*

### Lancer en mode développement

```powershell
# Terminal 1 — backend
cd backend
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -e ".[dev]"
uvicorn app.main:app --reload

# Terminal 2 — frontend
cd frontend
npm install
npm run dev
```

Puis ouvrir http://localhost:5173.

### Tests

```powershell
cd backend
pytest
```

Les tests de simulation comparent les résultats du solveur maison à ceux de ngspice
sur une série de circuits de référence (diviseur de tension, RC, RLC, redresseur…).

---

## Build de l'application Windows

```powershell
.\build.ps1
```

Le script :
1. compile le frontend (`npm run build`) ;
2. copie le résultat dans le backend ;
3. lance PyInstaller avec ngspice et le front inclus.

Résultat : `dist/Paprika/Paprika.exe`, à distribuer sous forme de dossier zippé.
Aucune installation n'est nécessaire sur la machine cible.

> Au premier lancement, Windows SmartScreen peut afficher un avertissement car l'exécutable
> n'est pas signé : cliquer sur « Informations complémentaires » puis « Exécuter quand même ».

---

## Organisation

4 personnes, environ 90 h chacune.

| Rôle | Responsabilité | Personne |
|---|---|---|
| Éditeur de schéma | Canvas, placement, fils, sélection, zoom, annuler/rétablir | *à définir* |
| Courbes et sondes | Visualiseur, sonde interactive, curseurs, mesures | *à définir* |
| Bibliothèque et netlist | Composants, modèles SPICE, génération de netlist, import `.asc`, API | *à définir* |
| Simulation | Intégration ngspice, analyses, solveur maison, packaging | *à définir* |

### Jalons

| Semaines | Objectif |
|---|---|
| S1–2 | Repo, CI, contrat d'API, ngspice validé sur toutes les machines, prototype netlist → courbe |
| S3–5 | Éditeur basique de bout en bout : dessiner un circuit RC et le simuler |
| S6–8 | Analyses AC et DC, sonde interactive, diodes et transistors, sauvegarde |
| S9–10 | Solveur maison, import `.asc`, annuler/rétablir, mesures |
| S11–12 | Tests, stabilisation, build final, rapport et démo |

---

## Licence

*À définir.* Attention : ngspice est distribué sous licence BSD modifiée ; vérifier la compatibilité
avant de choisir la licence du projet.