# CODEX Legal Engine

**Moteur d'analyse juridique calibré, vérifiable et connecté aux flux officiels.**

CODEX n'est pas un chatbot juridique. C'est un système de recherche et d'analyse conçu pour des professionnels du droit — avocats, juristes, universitaires — qui traite chaque requête comme un acte technique : calibration automatique, étiquetage épistémique de chaque assertion, vérification active des sources sur les bases officielles.

Le principe fondateur : **l'instrument, pas le consultant.** CODEX produit du matériau vérifié, tracé et étiqueté que le professionnel audite et intègre à son propre raisonnement. Il ne remplace jamais le jugement humain. Il le nourrit.

> 📖 **Documentation complète** : [Notion — CODEX Engine](LIEN_NOTION_À_REMPLACER)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    PROFESSIONNEL DU DROIT                    │
│                     (requête juridique)                      │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                     CLAUDE (moteur CODEX)                     │
│  System prompt noyau + 4 annexes (B, C, D, E)                │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ 1. Calibration    → appel MCP au calibrateur Haiku     │  │
│  │ 2. Couches        → sélection conditionnelle (max 3)   │  │
│  │ 3. Analyse        → étiquetage épistémique 5 niveaux   │  │
│  │ 4. Vérification   → protocole CoVe anti-hallucination  │  │
│  └────────────────────────────────────────────────────────┘  │
│                           │                                  │
│              ┌────────────┼────────────┐                     │
│              ▼            ▼            ▼                      │
│         Calibrateur    Judilibre    Recherche                 │
│          (Haiku)      (7 outils)     web                     │
└──────────────────────────────────────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
┌──────────────────────┐  ┌──────────────────────────────────┐
│    Make (MCP)         │  │   API Judilibre (PISTE)          │
│  Orchestration des    │  │   Cour de cassation, CA, TJ,    │
│  scénarios            │  │   tribunaux de commerce          │
└──────────────────────┘  └──────────────────────────────────┘
```

**Boucle en 4 temps à chaque requête :**

1. **Calibration externalisée** — Un calibrateur dédié (Haiku via MCP/Make) analyse la requête et retourne un JSON structuré : archétype, régime épistémique, curseurs rigueur/périmètre/profondeur, couches à activer, branches du droit connexes.
2. **Sélection des couches** — Le moteur active jusqu'à 3 couches conditionnelles parmi 5 (vérification citations, multi-branches, conformité normative, diachronique, cohérence). La recherche web reste permanente.
3. **Analyse étiquetée** — Chaque assertion reçoit une étiquette : `[ÉTABLI]`, `[POSITION MAJORITAIRE]`, `[CONNEXION ARGUMENTÉE]`, `[DÉBATTU]`, `[PROSPECTIF]`.
4. **Vérification active (CoVe)** — Interrogation directe de Judilibre pour confirmer chaque citation avant intégration. Pas de citation de mémoire non vérifiée.

---

## Contenu du dépôt

### Prompts (Projet Claude)

| Fichier | Rôle |
|---|---|
| `Codex Engine/CODEX_ENGINE_SYSTEM_PROMPT.md` | **Noyau** — System prompt principal du moteur |
| `Codex Engine/ANNEXE_B_Strategies_analyse.md` | 6 couches d'analyse conditionnelles |
| `Codex Engine/ANNEXE_C_Sources_Citations.md` | Hiérarchie des sources, formats de citation, protocole CoVe |
| `Codex Engine/ANNEXE_D_Etiquetage_juridique.md` | 5 niveaux d'étiquetage épistémique |
| `Codex Engine/ANNEXE_E_Posture.md` | 7 règles de posture, anti-patterns |

### Installation Make

| Fichier | Rôle |
|---|---|
| `Codex Claude Installateur/INSTALLATEUR_CODEX.md` | Prompt d'installation guidée (Claude installe lui-même les scénarios Make) |
| `Codex Claude Installateur/CODEX_ANNEXE_A_BLUEPRINTS.md` | 8 blueprints JSON pour Make (calibrateur + 7 outils Judilibre) |
| `Codex Claude Installateur/CODEX_ANNEXE_B_CALIBRATEUR.md` | System prompt du calibrateur Haiku |

---

## Prérequis

| Service | Coût | Rôle |
|---|---|---|
| **Claude Pro** | ~18 €/mois | Moteur d'analyse (projets + MCP) |
| **Make** (plan Core) | ~9 €/mois | Orchestration des scénarios MCP |
| **API Anthropic** | ~10 € de crédit (dure plusieurs mois) | Calibrateur Haiku |
| **PISTE** (API Judilibre) | Gratuit | Accès à la jurisprudence officielle |

**Coût total : environ 28 €/mois.** CODEX lui-même est 100 % gratuit.

---

## Installation

L'installation est **entièrement guidée par Claude**. Vous n'avez pas besoin de toucher à du code ni de configurer Make manuellement.

### Étape 1 — Créer les comptes

- Un compte [Make](https://www.make.com/en/register) (plan Core minimum — 8 scénarios nécessaires)
- Un compte [PISTE](https://piste.gouv.fr) avec l'API Judilibre activée
- Un compte [API Anthropic](https://console.anthropic.com) avec 10 € de crédit

### Étape 2 — Connecter Make à Claude

Dans Claude → icône connecteurs → chercher "Make" → connecter → autoriser.

### Étape 3 — Lancer l'installation guidée

1. Créer un nouveau **Projet** dans Claude
2. Charger les 3 fichiers du dossier `installation/` dans les fichiers du Projet
3. Coller le contenu de `installation/INSTALLATEUR_CODEX.md` dans les **Instructions** du Projet
4. Ouvrir une conversation dans ce Projet
5. Suivre les instructions — Claude détecte votre environnement, crée les 8 scénarios Make et configure tout

### Étape 4 — Créer le Projet CODEX

Une fois l'installation terminée :

1. Créer un **nouveau Projet** dans Claude nommé "CODEX Legal Engine"
2. Coller le contenu de `prompts/CODEX_ENGINE_SYSTEM_PROMPT.md` dans les **Instructions**
3. Charger les 4 annexes (`ANNEXE_B` à `ANNEXE_E`) dans les **fichiers** du Projet
4. Ouvrir une conversation — CODEX est opérationnel

---

## Principes de conception

**Calibration externalisée sur modèle léger.** Le calibrateur tourne sur Haiku, pas sur le modèle principal. Rapide, peu coûteux, isolé. Le moteur se concentre sur l'analyse.

**Curseurs plutôt que modes.** Rigueur, périmètre et profondeur sont des curseurs indépendants. Une requête peut être à haute rigueur mais faible périmètre. Un mode "expert" unique ne capturerait pas cette granularité.

**Étiquetage épistémique obligatoire.** 5 niveaux, pas 3. La distinction entre `[POSITION MAJORITAIRE]` et `[ÉTABLI]` est cruciale : la première admet des exceptions, la seconde non.

**Matrice de transversalité à scores.** 17 domaines × 17 domaines, scores de 0 à 3. Le calibrateur dose la transversalité selon la pertinence réelle plutôt qu'une liste binaire.

**Judilibre comme source primaire.** Connexion directe à l'API de la Cour de cassation. Le moteur ne cite pas de mémoire : il interroge, vérifie, confirme.

**Anti-patterns nommés.** 7 comportements interdits avec leur correction : l'oracle, le flatteur, le professeur, le bavard, le prudent excessif, le devin, le miroir.

---

## Limites connues

- **Pas d'accès direct à Légifrance.** Vérification des textes de loi via recherche web.
- **Périmètre temporel de Judilibre.** Certaines décisions anciennes ne sont pas dans la base.
- **Ordre administratif non couvert par Judilibre.** CE, CAA et TA vérifiés par web uniquement.
- **Knowledge cutoff du modèle.** Les réformes très récentes peuvent échapper à la connaissance interne. Judilibre et le web compensent.

---

## Licence

[MIT](LICENSE) — Utilisez, modifiez, distribuez librement.

---

## Auteur

**Tudual Lucas Huon** — Juriste · Prompt engineer

- [LinkedIn](https://www.linkedin.com/in/tudual-huon-700491222)

CODEX est un projet indépendant, sans lien commercial avec Anthropic, Make, ni aucun autre acteur. Il est né d'une conviction : les avancées en droit et en intelligence artificielle doivent être accessibles à tous.

---

*CODEX ENGINE — L'IA au service du professionnel, à côté de lui, et jamais sans lui.*
