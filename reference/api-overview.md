# API-Überblick

Die öffentliche **Factora API v1** ist eine REST-Schnittstelle zum Erzeugen,
Prüfen, Abrufen und Versenden elektronischer Rechnungen aus einem Fremdsystem
heraus.

- **Basis-URL:** `https://console.factora.software/api/v1`
- **Datenformat:** JSON (UTF-8)
- **Spezifikation:** OpenAPI 3.1 — interaktiv unter
  `https://console.factora.software/api/docs/`

## Authentifizierung

Jeder Aufruf wird mit einem **API-Schlüssel** als Bearer-Token autorisiert:

```
Authorization: Bearer fa_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### API-Schlüssel

- **Live-Schlüssel** beginnen mit `fa_live_`, **Sandbox-Schlüssel** mit
  `fa_test_`. Ältere Schlüssel ohne Umgebungsmarker (`fa_…`) authentifizieren
  weiterhin unverändert.
- Die ersten 14 Zeichen (Präfix + 6 Zeichen) bilden den **Key-Präfix**, der in
  Listen und Logs der Console sichtbar ist und den Schlüssel identifiziert,
  ohne das Geheimnis preiszugeben.
- Der Klartext wird **nur einmal** bei der Erstellung angezeigt. Serverseitig
  liegt kein Klartext: gespeichert werden ein **SHA-256-Hash** und ein
  **HMAC-SHA256**. Geht ein Schlüssel verloren, gibt es kein Recovery — neuen
  erzeugen, alten deaktivieren.
- Optional pro Schlüssel: **Ablaufdatum**, **IP-Whitelist** und **Scopes**.
- **Ablaufdatum** ist optional; ohne Wert läuft der Schlüssel nie ab. Nach
  Ablauf antwortet die API mit `401`.
- **Rotation:** In der Console lässt sich ein Schlüssel gegen einen neuen
  ersetzen. Der alte wird deaktiviert, bleibt aber für die Audit-Historie
  erhalten.
- Verwaltung in der Console unter
  `https://console.factora.software/dashboard/api-keys`.

### IP-Whitelist

Pro Schlüssel lässt sich eine Liste erlaubter IP-Adressen hinterlegen. Eine
**leere Liste = alle IPs erlaubt**. Ist die Liste gefüllt, muss der Request von
einer **exakt gelisteten Einzel-IP** kommen — **CIDR-Bereiche werden bewusst
abgelehnt**, weil die Prüfung serverseitig auf exakte Übereinstimmung geht.
Maßgeblich ist die vom Reverse-Proxy gesetzte Client-IP.

### Scopes (Berechtigungen)

Ein Schlüssel kann auf bestimmte Rechte beschränkt werden. Ohne Einschränkung
(leere Liste) gelten alle Rechte. Die Prüfung ist **methodenbasiert**: lesende
Methoden (`GET`) verlangen den `…:read`-Scope, schreibende (`POST` / `PATCH` /
`DELETE`) den `…:write`-Scope.

| Scope | Bedeutung |
|---|---|
| `invoices:read` | Rechnungen lesen, PDF/XML herunterladen |
| `invoices:write` | Rechnungen erstellen, finalisieren, versenden |
| `customers:read` | Kunden lesen |
| `customers:write` | Kunden anlegen |
| `mandanten:read` | Mandanten lesen |
| `mandanten:write` | Mandanten anlegen/ändern/löschen |
| `exports:read` | DATEV-Export herunterladen |
| `webhooks:read` | Webhooks und Zustellungen lesen |
| `webhooks:write` | Webhooks verwalten, Test senden |

## Sandbox vs. Live

Mit einem **Sandbox-Schlüssel** lässt sich die Anbindung kostenlos und
folgenlos testen. Es durchläuft dieselbe Pipeline wie Live, aber nichts wird
gespeichert.

| Aspekt | Sandbox | Live |
|---|---|---|
| Speicherung | keine (Trockenlauf) | vollständig |
| Erreichbare Endpunkte | nur Atomic + Hilfs­endpunkte (Preview/Convert) | alle |
| Limits/Kontingent | übersprungen | angewendet (402 bei Limit, 409 bei Dublette) |
| Idempotenz | optional | erforderlich |
| Antwort-Status | `200` mit `meta.sandbox = true` | `201` bei Anlage |

## Idempotenz

Schreibende Aufrufe (Atomic Invoice) erfordern im Live-Betrieb einen
Idempotenz-Schlüssel, damit ein versehentlich doppelt gesendeter Aufruf nicht
zwei Rechnungen erzeugt:

```
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

- **Format:** UUID.
- **Gültigkeit:** 48 Stunden.
- **Wiederholung:** Ein erneuter Aufruf mit demselben Schlüssel liefert die
  zwischengespeicherte Antwort zurück, gekennzeichnet mit dem Header
  `Idempotent-Replayed: true`.
- Im Sandbox-Modus ist der Schlüssel optional (es wird ohnehin nichts
  gespeichert).

## Limits (Throttling)

Es gibt zwei voneinander unabhängige Mengensteuerungen.

### Tageslimit (Aufrufe / 24 h)

Gilt **pro Schlüssel** und ist ein reiner **Anti-Missbrauchs-Schutz, keine
Preisstufe** — der API-Tarif wird nutzungsbasiert abgerechnet.

| Tarif | Aufrufe / 24 h | Schlüssel max. |
|---|---|---|
| `api` | 50.000 | 20 |

Wird das Limit überschritten, antwortet die API mit **HTTP 429** und einem
`Retry-After`-Header (Sekunden bis zum Reset). Das Fenster ist gleitend, es
gibt keinen Reset zu einer festen Uhrzeit. **Sandbox-Aufrufe zählen nicht mit.**
Begründete Ausreißer werden manuell angehoben.

### Weiche Dokument-Quota

Zusätzlich trägt jede schreibende Antwort den Stand der monatlichen
Dokument-Quota mit — **rein informativ, sie blockt einen Request nie**:

- in `meta.quota` mit `used`, `limit`, `percent`, `level`
- als Header `X-Factora-Documents-Used` und `X-Factora-Documents-Limit`

| Level | Auslastung | Verhalten |
|---|---|---|
| `ok` | < 80 % | normal |
| `warning` | 80–99 % | Hinweis, kein Block |
| `exceeded` | ≥ 100 % | Hinweis, kein Block |

Im API-Tarif ist die monatliche Dokumentgrenze **unbegrenzt**; abgerechnet wird
nutzungsbasiert.

### Harte Rechnungsgrenze (nur Free)

Davon zu unterscheiden ist die harte Monatsgrenze des Free-Tarifs
(5 Rechnungen/Monat). Wird sie erreicht, blockt die API das Erstellen mit
**HTTP 402** (`limit_reached`). Solo-, Business- und API-Tarif haben hier keine
harte Grenze.

## Fehler- und Antwortstruktur

Alle Antworten verwenden eine einheitliche Hülle:

```json
{
  "valid": true,
  "data": { },
  "errors": [],
  "meta": {}
}
```

Im Fehlerfall:

```json
{
  "valid": false,
  "data": null,
  "errors": [
    {
      "code": "schema",
      "severity": "error",
      "message": "Beschreibung des Fehlers",
      "field": "items[0].vat_rate",
      "rule": "BR-CO-3",
      "bt": "BT-152",
      "location": "//ram:ApplicableTradeTax"
    }
  ],
  "meta": {}
}
```

Die Felder der Hülle:

| Feld | Inhalt |
|---|---|
| `valid` | `true` bei Erfolg, `false` im Fehlerfall |
| `data` | das Ergebnis (Objekt oder Array), `null` im Fehlerfall |
| `errors` | Liste der Fehler-Einträge, bei Erfolg leer |
| `meta` | Zusatzinfos (Paginierung, Quota, `sandbox`-Flag) |

Das **Fehlerobjekt** enthält `code`, `severity` und `message` immer, die
übrigen Felder je nach Fehlerart: `field` (Eingabepfad), `rule`
(EN-16931-/Geschäftsregel), `bt` (Geschäftsbegriff) und `location` (XPath bei
XML-Befunden).

> Bei **5xx** greift die Hülle nicht — das ist ein unerwarteter Serverfehler.
> Bitte mit Request-Details an den Support melden.

### Erfolgs-Codes

| HTTP | Bedeutung |
|---|---|
| 200 | Erfolg (Lesen, Idempotenz-Replay, Sandbox-Trockenlauf, Preview) |
| 201 | Erstellt (Live-Rechnung, Kunde, Mandant, Webhook) |
| 204 | Kein Inhalt (erfolgreiches `DELETE`) |

### Fehlercodes

| HTTP | Code | Bedeutung |
|---|---|---|
| 400 | `schema` | Ungültiges JSON, fehlende/falsche Felder, Summen-Abweichung, ungültiger Idempotenz-Schlüssel |
| 400 | `business` | Verstoß gegen EN-16931-/GoBD-Geschäftsregel |
| 400 | `kosit` | Befund der amtlichen KoSIT-Prüfung |
| 400 | `invalid_state` | Rechnung im falschen Status (z. B. Versand eines Entwurfs) |
| 400 | `unsupported_format` | Eingabeformat nicht unterstützt |
| 401 | `authentication` | Schlüssel fehlt/ungültig/inaktiv/abgelaufen, IP nicht erlaubt |
| 402 | `limit_reached` | Harte Rechnungs-Obergrenze erreicht — **nur Free-Tarif** |
| 403 | `permission` | Kein API-Zugang, fehlender Scope, Sandbox-Schlüssel auf unzulässigem Endpunkt, IP nicht erlaubt |
| 404 | `not_found` | Objekt existiert nicht oder gehört zu anderem Konto |
| 405 | `method_not_allowed` | HTTP-Methode am Endpunkt nicht erlaubt |
| 406 | `not_acceptable` | Nicht unterstütztes Antwortformat |
| 409 | `conflict` | Doppelte Rechnungsnummer, Dublette, Mandant in Verwendung |
| 415 | `unsupported_media_type` | Nicht unterstützter `Content-Type` |
| 429 | `rate_limited` | Tageskontingent überschritten — Antwort trägt `Retry-After` |
| 502 | `bad_gateway` | Webhook-Zustellung fehlgeschlagen (Test) |

## Mandanten (Multi-Mandanten-Modell)

Ein Integrator kann unter einem Konto beliebig viele **Mandanten** (eigene
Endkunden) führen. Jeder Mandant hat eigene Verkäufer-Stammdaten. Ein Aufruf
wählt den Mandanten über `mandant_id` im Rechnungs-Payload; dessen
Stammdaten werden automatisch als Verkäufer eingesetzt. Kunden und Rechnungen
werden **pro Mandant getrennt** geführt. Siehe [Endpoints](api-endpoints.md)
und [Atomic Invoice](atomic-invoice.md).
