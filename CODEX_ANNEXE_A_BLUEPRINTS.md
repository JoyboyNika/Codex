# ANNEXE A — BLUEPRINTS DES 8 SCÉNARIOS CODEX

## Instructions d'utilisation

Cette annexe est appelée par le prompt d'installation CODEX (le corps).
Elle contient les 8 blueprints JSON à passer à Make:scenarios_create.

**Variables à remplacer avant injection :**
- `{{PISTE_KEY_ID}}` → clé PISTE fournie par l'utilisateur (collectée à l'Étape 2)
- `{{ANTHROPIC_CONNECTION_ID}}` → ID numérique de la connexion Anthropic dans Make (détecté à l'Étape 1.3 ou 3)
- `{{TEAM_ID}}` → ID numérique de l'équipe Make (détecté à l'Étape 1.3)
- `{{SYSTEM_PROMPT_CALIBRATEUR}}` → contenu complet de CODEX_ANNEXE_B_CALIBRATEUR.md (uniquement pour le blueprint A.1)

**Paramètres communs à tous les scénarios :**
- scheduling : `{"type": "on-demand"}`
- teamId : `{{TEAM_ID}}`
- confirmed : `true`

**Après chaque création :** appeler Make:scenarios_update avec le nom et la description indiqués.

---

## A.1 — CALIBRATEUR CODEX v3

**Nom :** CODEX — Calibrateur v3
**Description :** Calibre une requête juridique via Haiku. JSON compact: statut, archétype, probes, curseurs, régime, couches, web, transversalité.

**Blueprint :**

```json
{
  "flow": [
    {
      "id": 1,
      "module": "scenario-service:StartSubscenario",
      "version": 2,
      "metadata": {
        "restore": {},
        "designer": {"x": 0, "y": 0},
        "interface": [
          {"name": "message", "type": "text", "required": true, "description": "Requête juridique du professionnel"},
          {"name": "calibration_active", "type": "text", "required": false, "description": "JSON calibration active (vide si première requête)"}
        ]
      }
    },
    {
      "id": 2,
      "mapper": {
        "model": "claude-haiku-4-5-20251001",
        "system": "{{SYSTEM_PROMPT_CALIBRATEUR}}",
        "messages": [
          {
            "role": "user",
            "content": "<message>{{var.input.message}}</message>\n<calibration_active>{{ifempty(var.input.calibration_active; \"null\")}}</calibration_active>",
            "inputType": "single"
          }
        ],
        "max_tokens": 1000,
        "temperature": 0
      },
      "module": "anthropic-claude:createAMessage",
      "version": 1,
      "metadata": {"designer": {"x": 300, "y": 0}},
      "parameters": {"__IMTCONN__": {{ANTHROPIC_CONNECTION_ID}}}
    },
    {
      "id": 3,
      "mapper": {"tool_output": "{{`2`}}"},
      "module": "scenario-service:ReturnData",
      "version": 2,
      "metadata": {
        "expect": [{"name": "tool_output", "type": "dynamicCollection", "label": ""}],
        "restore": {},
        "designer": {"x": 600, "y": 0}
      },
      "parameters": {}
    }
  ],
  "name": "CODEX — Calibrateur v3",
  "metadata": {
    "instant": false,
    "version": 1,
    "designer": {"orphans": []},
    "scenario": {
      "dlq": false,
      "slots": null,
      "dataloss": false,
      "maxErrors": 3,
      "autoCommit": true,
      "roundtrips": 1,
      "sequential": false,
      "confidential": false,
      "freshVariables": false,
      "autoCommitTriggerLast": true
    }
  }
}
```

---

## A.2 — JUDILIBRE — RECHERCHE (Cour de cassation)

**Nom :** Judilibre — Recherche
**Description :** Recherche full-text dans Judilibre (Cour de cassation). Expressions juridiques : utiliser operator=exact. Chambre criminelle = cr (PAS crim).

**Blueprint :**

```json
{
  "flow": [
    {
      "id": 1,
      "module": "scenario-service:StartSubscenario",
      "version": 2,
      "metadata": {
        "restore": {},
        "designer": {"x": 0, "y": 0},
        "interface": [
          {"name": "query", "type": "text", "required": true, "description": "Mots-clés. * pour tout."},
          {"name": "operator", "type": "text", "required": false, "description": "or (défaut), and, exact"},
          {"name": "sort", "type": "text", "required": false, "description": "scorepub (défaut), score, date"},
          {"name": "order", "type": "text", "required": false, "description": "desc (défaut), asc"},
          {"name": "chamber", "type": "text", "required": false, "description": "Code chambre : soc, cr, comm, civ1, civ2, civ3, mi, pl, cream, ordo"},
          {"name": "date_start", "type": "text", "required": false, "description": "Date début ISO-8601 (ex: 2024-01-01)"},
          {"name": "date_end", "type": "text", "required": false, "description": "Date fin ISO-8601"},
          {"name": "solution", "type": "text", "required": false, "description": "rejet, cassation, annulation, etc."},
          {"name": "publication", "type": "text", "required": false, "description": "b=Bulletin, r=Rapport, c=Communiqué, l=Lettre, n=Non publié"},
          {"name": "page_size", "type": "text", "required": false, "description": "1-50, défaut 10"},
          {"name": "page", "type": "text", "required": false, "description": "Page (commence à 0)"}
        ]
      }
    },
    {
      "id": 2,
      "mapper": {
        "url": "https://api.piste.gouv.fr/cassation/judilibre/v1.0/search?query={{var.input.query}}&operator={{ifempty(var.input.operator; \"or\")}}&sort={{ifempty(var.input.sort; \"scorepub\")}}&order={{ifempty(var.input.order; \"desc\")}}&page_size={{ifempty(var.input.page_size; \"10\")}}&resolve_references=true{{if(var.input.chamber; \"&chamber=\" + var.input.chamber; \"\")}}{{if(var.input.date_start; \"&date_start=\" + var.input.date_start; \"\")}}{{if(var.input.date_end; \"&date_end=\" + var.input.date_end; \"\")}}{{if(var.input.solution; \"&solution=\" + var.input.solution; \"\")}}{{if(var.input.publication; \"&publication=\" + var.input.publication; \"\")}}{{if(var.input.page; \"&page=\" + var.input.page; \"\")}}",
        "method": "get",
        "headers": [
          {"name": "KeyId", "value": "{{PISTE_KEY_ID}}"},
          {"name": "accept", "value": "application/json"}
        ],
        "shareCookies": false,
        "parseResponse": true,
        "allowRedirects": true,
        "stopOnHttpError": true,
        "requestCompressedContent": true
      },
      "module": "http:MakeRequest",
      "version": 4,
      "metadata": {"designer": {"x": 300, "y": 0}},
      "parameters": {"authenticationType": "noAuth"}
    },
    {
      "id": 3,
      "mapper": {"tool_output": "{{`2`}}"},
      "module": "scenario-service:ReturnData",
      "version": 2,
      "metadata": {
        "expect": [{"name": "tool_output", "type": "dynamicCollection", "label": ""}],
        "restore": {},
        "designer": {"x": 600, "y": 0}
      },
      "parameters": {}
    }
  ],
  "name": "Judilibre — Recherche",
  "metadata": {
    "instant": false,
    "version": 1,
    "designer": {"orphans": []},
    "scenario": {
      "dlq": false,
      "slots": null,
      "dataloss": false,
      "maxErrors": 3,
      "autoCommit": true,
      "roundtrips": 1,
      "sequential": false,
      "confidential": false,
      "freshVariables": false,
      "autoCommitTriggerLast": true
    }
  }
}
```

---

## A.3 — JUDILIBRE — DÉCISION

**Nom :** Judilibre — Décision
**Description :** Récupère le texte intégral d'une décision Judilibre par son ID. Retourne texte, visa, zones, rapprochements.

**Blueprint :**

```json
{
  "flow": [
    {
      "id": 1,
      "module": "scenario-service:StartSubscenario",
      "version": 2,
      "metadata": {
        "restore": {},
        "designer": {"x": 0, "y": 0},
        "interface": [
          {"name": "decisionId", "type": "text", "required": true, "description": "ID de la décision (obtenu via la recherche)"}
        ]
      }
    },
    {
      "id": 2,
      "mapper": {
        "url": "https://api.piste.gouv.fr/cassation/judilibre/v1.0/decision?id={{var.input.decisionId}}&resolve_references=true",
        "method": "get",
        "headers": [
          {"name": "KeyId", "value": "{{PISTE_KEY_ID}}"},
          {"name": "accept", "value": "application/json"}
        ],
        "shareCookies": false,
        "parseResponse": true,
        "allowRedirects": true,
        "stopOnHttpError": true,
        "requestCompressedContent": true
      },
      "module": "http:MakeRequest",
      "version": 4,
      "metadata": {"designer": {"x": 300, "y": 0}},
      "parameters": {"authenticationType": "noAuth"}
    },
    {
      "id": 3,
      "mapper": {"tool_output": "{{`2`}}"},
      "module": "scenario-service:ReturnData",
      "version": 2,
      "metadata": {
        "expect": [{"name": "tool_output", "type": "dynamicCollection", "label": ""}],
        "restore": {},
        "designer": {"x": 600, "y": 0}
      },
      "parameters": {}
    }
  ],
  "name": "Judilibre — Décision",
  "metadata": {
    "instant": false,
    "version": 1,
    "designer": {"orphans": []},
    "scenario": {
      "dlq": false,
      "slots": null,
      "dataloss": false,
      "maxErrors": 3,
      "autoCommit": true,
      "roundtrips": 1,
      "sequential": false,
      "confidential": false,
      "freshVariables": false,
      "autoCommitTriggerLast": true
    }
  }
}
```

---

## A.4 — JUDILIBRE — TAXONOMIE

**Nom :** Judilibre — Taxonomie
**Description :** Récupère la taxonomie Judilibre (listes de chambres, juridictions, formations, publications, solutions, thèmes).

**Blueprint :**

```json
{
  "flow": [
    {
      "id": 1,
      "module": "scenario-service:StartSubscenario",
      "version": 2,
      "metadata": {
        "restore": {},
        "designer": {"x": 0, "y": 0},
        "interface": [
          {"name": "taxonId", "type": "text", "required": false, "description": "chamber, jurisdiction, formation, publication, solution, theme, type, field"}
        ]
      }
    },
    {
      "id": 2,
      "mapper": {
        "url": "https://api.piste.gouv.fr/cassation/judilibre/v1.0/taxonomy?id={{var.input.taxonId}}",
        "method": "get",
        "headers": [
          {"name": "KeyId", "value": "{{PISTE_KEY_ID}}"},
          {"name": "accept", "value": "application/json"}
        ],
        "shareCookies": false,
        "parseResponse": true,
        "allowRedirects": true,
        "stopOnHttpError": true,
        "requestCompressedContent": true
      },
      "module": "http:MakeRequest",
      "version": 4,
      "metadata": {"designer": {"x": 300, "y": 0}},
      "parameters": {"authenticationType": "noAuth"}
    },
    {
      "id": 3,
      "mapper": {"tool_output": "{{`2`}}"},
      "module": "scenario-service:ReturnData",
      "version": 2,
      "metadata": {
        "expect": [{"name": "tool_output", "type": "dynamicCollection", "label": ""}],
        "restore": {},
        "designer": {"x": 600, "y": 0}
      },
      "parameters": {}
    }
  ],
  "name": "Judilibre — Taxonomie",
  "metadata": {
    "instant": false,
    "version": 1,
    "designer": {"orphans": []},
    "scenario": {
      "dlq": false,
      "slots": null,
      "dataloss": false,
      "maxErrors": 3,
      "autoCommit": true,
      "roundtrips": 1,
      "sequential": false,
      "confidential": false,
      "freshVariables": false,
      "autoCommitTriggerLast": true
    }
  }
}
```

---

## A.5 — JUDILIBRE — VÉRIFICATION POURVOI

**Nom :** Judilibre — Vérification Pourvoi
**Description :** Vérifie l'existence d'une décision par son numéro de pourvoi. Usage principal : protocole CoVe de vérification.

**Blueprint :**

```json
{
  "flow": [
    {
      "id": 1,
      "module": "scenario-service:StartSubscenario",
      "version": 2,
      "metadata": {
        "restore": {},
        "designer": {"x": 0, "y": 0},
        "interface": [
          {"name": "number", "type": "text", "required": true, "description": "Numéro de pourvoi (ex: 21-19.432, 23-16.705)"}
        ]
      }
    },
    {
      "id": 2,
      "mapper": {
        "url": "https://api.piste.gouv.fr/cassation/judilibre/v1.0/search?query={{var.input.number}}&operator=exact&page_size=5&resolve_references=true",
        "method": "get",
        "headers": [
          {"name": "KeyId", "value": "{{PISTE_KEY_ID}}"},
          {"name": "accept", "value": "application/json"}
        ],
        "shareCookies": false,
        "parseResponse": true,
        "allowRedirects": true,
        "stopOnHttpError": true,
        "requestCompressedContent": true
      },
      "module": "http:MakeRequest",
      "version": 4,
      "metadata": {"designer": {"x": 300, "y": 0}},
      "parameters": {"authenticationType": "noAuth"}
    },
    {
      "id": 3,
      "mapper": {"tool_output": "{{`2`}}"},
      "module": "scenario-service:ReturnData",
      "version": 2,
      "metadata": {
        "expect": [{"name": "tool_output", "type": "dynamicCollection", "label": ""}],
        "restore": {},
        "designer": {"x": 600, "y": 0}
      },
      "parameters": {}
    }
  ],
  "name": "Judilibre — Vérification Pourvoi",
  "metadata": {
    "instant": false,
    "version": 1,
    "designer": {"orphans": []},
    "scenario": {
      "dlq": false,
      "slots": null,
      "dataloss": false,
      "maxErrors": 3,
      "autoCommit": true,
      "roundtrips": 1,
      "sequential": false,
      "confidential": false,
      "freshVariables": false,
      "autoCommitTriggerLast": true
    }
  }
}
```

---

## A.6 — JUDILIBRE — RECHERCHE COURS D'APPEL

**Nom :** Judilibre — Recherche Cours d'appel
**Description :** Recherche full-text dans Judilibre limitée aux cours d'appel. Jurisprudence de fond, tendances, positions divergentes.

**Blueprint :**

```json
{
  "flow": [
    {
      "id": 1,
      "module": "scenario-service:StartSubscenario",
      "version": 2,
      "metadata": {
        "restore": {},
        "designer": {"x": 0, "y": 0},
        "interface": [
          {"name": "query", "type": "text", "required": true, "description": "Mots-clés"},
          {"name": "operator", "type": "text", "required": false, "description": "or (défaut), and, exact"},
          {"name": "sort", "type": "text", "required": false, "description": "scorepub (défaut), score, date"},
          {"name": "order", "type": "text", "required": false, "description": "desc (défaut), asc"},
          {"name": "chamber", "type": "text", "required": false, "description": "Code chambre"},
          {"name": "date_start", "type": "text", "required": false, "description": "Date début ISO-8601"},
          {"name": "date_end", "type": "text", "required": false, "description": "Date fin ISO-8601"},
          {"name": "solution", "type": "text", "required": false, "description": "Type de solution"},
          {"name": "page_size", "type": "text", "required": false, "description": "1-50, défaut 10"},
          {"name": "page", "type": "text", "required": false, "description": "Page (commence à 0)"}
        ]
      }
    },
    {
      "id": 2,
      "mapper": {
        "url": "https://api.piste.gouv.fr/cassation/judilibre/v1.0/search?query={{var.input.query}}&jurisdiction=ca&operator={{ifempty(var.input.operator; \"or\")}}&sort={{ifempty(var.input.sort; \"scorepub\")}}&order={{ifempty(var.input.order; \"desc\")}}&page_size={{ifempty(var.input.page_size; \"10\")}}&resolve_references=true{{if(var.input.chamber; \"&chamber=\" + var.input.chamber; \"\")}}{{if(var.input.date_start; \"&date_start=\" + var.input.date_start; \"\")}}{{if(var.input.date_end; \"&date_end=\" + var.input.date_end; \"\")}}{{if(var.input.solution; \"&solution=\" + var.input.solution; \"\")}}{{if(var.input.page; \"&page=\" + var.input.page; \"\")}}",
        "method": "get",
        "headers": [
          {"name": "KeyId", "value": "{{PISTE_KEY_ID}}"},
          {"name": "accept", "value": "application/json"}
        ],
        "shareCookies": false,
        "parseResponse": true,
        "allowRedirects": true,
        "stopOnHttpError": true,
        "requestCompressedContent": true
      },
      "module": "http:MakeRequest",
      "version": 4,
      "metadata": {"designer": {"x": 300, "y": 0}},
      "parameters": {"authenticationType": "noAuth"}
    },
    {
      "id": 3,
      "mapper": {"tool_output": "{{`2`}}"},
      "module": "scenario-service:ReturnData",
      "version": 2,
      "metadata": {
        "expect": [{"name": "tool_output", "type": "dynamicCollection", "label": ""}],
        "restore": {},
        "designer": {"x": 600, "y": 0}
      },
      "parameters": {}
    }
  ],
  "name": "Judilibre — Recherche Cours d'appel",
  "metadata": {
    "instant": false,
    "version": 1,
    "designer": {"orphans": []},
    "scenario": {
      "dlq": false,
      "slots": null,
      "dataloss": false,
      "maxErrors": 3,
      "autoCommit": true,
      "roundtrips": 1,
      "sequential": false,
      "confidential": false,
      "freshVariables": false,
      "autoCommitTriggerLast": true
    }
  }
}
```

---

## A.7 — JUDILIBRE — RECHERCHE TRIBUNAUX JUDICIAIRES

**Nom :** Judilibre — Recherche Tribunaux judiciaires
**Description :** Recherche full-text dans Judilibre limitée aux tribunaux judiciaires. Jurisprudence de première instance, application locale du droit.

**Blueprint :** Identique à A.6 avec les modifications suivantes :
- Dans l'URL du mapper : remplacer `jurisdiction=ca` par `jurisdiction=tj`
- Dans blueprint.name : `"Judilibre — Recherche Tribunaux judiciaires"`

---

## A.8 — JUDILIBRE — RECHERCHE TRIBUNAUX DE COMMERCE

**Nom :** Judilibre — Recherche Tribunaux de commerce
**Description :** Recherche full-text dans Judilibre limitée aux tribunaux de commerce. Contentieux commercial, procédures collectives, droit des affaires.

**Blueprint :** Identique à A.6 avec les modifications suivantes :
- Dans l'URL du mapper : remplacer `jurisdiction=ca` par `jurisdiction=tcom`
- Dans blueprint.name : `"Judilibre — Recherche Tribunaux de commerce"`
