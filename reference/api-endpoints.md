# Endpoints

Alle Pfade relativ zur Basis-URL `https://console.factora.software/api/v1`.
Authentifizierung, Antwort-Hülle und Fehlercodes siehe
[API-Überblick](api-overview.md).

## Schema & interaktive Referenz

Die API ist vollständig per **OpenAPI 3.1** beschrieben (Titel „Factora API",
Version 1.0.0). Bei Abweichungen zwischen Prosa und Schema ist das **Schema
maßgeblich**.

| Ressource | Pfad | Zugriff |
|---|---|---|
| OpenAPI-Schema (3.1) | `GET /api/schema/` | öffentlich |
| Swagger-UI | `GET /api/docs/` | öffentlich |
| ReDoc | `GET /api/redoc/` | öffentlich |
| Scalar-Referenz | `https://console.factora.software/docs` | öffentlich |
| Vollschema | `GET /api/schema/full/` · `GET /api/docs/full/` | authentifiziert |

## Vollständige Endpunktliste

| Methode | Pfad | Zweck | Scope | Sandbox |
|---|---|---|---|---|
| POST | `/invoices/atomic/` | Rechnung in einem Aufruf erstellen, finalisieren, PDF+XML zurückgeben | `invoices:write` | ✓ |
| POST | `/invoices/preview/` | Summen live berechnen (ohne Speicherung) | `invoices:read` | ✓ |
| POST | `/invoices/convert/` | EN-16931-JSON → geprüftes XML (ohne Speicherung) | `invoices:write` | ✓ |
| GET | `/invoices/` | Rechnungen auflisten (paginiert, filterbar) | `invoices:read` | — |
| POST | `/invoices/` | Rechnung als Entwurf anlegen | `invoices:write` | — |
| GET | `/invoices/{id}/` | Einzelne Rechnung abrufen | `invoices:read` | — |
| POST | `/invoices/{id}/finalize/` | Entwurf finalisieren (PDF/XML erzeugen) | `invoices:write` | — |
| POST | `/invoices/{id}/send/` | Finalisierte Rechnung per E-Mail versenden | `invoices:write` | — |
| GET | `/invoices/{id}/pdf/` | ZUGFeRD-PDF herunterladen | `invoices:read` | — |
| GET | `/invoices/{id}/xml/` | XRechnung-XML herunterladen | `invoices:read` | — |
| GET | `/invoices/{id}/xml/cii/` | CII/XRechnung-XML — finalisiert: das archivierte XML (= `/xml/`), Entwurf: aus dem aktuellen Stand erzeugt (B-AP91) | `invoices:read` | — |
| GET | `/invoices/{id}/xml/ubl/` | UBL/Peppol-XML eines Entwurfs; finalisiert → `409` (kein UBL-Artefakt archiviert, kein Re-Rendern; B-AP91) | `invoices:read` | — |
| POST | `/invoices/{id}/dispatch/` | Rechnung zum Transport einreihen (E-Mail/Peppol) | `invoices:write` | — |
| GET | `/invoices/{id}/dispatch-log/` | Versandhistorie abrufen | `invoices:read` | — |
| GET | `/customers/` | Kunden auflisten | `customers:read` | — |
| POST | `/customers/` | Kunden anlegen | `customers:write` | — |
| GET | `/mandanten/` | Mandanten auflisten | `mandanten:read` | — |
| POST | `/mandanten/` | Mandant anlegen | `mandanten:write` | — |
| GET | `/mandanten/{id}/` | Mandant abrufen | `mandanten:read` | — |
| PATCH | `/mandanten/{id}/` | Mandant ändern (partiell) | `mandanten:write` | — |
| DELETE | `/mandanten/{id}/` | Mandant löschen (nur ohne Rechnungen/Kunden) | `mandanten:write` | — |
| POST | `/mandanten/bulk/` | Bis zu 200 Mandanten anlegen (alles-oder-nichts) | `mandanten:write` | — |
| GET | `/exports/datev/` | DATEV-Buchungsdaten als CSV | `exports:read` | — |
| GET | `/webhooks/` | Webhooks auflisten | `webhooks:read` | — |
| POST | `/webhooks/` | Webhook anlegen | `webhooks:write` | — |
| GET | `/webhooks/{id}/` | Webhook abrufen | `webhooks:read` | — |
| PATCH | `/webhooks/{id}/` | Webhook ändern | `webhooks:write` | — |
| DELETE | `/webhooks/{id}/` | Webhook deaktivieren | `webhooks:write` | — |
| POST | `/webhooks/{id}/test/` | Test-Webhook senden | `webhooks:write` | — |
| GET | `/webhooks/{id}/deliveries/` | Zustellungen eines Webhooks auflisten | `webhooks:read` | — |

## Rechnungen

### Atomic Invoice — `POST /invoices/atomic/`

Der zentrale Endpunkt für die ERP-Integration: ein einziger Aufruf erstellt,
finalisiert und prüft die Rechnung und liefert das fertige **ZUGFeRD-PDF**
und die **XRechnung-XML** (beide Base64) zurück. Erfordert im Live-Betrieb
einen Idempotenz-Schlüssel. Das **vollständige Feldschema** steht unter
[Atomic Invoice](atomic-invoice.md).

### Preview — `POST /invoices/preview/`

Berechnet aus übergebenen Positionen die Summen exakt (Dezimal) inklusive
Aufschlüsselung je Steuersatz. Keine Speicherung, auch mit Sandbox-Schlüssel
und bei inaktivem Abo nutzbar.

### Convert — `POST /invoices/convert/`

Rendert EN-16931-Rechnungsdaten ohne Speicherung in geprüftes XML
(CII/XRechnung oder UBL, je nach Rechnungstyp) und gibt KoSIT-Befunde zurück.
Ideal, um die eigene Datenabbildung zu testen.

### Klassischer Lebenszyklus

Statt des Atomic-Wegs lässt sich eine Rechnung auch schrittweise führen:
anlegen (`POST /invoices/`) → finalisieren (`/finalize/`) → herunterladen
(`/pdf/`, `/xml/`) → versenden (`/send/`) oder einreihen (`/dispatch/`).
Listen sind paginiert und filterbar; per `mandant`-Filter lässt sich auf
einen Mandanten einschränken.

## Mandanten

CRUD über `/mandanten/`. Der **Bulk-Endpunkt** legt bis zu 200 Mandanten in
einer Transaktion an (**alles oder nichts** — schlägt eine Zeile fehl, z. B.
durch einen doppelten Namen, wird nichts angelegt; Antwort 409). Ein Mandant
kann nicht gelöscht werden, solange Rechnungen oder Kunden an ihm hängen
(409, GoBD).

## Kunden

Auflisten und Anlegen über `/customers/`. Auf dem Mandanten-Pfad werden
Kunden pro Mandant getrennt geführt.

## Exporte

`GET /exports/datev/` — siehe [Exporte](exports.md).

## Webhooks

CRUD, Testversand und Zustellhistorie über `/webhooks/` — siehe
[Webhooks](webhooks.md).
