# Factora — API-Integrationsleitfaden

_E-Rechnung (XRechnung / ZUGFeRD) per REST-API_

# 1. Überblick

Factora erzeugt EN-16931-konforme E-Rechnungen aus Ihrem ERP-, Warenwirtschafts- oder Abrechnungssystem über eine REST-API. Ausgabeformate: **XRechnung 3.0** (CII), **Peppol BIS 3.0** (UBL) und **ZUGFeRD** (PDF/A-3 mit eingebettetem XML). Alle 164 Felder der EN 16931 werden unterstützt (siehe separate Feld-Referenz).

Der empfohlene Weg ist der **atomic-Endpoint**: ein einziger Request erstellt, validiert, finalisiert die Rechnung und liefert XML + PDF zurück. Wer mehr Kontrolle braucht, kann den Lebenszyklus auch über die granularen Endpoints steuern (Abschnitt 7).

# 2. Authentifizierung

- Jeder Request trägt den Header `Authorization: Bearer fa_…`.
- Der Schlüssel (Prefix `fa_`) wird **einmalig bei der Erstellung** angezeigt; Factora speichert nur einen Hash. Geht er verloren, wird ein neuer erzeugt.
- Optional pro Schlüssel konfigurierbar: **IP-Whitelist** (einzelne IP-Adressen, keine CIDR-Bereiche), **Scopes** (Lese-/Schreibrechte) und **Ablaufdatum**.
- Bezug des Schlüssels: über die Console (Self-Service) oder durch Factora bereitgestellt. Voraussetzung ist der API-Tarif.

> Der API-Schlüssel ist ein Vollzugriff auf das Konto. Niemals in Repos, Logs oder Client-Code ablegen; serverseitig speichern und die IP-Whitelist nutzen.

# 3. Basis-URL & Antwortformat

- Basis-URL: `https://console.factora.software/api/v1/`
- Alle Bodies sind `application/json` (UTF-8).
- PDF und XML werden als **Base64** im JSON-Body geliefert.

# 4. Schnellstart — atomic

Pflicht-Header: `Authorization`, `Content-Type: application/json` und `Idempotency-Key` (siehe Abschnitt 5).

**Request**

```
POST /api/v1/invoices/atomic/
Authorization: Bearer fa_****************
Content-Type: application/json
Idempotency-Key: 7e0e8a2c-3b1d-4f6a-9c2e-1a2b3c4d5e6f

{
  "invoice_header": {
    "invoice_number": "RE-2026-0001",
    "invoice_date": "2026-05-22",
    "payment_due_date": "2026-06-21",
    "buyer_reference": "04011000-12345-06",
    "order_reference": "DB-4500123456",
    "profile": "xrechnung"
  },
  "seller_snapshot": {
    "name": "Muster GmbH", "street": "Hauptstr. 1",
    "zip": "10115", "city": "Berlin", "vat_id": "DE123456789",
    "iban": "DE21500500009876543210"
  },
  "buyer": {
    "name": "Beispiel AG", "street": "Bahnhofstr. 2",
    "zip": "60486", "city": "Frankfurt am Main", "country": "DE",
    "contact": { "email": "einkauf@beispiel.example" }
  },
  "items": [
    { "description": "Beratungsleistung", "quantity": "1",
      "unit": "C62", "unit_price_net": "100.00", "vat_rate": "19" }
  ],
  "validation_totals": {
    "net_amount": "100.00", "vat_amount": "19.00", "gross_amount": "119.00"
  }
}
```

**Response**

```
201 Created

{
  "id": 123,
  "invoice_number": "RE-2026-0001",
  "status": "final",
  "profile": "xrechnung",
  "total": "119.00",
  "pdf_base64": "JVBERi0xLjQ...",   ZUGFeRD-PDF (PDF/A-3)
  "xml_base64": "PD94bWwg..."        XRechnung (CII)
}
```

Die wichtigsten Bausteine des Request-Bodys:

| Feld | Inhalt |
| --- | --- |
| invoice_header | Kopfdaten: Nummer, Datum, Fälligkeit, Referenzen (u. a. BT-10 buyer_reference, BT-13 order_reference). |
| invoice_header.profile | Optional. Semantisches Profil: `xrechnung` (deutscher CIUS inkl. BR-DE, Standard) oder `en16931` (europäisches Kernmodell). Weggelassen = `xrechnung`. Unabhängig vom Ausgabeformat; das aufgelöste Profil steht immer in der Antwort unter `profile`. |
| seller_snapshot | Verkäuferdaten inkl. IBAN/USt-IdNr. — wird GoBD-konform eingefroren. |
| buyer | Käuferdaten; buyer.contact.email = BT-58. Kunde wird bei Bedarf automatisch angelegt/gefunden. |
| items[] | Positionen: Beschreibung, Menge, Einheit (UNCL5272), Nettopreis, USt-Satz, optional Rabatte/Zuschläge/Klassifizierung. |
| validation_totals | Ihre ERP-Summen. Factora rechnet nach und weist Abweichungen ab (Schutz vor Rundungs-/Logikfehlern). |
| extended_fields | Optionale Zusatzfelder; werden mit den Schlüssel-Standardwerten gemerged (Abschnitt 6). |
| attachments[] | Optionale Anhänge (BG-24): eingebettet (Base64) oder als externer Link. |

# 5. Idempotenz (Pflicht bei atomic)

- Der Header `Idempotency-Key` (eine UUID) ist auf dem atomic-POST **erforderlich** — fehlt er, antwortet die API mit `400`.
- Ein erneuter Request mit **demselben** Schlüssel innerhalb von **48 Stunden** liefert die ursprüngliche Antwort zurück (Header `Idempotent-Replayed: true`) und legt **keine** zweite Rechnung an.
- Pro echtem Geschäftsvorfall einen neuen UUID erzeugen.

# 6. Standardwerte je Zugang

Pro API-Schlüssel lassen sich **Standardwerte** hinterlegen, die automatisch in jede atomic-Anfrage übernommen werden. Werte aus dem Request haben Vorrang. Ideal für wiederkehrende Konstanten — etwa feste Referenzen im Deutsche-Bahn-Prozess. So entfällt das manuelle Pflegen pro Rechnung.

# 7. Granulare Endpoints

| Zweck | Endpoint |
| --- | --- |
| Alles-in-einem (empfohlen) | POST /api/v1/invoices/atomic/ |
| Anlegen / Auflisten | GET, POST /api/v1/invoices/ |
| Detail | GET /api/v1/invoices/{id}/ |
| Finalisieren | POST /api/v1/invoices/{id}/finalize/ |
| Versenden (E-Mail) | POST /api/v1/invoices/{id}/send/ |
| ZUGFeRD-PDF | GET /api/v1/invoices/{id}/pdf/ |
| XRechnung-XML (CII/UBL) | GET /api/v1/invoices/{id}/xml/ |
| Transport / Peppol-Versand | POST /api/v1/invoices/{id}/dispatch/ |
| Kunden (lesend) | GET /api/v1/customers/ |
| DATEV-Export | GET /api/v1/exports/datev/ |
| Webhooks | /api/v1/webhooks/… |

# 8. DATEV-Export

Über `GET /api/v1/exports/datev/` liefert Factora einen DATEV-konformen Export (Buchungsstapel- bzw. Rechnungsdaten-Format, CP1252).

> Es handelt sich um einen Datei-Export zum Import in DATEV, nicht um einen vollautomatischen Live-Upload in DATEV.

# 9. Limits & Tarife

- Es gibt **einen** nutzungsbasierten API-Tarif; abgerechnet wird pro finalisierter Rechnung über die Console.
- **Tageslimit: 50.000 Aufrufe / 24 h** pro Schlüssel — ein Anti-Missbrauchs-Schutz, keine Preisstufe. Überschreitung → `429` mit `Retry-After`.
- **Bis zu 20 Schlüssel** je Konto; die Last lässt sich darüber verteilen.
- Sandbox-Aufrufe zählen nicht auf das Tageslimit.
- Die monatliche Dokument-Quota ist im API-Tarif **unbegrenzt** und wird nur informativ mitgeliefert (`meta.quota` sowie die Header `X-Factora-Documents-Used` / `X-Factora-Documents-Limit`) — sie blockt einen Request nie.
- Davon zu unterscheiden: `402` = harte Rechnungs-Obergrenze; die greift ausschließlich im Free-Tarif, nicht im API-Tarif.

# 10. Fehlercodes

| Status | Bedeutung |
| --- | --- |
| 201 | Rechnung erstellt (atomic) — XML + PDF im Body. |
| 400 | Ungültiger Body, ungültiges JSON, fehlender/ungültiger Idempotency-Key oder Validierungsfehler (z. B. Summen weichen ab). |
| 401 | Authentifizierung fehlgeschlagen (Schlüssel fehlt/ungültig /abgelaufen, IP nicht erlaubt). |
| 402 | Harte Rechnungs-Obergrenze erreicht (nur Free-Tarif) — Upgrade erforderlich, kein Rate-Limit. |
| 403 | Kein API-Zugang (Tarif) oder fehlender Scope. |
| 409 | Rechnungsnummer existiert bereits. |
| 429 | Tageskontingent des Schlüssels erschöpft (50.000 Aufrufe / 24 h) — Antwort trägt `Retry-After`. |
