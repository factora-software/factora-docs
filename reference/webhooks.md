# Webhooks

Webhooks benachrichtigen ein Fremdsystem in Echtzeit über Ereignisse zu
Rechnungen. Factora sendet dazu einen signierten HTTP-POST an eine von Ihnen
hinterlegte HTTPS-URL.

Verwaltung über die Endpunkte unter `/api/v1/webhooks/` (siehe
[Endpoints](api-endpoints.md)).

## Ereignistypen

| Ereignis | Auslöser |
|---|---|
| `invoice.created` | Rechnungsentwurf angelegt |
| `invoice.finalized` | Rechnung finalisiert (PDF/XML erzeugt) |
| `invoice.dispatched` | Rechnung zum Transport eingereiht |
| `invoice.delivered` | Rechnung erfolgreich zugestellt (E-Mail/Peppol) |
| `invoice.paid` | Rechnung als bezahlt markiert |
| `invoice.cancelled` | Rechnung storniert |
| `invoice.rejected` | Rechnung abgelehnt (Prüf-/Zustellfehler) |

Beim Anlegen eines Webhooks wählen Sie über `subscribed_events` die abonnierten
Ereignisse. Zum Testen existiert zusätzlich `webhook.test` (siehe unten).

## Payload

Der Rumpf ist ein **flaches JSON-Objekt** — ein Schnappschuss der Rechnung zum
Zeitpunkt des Ereignisses:

| Feld | Inhalt |
|---|---|
| `event_type` | Ereignistyp (siehe oben) |
| `invoice_number` | Rechnungsnummer |
| `invoice_id` | interne ID |
| `status` | Rechnungsstatus |
| `invoice_type` | Rechnungstyp |
| `total` / `subtotal` / `tax_amount` | Beträge (String, Dezimal) |
| `currency` | Währungscode |
| `customer_name` | Kundenname |
| `invoice_date` / `due_date` | Rechnungs- und Fälligkeitsdatum (ISO) |
| `occurred_at` | Zeitstempel des Ereignisses (ISO 8601) |

Beispiel (`invoice.finalized`):

```json
{
  "event_type": "invoice.finalized",
  "invoice_number": "RE-2026-0001",
  "invoice_id": 42,
  "status": "finalized",
  "total": "595.00",
  "currency": "EUR",
  "occurred_at": "2026-06-18T10:15:00+00:00"
}
```

## Aufbau der Zustellung

Jeder Webhook-Request enthält:

| Header | Wert |
|---|---|
| `Content-Type` | `application/json` |
| `X-Factora-Event` | Ereignistyp (z. B. `invoice.finalized`) |
| `X-Factora-Signature` | `sha256=<hex>` — HMAC-SHA256 des Rumpfes |
| `User-Agent` | `Factora-Webhooks/1.0` |

## Signaturprüfung

Die Signatur sichert Echtheit und Unverändertheit. Berechnung:

```
X-Factora-Signature = "sha256=" + HMAC_SHA256(secret, raw_request_body)
```

Der Empfänger berechnet denselben HMAC mit dem **bei der Webhook-Anlage
vereinbarten Secret** über den **rohen Request-Body** und vergleicht das
Ergebnis. Stimmen die Werte nicht überein, ist der Request abzulehnen.

> Das Secret wird auf Ihrer Seite gewählt und bei Factora **AES-256-GCM**-
> verschlüsselt gespeichert. Es verlässt das System nicht im Klartext.

## Zustellung & Wiederholung

| Eigenschaft | Wert |
|---|---|
| Erfolgskriterium | HTTP-Status 200–299 |
| Timeout | 10 Sekunden je Versuch |
| Maximale Versuche | 5 |
| Wiederholungsabstände | 30 s · 2 min · 10 min · 1 h · 6 h |
| Weiterleitungen | werden nicht gefolgt (Schutz gegen Rebinding) |
| Gespeicherter Antwort-Rumpf | die ersten 2.000 Zeichen |
| Automatische Abschaltung | nach **8** aufeinanderfolgenden Fehlschlägen |

Ein automatisch abgeschalteter Endpunkt lässt sich in der Console wieder
aktivieren.

Die Zustellhistorie eines Webhooks ist über
`GET /webhooks/{id}/deliveries/` abrufbar (Versuche, Status, Antwortzeit,
nächste Wiederholung). Jede Zustellung führt u. a. `event_type`, `attempts`,
`max_attempts`, `response_status`, `success`, `completed_at`.

## Test-Webhook

`POST /webhooks/{id}/test/` sendet **synchron** einen Test-Request an die
hinterlegte URL und gibt das Ergebnis direkt zurück:

```json
{ "valid": true, "data": { "success": true, "status_code": 200 }, "meta": { "duration_ms": 342 } }
```

Ist die URL nicht erreichbar, antwortet die API mit `success: false` und
einer Fehlerbeschreibung (HTTP 502).

## Sicherheitsanforderungen an die Ziel-URL

- Nur **HTTPS**.
- Nur **öffentlich erreichbare** Adressen (interne/lokale Hosts werden
  abgelehnt — SSRF-Schutz).
- Die Prüfung läuft nicht nur beim Anlegen, sondern **unmittelbar vor jedem
  Versand erneut** — damit greift ein nachträgliches Umbiegen des DNS-Namens
  auf eine interne Adresse (DNS-Rebinding) nicht.
