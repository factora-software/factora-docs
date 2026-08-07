# Sandbox & Testing

Sandbox-Schlüssel (`fa_test_…`) sind **dauerhaft kostenlos** und durchlaufen
dieselbe Pipeline wie der Live-Betrieb (Validierung, PDF- und XML-Erzeugung) —
**persistieren aber nichts**. Es gibt keinen zeitlich begrenzten Trial: Sie
testen beliebig lange mit Sandbox-Schlüsseln und schalten den Live-Betrieb
erst frei, wenn Sie eine Zahlungsmethode hinterlegen.

Anlegen eines Sandbox-Schlüssels: siehe
[API-Überblick](api-overview.md#api-schlüssel).

## Wo Sandbox-Schlüssel funktionieren

Sandbox-Schlüssel sind **nur** auf den persistenzfreien Endpunkten zugelassen:

| Endpunkt | Verhalten mit Sandbox-Schlüssel |
|---|---|
| `POST /invoices/atomic/` | Trockenlauf: validiert und erzeugt PDF/XML, verwirft danach |
| `POST /invoices/preview/` | Summen-Vorschau (dezimalgenau, inkl. Positions-Nachlässe und -Zuschläge) |

Alle anderen v1-Endpunkte lehnen Sandbox-Schlüssel mit **403** ab — sie würden
echte Kontodaten lesen oder schreiben.

## Was der Trockenlauf macht

`POST /invoices/atomic/` mit einem Sandbox-Schlüssel:

- läuft durch die **vollständige** Validierung (EN-16931-Geschäftsregeln,
  KoSIT) und erzeugt **PDF und XML** als Base64 in der Antwort,
- antwortet mit **HTTP 200** (Live erzeugt `201`),
- setzt `meta.sandbox: true`,
- **speichert nichts** — keine Rechnung, keinen Kunden, **keine
  Nummernkreis-Reservierung**,
- **zählt nicht** auf Kontingente und Limits,
- **überspringt die Idempotenz** vollständig: der `Idempotency-Key`-Header ist
  hier optional und wird ignoriert (ohne Persistenz ist kein Replay nötig).

Sandbox umgeht außerdem die Abo-Prüfung — Testen bleibt kostenlos, unabhängig
vom Abrechnungsstatus.

## Beispiel

Trockenlauf einer kompletten Rechnung:

```bash
curl -X POST https://console.factora.software/api/v1/invoices/atomic/ \
  -H 'Authorization: Bearer fa_test_xxx…' \
  -H 'Content-Type: application/json' \
  -d @invoice.json
```

Antwort (gekürzt):

```json
{
  "valid": true,
  "data": {
    "id": null,
    "status": "sandbox",
    "pdf_base64": "JVBERi0…",
    "xml_base64": "PD94bWw…"
  },
  "errors": [],
  "meta": { "sandbox": true }
}
```

## Hinweise

- **403 Forbidden** — Sandbox-Schlüssel auf einem nicht zugelassenen Endpunkt
  (alles außer `atomic` und `preview`).
- Sandbox prüft **keine** Dubletten, da nichts gespeichert wird.
- Für den Wechsel auf Live: in der Console eine Zahlungsmethode hinterlegen und
  einen **Live-Schlüssel** (`fa_live_…`) erstellen.

Siehe auch [Schnellstart](../api/quickstart.md) und
[Endpoints](api-endpoints.md).
