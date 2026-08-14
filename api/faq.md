# Factora API — FAQ

> Häufige Fragen rund um Public API v1, Onboarding, Tarife und Integration.
> Ergänzt das interaktive Schema unter `/api/docs/` und die Postman-Collection
> in `postman/factora_api_collection.json`.

## Onboarding

### Wie registriere ich mich als Integrator?

Über den Self-Service-Pfad:
- `https://console.factora.software/auth/register?tier=api`

Pflichtfelder: Firmenname, USt-IdNr., Tech-Kontakt-E-Mail, Passwort.
Nach Bestätigung der E-Mail landest du direkt im Integrator-Dashboard.

### Was ist der Tech-Kontakt?

Die E-Mail-Adresse, an die die Onboarding-Mail und perspektivisch
API-Statusmeldungen gehen. Aktuell identisch mit dem Login der
Owner-Rolle.

### Brauche ich eine Kreditkarte zum Ausprobieren?

Nein — und es gibt auch keine Frist. Es existiert **kein zeitlich
begrenzter Trial**. Nach der Registrierung stehen sofort **Sandbox-Keys**
(`fa_test_…`) bereit, die **dauerhaft kostenlos** sind: `POST
/api/v1/invoices/atomic/` läuft damit als Dry-Run durch dieselbe Pipeline
(Validierung, PDF + XML), persistiert aber nichts (`sandbox: true`).

Ein Zahlungsmittel wird erst für den **Live-Betrieb** hinterlegt
(Console → Abrechnung). Damit wird die metered Stripe-Subscription aktiv
und Live-Keys (`fa_live_…`) lassen sich erzeugen.

### Was gilt, solange keine Zahlungsmethode hinterlegt ist?

Der Tenant steht auf `pending_card`:
- Daten und Keys bleiben erhalten
- Sandbox-Keys funktionieren unverändert und kostenlos
- Über Live-Keys sind nur lesende Aufrufe (GET / HEAD / OPTIONS) sowie
  `POST /api/v1/invoices/convert/` (persistiert nichts) erreichbar;
  jeder andere schreibende Aufruf wird abgelehnt
- Freischaltung jederzeit über den Stripe-Checkout im Abrechnungs-Tab

Dasselbe Verhalten greift bei `paused` — das ist ein **kostenpflichtiges
Abo mit fehlgeschlagener Zahlung**, nicht ein abgelaufener Testzeitraum.
Finalisierte Rechnungen bleiben in jedem dieser Zustände abrufbar (GoBD:
10 Jahre Aufbewahrung).

### Was kostet die API?

Es gibt **einen** nutzungsbasierten API-Tarif — keine feste Preis-Leiter
(`api_starter`/`api_pro` gibt es nicht mehr). Abgerechnet wird über die
**Console** (`console.factora.software`):

| Posten | Wert |
|---|---|
| Plattform-Gebühr | **49 €/Monat** netto |
| 1–250 Live-Rechnungen/Monat | **0 €** (in Plattform-Gebühr enthalten) |
| 251–1.000 | **0,12 €** / Rechnung |
| 1.001–5.000 | **0,07 €** / Rechnung |
| ab 5.001 | **0,04 €** / Rechnung |
| Sandbox-Keys | kostenlos |

Sandbox-Keys sind kostenlos zum Entwickeln und Testen; die Staffel ist
graduiert und kontoweit gepoolt. Volumen-/Preisdetails immer in der
Console prüfen.

## Authentifizierung

### Wie erstelle ich meinen ersten API-Key?

1. Login → `/dashboard/integrator`
2. Button "Neuen Key erstellen"
3. Key wird **genau einmal** angezeigt (Prefix `fa_…`, 64 Zeichen)
4. Sofort in dein Secrets-Management übernehmen — Factora speichert nur
   den SHA-256-Hash, kein Recovery möglich

### Was tun, wenn ein Key kompromittiert wurde?

Im Integrator-Dashboard löschen (`DELETE /api/api-keys/<id>/`) und einen
neuen Key erstellen. Der gelöschte Key wird sofort serverseitig invalidiert.

### Kann ich einen Key auf bestimmte IPs beschränken?

Ja, beim Anlegen des Keys eine IP-Whitelist setzen. Erlaubt sind
ausschließlich **einzelne IP-Adressen** — CIDR-Bereiche werden bewusst
abgelehnt, weil die Prüfung serverseitig auf exakte Übereinstimmung geht.
Leere Liste = alle IPs erlaubt. Requests von anderen IPs erhalten HTTP 401.

### Welche Scopes gibt es?

Neun, jeweils mit Doppelpunkt geschrieben:

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

Eine **leere** Scope-Liste bedeutet: alle Rechte. Die Prüfung ist
methodenbasiert — `GET` verlangt den `…:read`-Scope, `POST`/`PATCH`/
`DELETE` den `…:write`-Scope.

## Endpoints

### Wann nutze ich `/v1/invoices/atomic/` vs. den Step-by-Step-Flow?

**Atomic** (ein Request, ein Result):
- ERP-System hält Stammdaten selbst
- Snapshots (Seller, Buyer, Shipping) werden komplett mitgeschickt
- Antwort enthält ZUGFeRD-PDF + XRechnung-XML als Base64
- Geeignet für Pipelines mit vollem Rechnungszustand zur Übergabe

**Step-by-Step** (`/v1/customers/`, `/v1/invoices/`, dann `finalize`,
`pdf`, `xml`):
- App-zentrierte Nutzung mit Factora als führendem System
- Customer-Stammdaten leben in Factora
- Mehrere Rechnungen pro Customer → spart Redundanz

### Wofür ist der `Idempotency-Key`?

Der Header `Idempotency-Key: <uuid>` schützt POSTs vor versehentlichen
Duplikaten (Netzwerk-Retry, Browser-Refresh, Worker-Wiederholung):
- TTL: 48 Stunden
- Bei identischem Key wird die ursprüngliche Antwort 1:1
  zurückgegeben — kein zweiter DB-Insert
- Aktuell unterstützt: `POST /api/v1/invoices/atomic/`

Empfehlung: pro logischem Geschäftsvorgang einmalig generieren
(z.B. UUID v4 deines ERP-Auftrags) und bei Retries beibehalten.

### Was bedeutet `order_reference` (BT-13)?

Die Bestellnummer aus dem Käufer-System. Wird in ZUGFeRD und XRechnung
als BT-13 (`Purchase order reference`) ausgegeben. Pflicht für viele
B2B- und B2G-Empfänger (z.B. Behörden mit Leitweg-ID).

### Wie viele Items sind pro Rechnung erlaubt?

Aktuell kein hartes Limit, aber der Atomic-Request darf insgesamt nicht
größer als 10 MB sein (inklusive Base64-Anhänge). Bei sehr großen
Rechnungen Step-by-Step nutzen.

## Rate-Limits & Errors

### Was passiert bei HTTP 429?

Tageskontingent erschöpft (**50.000 Aufrufe / 24 h** im API-Tarif — ein
Anti-Missbrauchs-Schutz, keine Preisstufe). Der Response enthält:
- Header `Retry-After: <sekunden>` (Sekunden bis Reset)
- Body mit `errors[0].code = "rate_limited"`

Das Fenster ist **gleitend über 24 Stunden**, kein Reset zu einer festen
Uhrzeit — `Retry-After` abwarten. Das Kontingent gilt pro Key, nicht pro
Tenant; je Konto sind bis zu 20 Keys möglich, die Last lässt sich also
verteilen. Sandbox-Aufrufe zählen nicht mit.

### Was bedeutet HTTP 402?

`Payment Required` — die Subscription ist nicht aktiv
(`pending_card`, `paused`, `expired`, `canceled` oder `past_due`).
Lesende Aufrufe und `/convert/` bleiben erreichbar, schreibende nicht.
Der Owner hinterlegt im Abrechnungs-Tab eine Zahlungsmethode bzw.
reaktiviert das Abo über den Stripe-Checkout.

### Welche Felder kommen bei einem Validation-Error zurück?

```json
{
  "data": null,
  "errors": [
    {
      "code": "invalid_vat_id",
      "field": "customer.vat_id",
      "message": "USt-IdNr. ist ungültig oder nicht im EU-VIES-System bekannt."
    }
  ],
  "meta": {"request_id": "req_xxx"}
}
```

`request_id` bei Bug-Reports an Support mitschicken.

## Tooling

### Gibt es eine Postman-Collection?

Ja: `postman/factora_api_collection.json` (im Repo, mit Environment
`postman/factora_dev_environment.json`). Variablen vor erstem Use
setzen: `base_url`, `api_key`. `customer_id` und `invoice_id` werden
durch die Test-Skripte gesetzt.

### Gibt es ein OpenAPI-Schema?

- Raw YAML/JSON: `GET /api/schema/`
- Interaktive Swagger-UI: `/api/docs/`
- ReDoc: `/api/redoc/`

### Wo finde ich Code-Beispiele?

cURL-Snippets im Schema-Description (sichtbar oben im Swagger-UI).
Für andere Sprachen: OpenAPI-Generator gegen `/api/schema/` laufen
lassen — generiert Client-SDKs für ~50 Sprachen.

## Compliance

### Speichert Factora die API-Keys im Klartext?

Nein. Nur SHA-256-Hash. Beim Vergleich wird der eingehende Bearer-Token
gehasht und mit dem gespeicherten Hash verglichen.

### Wo werden meine Daten gehostet?

Hetzner Cloud, Standort Nürnberg (eu-central-1). Keine US-Provider im
Pfad für personenbezogene Daten (DSGVO-konform).

### Brauche ich einen AVV?

Ja, vor produktivem Einsatz. AVV wird beim Tarif-Upgrade automatisch
zur Unterschrift gestellt (in Vorbereitung — bis dahin auf Anfrage:
`legal@factora.software`).

### Sind die Rechnungen revisionssicher (GoBD)?

Ja. Finalisierte Rechnungen sind im Backend immutable (Number, Datum,
Beträge, Customer-Snapshot, Audit-Log). Aufbewahrungspflicht 10 Jahre
liegt beim Rechnungssteller — Factora archiviert solange der Tenant
aktiv ist.

## Support

`support@factora.software` — Antworten kommen vom Founder persönlich.
Bei Bug-Reports bitte mitschicken:
- API-Key-Suffix (letzte 4 Zeichen) — niemals den ganzen Key
- `request_id` aus dem Response-Body
- Endpoint + Methode + ungefähre Uhrzeit (UTC)
- Repro-Schritte oder cURL-Snippet
