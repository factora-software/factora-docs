# Schnellstart — erste Rechnung in drei Schritten

Es gibt zwei Wege zur ersten validen Sandbox-Rechnung. Der erste braucht
keinen Editor und keine Datei, der zweite ist der, den Sie am Ende in Ihr
System einbauen. Beide führen durch dieselbe Pipeline.

## Voraussetzungen

- Ein Factora-Account im **API-Tarif**. Registrierung über die Console:
  `https://console.factora.software/auth/register`. Es gibt **keinen
  befristeten Trial** — **Sandbox-Schlüssel sind dauerhaft kostenlos**, der
  Live-Betrieb wird durch Hinterlegen einer Zahlungsmethode in der Console
  freigeschaltet. Für die Registrierung brauchen Sie eine USt-IdNr.
- Bestätigte E-Mail-Adresse.

> **Basis-URL aller Endpunkte:** `https://console.factora.software/api/v1/`

Zu Preisen und Mengenstaffel siehe [FAQ](faq.md#was-kostet-die-api).

---

# Weg A — Quickstart in der Console (empfohlen)

Drei Schritte, kein Code, keine Datei anlegen.

1. **Anmelden** unter `https://console.factora.software/auth/login`.
2. Im Menü **Quickstart** öffnen — direkt:
   `https://console.factora.software/dashboard/quickstart`
3. Der Quickstart legt den **Sandbox-Schlüssel** an, zeigt einen
   **vollständigen Beispiel-Request** samt fertigem `curl` und hat einen
   Knopf **„Test-Request senden"**. Ein Klick, und die Antwort steht darunter.

Danach haben Sie einen Schlüssel, einen funktionierenden Request und die
Bestätigung, dass die Kette steht. Erst dann lohnt sich der Blick in die
Feldreferenz.

Warum dieser Weg zuerst steht: Der Beispiel-Request der Console ist
vollständig — er trägt jedes Feld, das das Standardprofil `xrechnung`
verlangt. Von Hand zusammengesetzte erste Requests scheitern in der Praxis
fast immer an genau zwei Feldern (`seller_snapshot.contact` und
`invoice_header.buyer_reference`).

---

# Weg B — mit curl

## 1. API-Schlüssel

Schlüssel erstellen Sie in der Console unter
`https://console.factora.software/dashboard/api-keys` (oder Sie nehmen den,
den der Quickstart schon angelegt hat).

- Der Schlüssel beginnt je nach Umgebung mit **`fa_live_`** (Live) bzw.
  **`fa_test_`** (Sandbox). Ältere Schlüssel (`fa_…`) authentifizieren
  weiterhin.
- Sandbox-Schlüssel sind kostenlos und führen `POST /invoices/atomic/` als
  Trockenlauf aus: volle Validierung, ein sichtbar entwertetes PDF als
  Vorschau (kein XML), nichts wird gespeichert — siehe
  [Sandbox & Testing](../reference/sandbox.md).
- Der Klartext wird **genau einmal** angezeigt. Serverseitig liegt kein
  Klartext, nur ein SHA-256-Hash und ein HMAC. Geht der Schlüssel verloren,
  gibt es kein Recovery — neuen erstellen, alten deaktivieren.
- Optional pro Schlüssel: **IP-Whitelist**, **Scopes**, **Ablaufdatum** —
  Details unter [API-Überblick](../reference/api-overview.md#api-schlüssel).

Das Tageslimit beträgt **50.000 Aufrufe / 24 h** je Schlüssel bei **bis zu
20 Schlüsseln** je Konto. Es ist ein Anti-Missbrauchs-Schutz, keine
Preisstufe.

## 2. Authentifizierung

Jeder Aufruf trägt den Schlüssel im `Authorization`-Header als Bearer-Token:

```
Authorization: Bearer fa_test_xxx…
```

## 3. Die erste Rechnung

Ein Aufruf erstellt, validiert und finalisiert die Rechnung und liefert PDF
und XML als Base64 zurück. Der folgende Rumpf ist **vollständig** — er
enthält jedes Feld, das das Standardprofil `xrechnung` verlangt, und lässt
sich unverändert absenden:

```bash
curl -X POST https://console.factora.software/api/v1/invoices/atomic/ \
  -H 'Authorization: Bearer fa_test_xxx…' \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: 9f4b7d92-3e4a-4b1d-9c6f-1234567890ab' \
  -d '{
  "api_mode": "atomic_single_post",
  "invoice_header": {
    "invoice_number": "RE-2026-0001",
    "invoice_date": "2026-05-22",
    "payment_due_date": "2026-06-21",
    "buyer_reference": "PO-2026-001",
    "profile": "xrechnung"
  },
  "seller_snapshot": {
    "name": "Muster GmbH",
    "street": "Hauptstr. 1",
    "zip": "10115",
    "city": "Berlin",
    "vat_id": "DE123456789",
    "iban": "DE21500500009876543210",
    "contact": {
      "name": "E. Musterfrau",
      "phone": "+49 30 1234567",
      "email": "rechnung@muster.example"
    }
  },
  "buyer": {
    "name": "Beispiel AG",
    "street": "Bahnhofstr. 2",
    "zip": "60486",
    "city": "Frankfurt am Main",
    "country": "DE",
    "contact": { "email": "einkauf@beispiel.example" }
  },
  "items": [
    {
      "description": "Beratungsleistung",
      "quantity": "1.000",
      "unit": "C62",
      "unit_price_net": "100.00",
      "vat_rate": "19.00"
    }
  ],
  "validation_totals": {
    "net_amount": "100.00",
    "vat_amount": "19.00",
    "gross_amount": "119.00"
  }
}'
```

Derselbe Rumpf noch einmal einzeln, zum Kopieren in ein Werkzeug Ihrer Wahl:

```json
{
  "api_mode": "atomic_single_post",
  "invoice_header": {
    "invoice_number": "RE-2026-0001",
    "invoice_date": "2026-05-22",
    "payment_due_date": "2026-06-21",
    "buyer_reference": "PO-2026-001",
    "profile": "xrechnung"
  },
  "seller_snapshot": {
    "name": "Muster GmbH",
    "street": "Hauptstr. 1",
    "zip": "10115",
    "city": "Berlin",
    "vat_id": "DE123456789",
    "iban": "DE21500500009876543210",
    "contact": {
      "name": "E. Musterfrau",
      "phone": "+49 30 1234567",
      "email": "rechnung@muster.example"
    }
  },
  "buyer": {
    "name": "Beispiel AG",
    "street": "Bahnhofstr. 2",
    "zip": "60486",
    "city": "Frankfurt am Main",
    "country": "DE",
    "contact": { "email": "einkauf@beispiel.example" }
  },
  "items": [
    {
      "description": "Beratungsleistung",
      "quantity": "1.000",
      "unit": "C62",
      "unit_price_net": "100.00",
      "vat_rate": "19.00"
    }
  ],
  "validation_totals": {
    "net_amount": "100.00",
    "vat_amount": "19.00",
    "gross_amount": "119.00"
  }
}
```

Der Header **`Idempotency-Key`** (UUID, Gültigkeit 48 h) verhindert
Duplikat-Rechnungen: Bei identischem Schlüssel wird die ursprüngliche Antwort
zurückgegeben — sicher für Wiederholungen. Auf `POST /invoices/atomic/` ist
er **Pflicht**.

## 4. Antwortstruktur

Alle Antworten folgen derselben Hülle:

```json
{ "valid": true, "data": { }, "errors": [ ], "meta": { } }
```

- **`valid`** — `true` bei Erfolg, `false` im Fehlerfall.
- **`data`** — das Ergebnis (bei Listen ein Array, bei Detail-Aufrufen ein
  Objekt), `null` im Fehlerfall. Die Nutzdaten liegen **immer** hier, nie auf
  oberster Ebene.
- **`errors`** — bei Erfolg leer; im Fehlerfall Einträge mit `code`,
  `severity`, `message` und je nach Fehlerart `field`, `rule`, `bt`,
  `location`.
- **`meta`** — Zusatzinfos (Paginierung, Quota, `sandbox`-Flag).

Im Sandbox-Betrieb ist der Status `200`, `data.id` ist `null`,
`data.status` ist `"sandbox"` und `meta.sandbox` ist `true` — es wird nichts
gespeichert. `data.pdf_base64` ist dann eine sichtbar entwertete Vorschau
(Wasserzeichen), `data.xml_base64` ist immer `null`. Live ist der Status `201`
mit PDF und XML.

## 5. Weitere Aufrufe

```bash
curl https://console.factora.software/api/v1/invoices/42/pdf/ \
  -H 'Authorization: Bearer fa_live_xxx…' \
  -o invoice-42.pdf
```

Die vollständige Endpunktliste mit Scopes und Sandbox-Eignung steht unter
[Endpoints](../reference/api-endpoints.md), das komplette Feldschema unter
[Atomic Invoice](../reference/atomic-invoice.md) — dort auch, welche Felder
das Profil `xrechnung` zusätzlich verlangt.

---

## Häufige Stolpersteine

- **400 mit `seller_snapshot.contact` oder `invoice_header.buyer_reference`**
  — die beiden häufigsten Ablehnungen überhaupt. Beide Felder sind unter dem
  Standardprofil `xrechnung` Pflicht (BR-DE-6/7 und BR-DE-15). Die
  vollständige Liste steht unter
  [Profilabhängige Pflichtfelder](../reference/atomic-invoice.md#profilabhängige-pflichtfelder).
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
