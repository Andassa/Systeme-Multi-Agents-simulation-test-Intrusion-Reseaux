# IDS multi-agents (NSL-KDD) — chaîne IDM → GAMA

Système de détection d’intrusions à sept agents. Conception par modèles
(CIM → PIM → PSM), génération GAML, simulation sur **GAMA 2025.6.4**.

Jeu de données : NSL-KDD (125 973 train / 22 544 test).

## Résultat attendu (simulation)

Sur `KDDTest+`, fusion règles + forêt (mode nominal, `taux_panne = 0`) :

| Métrique | Cible |
|---|---|
| Exactitude 5 classes | **≈ 78,9 %** |
| Rappel (attaques) | **≈ 67,7 %** |

Les moniteurs GAMA `decisions` et `journal` doivent progresser ensemble.
Les lignes `[ALERTE]` sont du bruit opérationnel, pas la métrique de validation.

## Démarrage rapide

```bash
pip install pyecore jinja2 numpy pandas lxml

python verifier_tout.py          # chaîne Python / artefacts

cd generator && python generer.py   # régénérer le GAML (zones protégées préservées)
```

Dans GAMA : importer le projet **`gama/`**, ouvrir `models/ids_sma.gaml`,
expérience **`ids_gui`**.

Si un ancien projet `06-gama` est encore ouvert, le fermer et réimporter `gama/`.

## Structure

```
.
├── README.md
├── .gitignore
├── paths.py                 # chemins du dépôt (référence unique)
├── verifier_tout.py         # smoke test de bout en bout
│
├── data/                    # NSL-KDD brut
├── design/                  # CIM, cadrage, diagrammes PlantUML
├── pim/                     # modèle indépendant de plateforme
├── psm/                     # métamodèle Ecore + instances XMI
├── generator/               # PyEcore + Jinja2 → GAML
├── ml/                      # apprentissage & évaluation
│   ├── constants.py         # COLS, classes, labels
│   ├── data_paths.py        # artifacts + localisation NSL-KDD
│   ├── metrics.py           # matrices / exactitude / rappel
│   ├── foret.py / regles.py # modèles
│   ├── preprocessing.py / evaluer_fusion.py / …
│   └── artifacts/           # JSON / NPZ / PKL (sorties)
└── gama/                    # projet GAMA
    └── models/
        ├── ids_sma.gaml     # modèle unique (global + agents + expérience)
        ├── generated/       # encodage, forêt (ne pas éditer)
        └── data/            # copie lue par AgentCapture
└── tests/
    └── test_baseline.py     # garde-fou exactitude ~0.789 + invariants
```

Pas de dossiers numérotés « étape 0…7 » : chaque répertoire correspond à un
rôle dans la chaîne, pas à un tutoriel.

## Ce qui est généré

| Artefact | Statut |
|---|---|
| `gama/models/ids_sma.gaml` | généré — éditer seulement les zones `@user-begin` / `@user-end` |
| `gama/models/generated/*` | généré — ne pas modifier |
| `psm/psm-ids-complet.xmi` | produit par `generator/construire_psm.py` |
| `ml/artifacts/*` | sorties d’apprentissage / évaluation |

## Pipeline agents (rappel)

`Capture` → `Extraction` → (`Règles` ∥ `IA`) → `Décision` → `Alertes` + `Journal`

Le journal est le seul agent qui voit l’étiquette réelle (matrice de confusion).

## Commandes utiles

```bash
# Garde-fou rapide (sans GAMA)
python tests/test_baseline.py

# Référence ML (fusion)
python ml/evaluer_fusion.py

# Oracle = transcription Python du métier GAML
python generator/oracle_simulation.py
```

Le modèle GAMA est un monolithe : sous GAMA 2025, les sous-`model` importés
ne partagent pas le `global` ni les types parent. Éditer seulement les zones
`@user-begin` / `@user-end` ; ne pas relancer `generer.py` sans précaution.

## Notes

- Les fichiers `*.md` hors `README.md` sont ignorés par Git (notes / rapports locaux).
- Ne pas relancer `generer.py` sans comprendre les zones protégées : le code métier
  y est préservé, mais les correctifs hors zones peuvent être écrasés.
- Pour une validation rapide dans GAMA : paramètre `Limite (0=tout)` ≈ `5000`.
