# Factora API — Postman / Newman E2E (C2 Smoke)

End-to-End-Smoke der **atomic-Invoice-API + KoSIT-Validierung** aus externer
HTTP-Perspektive. Läuft im Pre-Push-Gate (`scripts/precheck.sh`, Stage 3)
headless via Newman.

## Flow (Collection „Factora E2E - C2 Smoke")

1. **1.1** atomic-Rechnung anlegen (maximaler BR-DE-Payload, alle C2-Felder) → `POST {{base_url}}/invoices/atomic/`
2. **1.2** CII/XRechnung-XML holen → `GET {{base_url}}/invoices/{id}/xml/cii/`
3. **1.3** UBL/Peppol-BIS-XML holen → `GET {{base_url}}/invoices/{id}/xml/ubl/`
4. **2.1** CII gegen KoSIT-XRechnung-Validator → `POST {{kosit_url}}/`
5. **2.2** UBL gegen KoSIT-Peppol-Validator → `POST {{kosit_peppol_url}}/`

## Dateien

| Datei | Zweck | Git |
|---|---|---|
| `factora-e2e.postman_collection.json` | Collection (Requests + Tests) | committed |
| `local.postman_environment.template.json` | Environment-Vorlage mit Platzhaltern | committed |
| `local.postman_environment.json` | Echte Werte inkl. Token | **gitignored — NIEMALS committen** |

## Collection importieren (Postman GUI)

1. Postman → **Import** → `factora-e2e.postman_collection.json`.
2. Postman → **Import** → `local.postman_environment.template.json`, als aktives Environment wählen.
3. Platzhalter ersetzen und **als `local.postman_environment.json` exportieren** (Template nicht überschreiben).

## Environment-Variablen

| Variable | Beispiel | Bedeutung |
|---|---|---|
| `base_url` | `http://localhost:8000/api/v1` | API-Basis inkl. `/api/v1` |
| `api_token` | `fa_…` | Bearer-API-Key (Collection-Level-Auth) |
| `kosit_url` | `http://localhost:8081` | KoSIT-XRechnung-Validator (für CII) |
| `kosit_peppol_url` | `http://localhost:8082` | KoSIT-Peppol-Validator (für UBL) |

> Die KoSIT-Container laufen via `docker compose` (`validator` → 8081,
> `validator-peppol` → 8082). „Valid" = Report-Root mit `valid="true"`.
> Echten Token nie ins Repo — nur in `local.postman_environment.json` (gitignored).

## Newman lokal (headless)

```bash
npm install -g newman

newman run docs/api/postman/factora-e2e.postman_collection.json \
  -e docs/api/postman/local.postman_environment.json \
  --bail
```

## Im Pre-Push-Gate (`scripts/precheck.sh`)

Die Newman-Stage ist **Silent-Skip**, wenn `FACTORA_API_TOKEN` nicht gesetzt
**oder** `newman` nicht installiert ist — der Default-Lauf bricht also nicht.
**Vor Production-Deploy MUSS Newman mindestens einmal lokal grün sein.**

```bash
export FACTORA_API_TOKEN=...   # aktiviert Stage 3 (gleicher Wert wie api_token)
bash scripts/precheck.sh
```
