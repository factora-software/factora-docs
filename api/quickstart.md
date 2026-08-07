# Schnellstart — erster API-Call in 5 Minuten

Diese Anleitung bringt Sie vom leeren Account zum ersten authentifizierten
Aufruf gegen die Factora Public API v1: Schlüssel erstellen, mit einem
einfachen `GET` prüfen, Antwortstruktur verstehen. Für das Erzeugen einer
echten E-Rechnung geht es danach in den verlinkten Dokumenten weiter.

## Voraussetzungen

- Ein Factora-Account im **API-Tarif**. Registrierung über die Console:
  `https://console.factora.software/auth/register`. Es gibt **keinen
  befristeten Trial** — **Sandbox-Schlüssel sind dauerhaft kostenlos**, der
  Live-Betrieb wird durch Hinterlegen einer Zahlungsmethode in der Console
  freigeschaltet.
- Bestätigte E-Mail-Adresse und ein eingeloggter Account.
- Ein HTTP-Client Ihrer Wahl — die Beispiele nutzen `curl`.

> **Basis-URL aller Endpunkte:** `https://console.factora.software/api/v1/`

Zu Preisen und Mengenstaffel siehe [FAQ](faq.md#was-kostet-die-api).

## 1. API-Schlüssel erstellen

Schlüssel erstellen Sie in der Console unter
`https://console.factora.software/dashboard/api-keys`.

- Der Schlüssel beginnt je nach Umgebung mit **`fa_live_`** (Live) bzw.
  **`fa_test_`** (Sandbox). Ältere Schlüssel (`fa_…`) authentifizieren
  weiterhin.
- Beim Erstellen wählen Sie die Umgebung. Sandbox-Schlüssel sind kostenlos und
  führen `POST /invoices/atomic/` als Trockenlauf aus (volle Validierung,
  PDF + XML als Vorschau, nichts wird gespeichert) — siehe
  [Sandbox & Testing](../reference/sandbox.md).
- Der Klartext wird **genau einmal** angezeigt. Serverseitig liegt kein
  Klartext, nur ein SHA-256-Hash und ein HMAC. Geht der Schlüssel verloren,
  gibt es kein Recovery — neuen erstellen, alten deaktivieren.
- Optional pro Schlüssel: **IP-Whitelist**, **Scopes**, **Ablaufdatum** —
  Details unter [API-Überblick](../reference/api-overview.md#api-schlüssel).

Das Tageslimit des API-Tarifs beträgt **50.000 Aufrufe / 24 h** je Schlüssel
bei **bis zu 20 Schlüsseln** je Konto. Es ist ein Anti-Missbrauchs-Schutz,
keine Preisstufe.

## 2. Authentifizierung

Jeder Aufruf trägt den Schlüssel im `Authorization`-Header als Bearer-Token:

```
Authorization: Bearer fa_live_xxx…
```

## 3. Erster Aufruf — Schlüssel verifizieren

Der einfachste Aufruf ohne Rumpf ist das Auflisten der Rechnungen. Antwortet
er mit `200`, funktioniert der Schlüssel:

```bash
curl https://console.factora.software/api/v1/invoices/ \
  -H 'Authorization: Bearer fa_live_xxx…'
```

## 4. Antwortstruktur verstehen

Alle Antworten folgen derselben Hülle:

```json
{ "valid": true, "data": { }, "errors": [ ], "meta": { } }
```

- **`valid`** — `true` bei Erfolg, `false` im Fehlerfall.
- **`data`** — das Ergebnis (bei Listen ein Array, bei Detail-Aufrufen ein
  Objekt), `null` im Fehlerfall.
- **`errors`** — bei Erfolg leer; im Fehlerfall Einträge mit `code`,
  `severity`, `message` und je nach Fehlerart `field`, `rule`, `bt`,
  `location`.
- **`meta`** — Zusatzinfos (Paginierung, Quota, `sandbox`-Flag).

Damit ist der erste erfolgreiche API-Aufruf geschafft.

## Eine vollständige Rechnung in einem Aufruf (atomic)

`POST /invoices/atomic/` legt eine komplette Rechnung an und liefert PDF und
XML als Base64 zurück. Der Rumpf folgt dem EN-16931-Feldkatalog — siehe
[Feld-Referenz](en16931-field-reference.md) und
[Atomic Invoice](../reference/atomic-invoice.md).

```bash
curl -X POST https://console.factora.software/api/v1/invoices/atomic/ \
  -H 'Authorization: Bearer fa_live_xxx…' \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: 9f4b7d92-3e4a-4b1d-9c6f-1234567890ab' \
  -d @invoice.json
```

Der Header **`Idempotency-Key`** (UUID, Gültigkeit 48 h) verhindert
Duplikat-Rechnungen: Bei identischem Schlüssel wird die ursprüngliche Antwort
zurückgegeben — sicher für Wiederholungen.

### Rechnung finalisieren

```bash
curl -X POST https://console.factora.software/api/v1/invoices/42/finalize/ \
  -H 'Authorization: Bearer fa_live_xxx…'
```

### ZUGFeRD-PDF herunterladen

```bash
curl https://console.factora.software/api/v1/invoices/42/pdf/ \
  -H 'Authorization: Bearer fa_live_xxx…' \
  -o invoice-42.pdf
```

## Wie es weitergeht

Die vollständige Endpunktliste mit Scopes und Sandbox-Eignung steht unter
[Endpoints](../reference/api-endpoints.md).

## Häufige Stolpersteine

- **401 Unauthorized** — Header fehlt oder Schlüssel ist falsch, inaktiv oder
  abgelaufen. Exaktes Format prüfen: `Authorization: Bearer fa_…`.
- **403 Forbidden** — Tarif ohne API-Zugang, fehlender Scope, aufrufende IP
  nicht auf der Whitelist, oder ein Sandbox-Schlüssel auf einem nicht
  zugelassenen Endpunkt.
- **429 Too Many Requests** — Tageskontingent des Schlüssels erreicht. Die
  Antwort trägt einen `Retry-After`-Header; siehe
  [Limits](../reference/api-overview.md#limits-throttling).
- **Schlüssel verloren?** Kein Recovery — in der Console einen neuen erstellen
  und den alten deaktivieren.
- **Live noch nicht freigeschaltet?** Sandbox-Schlüssel funktionieren sofort;
  für Live-Schlüssel zuerst in der Console unter *Abrechnung* eine
  Zahlungsmethode hinterlegen.

Vollständige Status- und Fehlercodes:
[API-Überblick](../reference/api-overview.md#fehler--und-antwortstruktur).
