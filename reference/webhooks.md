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

Beim Anlegen eines Webhooks wählen Sie die abonnierten Ereignisse. Zum Testen
existiert zusätzlich `webhook.test` (siehe unten).

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

> Das Secret wird auf Ihrer Seite gewählt und bei Factora verschlüsselt
> gespeichert. Es verlässt das System nicht im Klartext.

## Zustellung & Wiederholung

| Eigenschaft | Wert |
|---|---|
| Erfolgskriterium | HTTP-Status 200–299 |
| Timeout | 10 Sekunden je Versuch |
| Maximale Versuche | 5 |
| Wiederholungsabstände | 30 s · 2 min · 10 min · 1 h · 6 h |
| Weiterleitungen | werden nicht gefolgt (Schutz gegen Rebinding) |
| Automatische Abschaltung | nach mehreren aufeinanderfolgenden Fehlschlägen |

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
