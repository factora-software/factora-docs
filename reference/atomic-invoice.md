# Atomic Invoice — vollständiges Feldschema

`POST /api/v1/invoices/atomic/`

Erstellt, finalisiert und prüft eine Rechnung in einem einzigen Aufruf und
liefert das fertige ZUGFeRD-PDF und die XRechnung-XML zurück. Dieses Dokument
listet **jedes Feld** des Payloads mit Typ, Pflicht/Optional, Default,
Format und EN-16931-Bezug (BT-Codes).

Konventionen: Beträge sind Dezimal-Strings (2 Nachkommastellen), Mengen
3 Stellen, Sätze 2 Stellen; Datumsangaben ISO 8601 (`YYYY-MM-DD`).

## Aufbau des Payloads

| Feld | Typ | Pflicht | Default | Bemerkung |
|---|---|---|---|---|
| `api_mode` | string | ja | — | muss `"atomic_single_post"` sein |
| `invoice_header` | Objekt | ja | — | Rechnungskopf (s. u.) |
| `validation_totals` | Objekt | ja | — | Kontrollsummen aus dem Quellsystem |
| `seller_snapshot` | Objekt | bedingt* | — | Verkäufer-Stammdaten |
| `mandant_id` | integer | bedingt* | — | alternativ: Mandant referenzieren |
| `buyer` | Objekt | ja | — | Käufer/Empfänger |
| `shipping` | Objekt | nein | — | abweichende Lieferadresse |
| `items` | Array | ja | — | min. 1 Position |
| `attachments` | Array | nein | `[]` | bis 10 Anhänge, max. 15 MB gesamt |
| `extended_fields` | Objekt | nein | `{}` | optionale EN-16931-Zusatzfelder |

\* **Genau eines** von `seller_snapshot` oder `mandant_id` ist anzugeben. Bei
`mandant_id` werden die Verkäuferdaten automatisch aus dem Mandanten gesetzt.

## Rechnungskopf — `invoice_header`

| Feld | Typ | Pflicht | Default | BT | Bemerkung |
|---|---|---|---|---|---|
| `invoice_number` | string(50) | ja | — | BT-1 | Rechnungsnummer |
| `invoice_type` | string(10) | nein | `"380"` | BT-3 | 380 Rechnung, 384 Korrektur, 386 Abschlag, 389 Gutschrift/Self-Billing |
| `invoice_date` | date | ja | — | BT-2 | Rechnungsdatum |
| `delivery_date` | date | nein | — | BT-72 | Liefer-/Leistungsdatum (erscheint im PDF als „Lieferdatum") |
| `billing_start_date` | date | nein | — | BT-73 | Abrechnungszeitraum Beginn |
| `billing_end_date` | date | nein | — | BT-74 | Ende (≥ Beginn) |
| `currency` | string(3) | nein | `"EUR"` | BT-5 | ISO 4217 |
| `order_reference` | string | nein | `""` | BT-13 | Bestellnummer |
| `buyer_reference` | string | nein | `""` | BT-10 | Käufer-Referenz (Leitweg-ID bei B2G) |
| `payment_due_date` | date | nein | — | BT-9 | Fälligkeitsdatum |
| `payment_terms` | string | nein | `""` | BT-20 | Zahlungsbedingungen (Freitext) |
| `payment_means_code` | string(4) | nein | `"58"` | BT-81 | UNCL 4461 (30 Überweisung, 58 SEPA, 49 Lastschrift …) |
| `prepaid_amount` | decimal | nein | — | BT-113 | Bereits gezahlter Betrag (≤ Brutto); BT-115 = Brutto − BT-113 |
| `tax_point_date` | date | nein | — | BT-7 | Steuerstichtag (alternativ zu BT-8) |
| `tax_point_date_code` | string(4) | nein | `""` | BT-8 | UNTDID 2475 (5/29/72) |
| `contract_reference` | string | nein | `""` | BT-12 | Vertragsnummer |
| `project_reference` | string | nein | `""` | BT-11 | Projektreferenz |
| `accounting_cost_code` | string | nein | `""` | BT-19 | Kostenstelle des Käufers |
| `customization_id` | string | nein | `""` | BT-24 | CustomizationID überschreiben (leer = XRechnung-Default) |
| `notes` | string | nein | `""` | BT-22 | Rechnungshinweis |
| `note_subject_code` | string(10) | nein | `""` | BT-21 | Hinweis-Code (UNTDID 4451) |
| `sales_order_reference` | string(100) | nein | `""` | BT-14 | Auftragsnummer des Verkäufers |
| `receiving_advice_reference` | string(100) | nein | `""` | BT-15 | Wareneingangsmeldung |
| `tender_lot_reference` | string(100) | nein | `""` | BT-17 | Vergabe-/Los-Referenz |
| `invoiced_object_id` | string(100) | nein | `""` | BT-18 | Objektkennung |
| `remittance_information` | string(140) | nein | `""` | BT-83 | Verwendungszweck (SEPA: ≤140) |
| `payment_means_text` | string(255) | nein | `""` | BT-82 | Zahlungsart (Freitext) |
| `payment_account_name` | string(255) | nein | `""` | BT-85 | Kontoinhaber |
| `payee_name` | string(255) | nein | `""` | BT-59 | abweichender Zahlungsempfänger (BG-10) |
| `payee_identifier` | string(64) | nein | `""` | BT-60 | Kennung des Zahlungsempfängers |
| `payee_legal_id` | string(64) | nein | `""` | BT-61 | Registernummer des Zahlungsempfängers |
| `card_pan` | string(32) | nein | `""` | BT-87 | nur letzte 4–6 Stellen |
| `card_holder` | string(255) | nein | `""` | BT-88 | Karteninhaber |
| `mandate_reference` | string(64) | nein | `""` | BT-89 | SEPA-Mandatsreferenz |
| `creditor_reference` | string(64) | nein | `""` | BT-90 | Gläubiger-ID |
| `debited_account` | string(34) | nein | `""` | BT-91 | IBAN des belasteten Kontos |
| `parent_invoice` | integer | nein | — | BG-3 | Bezugsrechnung (Korrektur/Gutschrift, BT-25/26) |
| `vat_accounting_currency` | string(3) | nein | `""` | BT-6 | abweichende USt-Währung |
| `tax_amount_accounting` | decimal | nein | — | BT-111 | USt in Buchungswährung (Pflicht, wenn BT-6 gesetzt) |
| `rounding_amount` | decimal | nein | — | BT-114 | Rundungsbetrag |

## Kontrollsummen — `validation_totals`

| Feld | Typ | Pflicht | Bemerkung |
|---|---|---|---|
| `net_amount` | decimal | ja | Summe der Netto-Positionsbeträge |
| `vat_amount` | decimal | ja | Gesamte USt |
| `gross_amount` | decimal | ja | Brutto (Netto + USt) |

Der Server berechnet die Summen aus den Positionen neu und vergleicht sie
mit diesen Werten. **Toleranz: 0,02 € je Feld** (netto, USt., brutto), absolut
gerechnet. Darüber wird die Anfrage mit **HTTP 400** abgelehnt; die Meldung
nennt den berechneten und den gesendeten Wert.

Innerhalb der Toleranz wird der Beleg angenommen, und es gelten die **vom
Server berechneten** Summen — nicht die gesendeten. Weichen beide voneinander
ab, weist die Antwort das in `meta.totals_adjusted` aus, damit die Differenz
nachvollziehbar dokumentiert werden kann:

```json
"meta": {
  "totals_adjusted": {
    "tolerance": "0.02",
    "fields": {
      "vat_amount":   { "sent": "0.47", "used": "0.48", "delta": "0.01" },
      "gross_amount": { "sent": "2.97", "used": "2.98", "delta": "0.01" }
    }
  }
}
```

`delta` ist `used - sent`. Stimmen die Summen auf den Cent überein, fehlt der
Schlüssel `totals_adjusted` ganz. Der häufigste Grund für eine Differenz ist
eine andere Rundungsregel im Quellsystem: Factora rundet kaufmännisch je
Position auf zwei Nachkommastellen und summiert danach.

## Verkäufer — `seller_snapshot`

| Feld | Typ | Pflicht | Default | BT | Bemerkung |
|---|---|---|---|---|---|
| `name` | string(255) | ja | — | BT-27 | Firmenname |
| `street` | string(255) | ja | — | BT-31 | Straße |
| `zip` | string(20) | ja | — | BT-34 | PLZ |
| `city` | string(255) | ja | — | BT-35 | Ort |
| `country` | string(2) | nein | `"DE"` | BT-40 | ISO 3166-1 |
| `country_subdivision` | string(100) | nein | `""` | BT-39 | Bundesland/Region |
| `address_line2` | string(255) | nein | `""` | BT-36 | Adresszusatz |
| `address_line3` | string(255) | nein | `""` | BT-162 | Adresszusatz |
| `vat_id` | string | nein | `""` | BT-31 | USt-IdNr. |
| `tax_number` | string | nein | `""` | — | Steuernummer |
| `gln` | string | nein | `""` | — | GLN (13 Stellen, GS1-Prüfziffer) |
| `identifier` | string(64) | nein | `""` | BT-29 | weitere Verkäuferkennung |
| `trading_name` | string(255) | nein | `""` | BT-28 | Handelsname |
| `legal_registration_id` | string(64) | nein | `""` | BT-30 | Handelsregisternummer |
| `additional_legal_info` | string(255) | nein | `""` | BT-33 | rechtliche Zusatzinfo |
| `iban` | string | nein | `""` | BT-84 | Zahlungs-IBAN |
| `bic` | string | nein | `""` | — | BIC/SWIFT |
| `contact` | Objekt | nein | — | BG-6 | Ansprechpartner (s. u.) |
| `tax_representative` | Objekt | nein | — | BG-11/12 | steuerlicher Vertreter (s. u.) |

### Ansprechpartner — `contact`

| Feld | Typ | Pflicht | Bemerkung |
|---|---|---|---|
| `name` | string(255) | nein | Kontaktperson |
| `phone` | string(50) | nein | Telefon |
| `email` | string(255) | nein | E-Mail |

### Steuerlicher Vertreter — `tax_representative` (BG-11/12)

Alle Felder optional; wird `name` gesetzt, werden `vat_id` und `country`
zu Pflichtangaben.

| Feld | Typ | BT | Bemerkung |
|---|---|---|---|
| `name` | string(255) | BT-62 | Name des Vertreters |
| `vat_id` | string(64) | BT-63 | USt-IdNr. des Vertreters |
| `street` | string(255) | BT-64 | Straße |
| `address_line2` | string(255) | BT-65 | Adresszusatz |
| `address_line3` | string(255) | BT-164 | Adresszusatz |
| `zip` | string(20) | BT-67 | PLZ |
| `city` | string(255) | BT-66 | Ort |
| `country_subdivision` | string(100) | BT-68 | Region |
| `country` | string(2) | BT-69 | Land |

## Käufer — `buyer`

| Feld | Typ | Pflicht | Default | BT | Bemerkung |
|---|---|---|---|---|---|
| `name` | string(255) | ja | — | BT-44 | Firmenname |
| `street` | string(255) | ja | — | BT-49 | Straße |
| `zip` | string(20) | ja | — | BT-50 | PLZ |
| `city` | string(255) | ja | — | BT-50 | Ort |
| `country` | string(2) | nein | `"DE"` | BT-55 | ISO 3166-1 |
| `country_subdivision` | string(100) | nein | `""` | BT-54 | Bundesland/Region |
| `address_line2` | string(255) | nein | `""` | BT-51 | Adresszusatz |
| `address_line3` | string(255) | nein | `""` | BT-163 | Adresszusatz |
| `vat_id` | string | nein | `""` | BT-48 | USt-IdNr. (Pflicht bei AE/K) |
| `gln` | string | nein | `""` | BT-44 | GLN (13 Stellen, GS1-Prüfziffer) |
| `identifier` | string(64) | nein | `""` | BT-46 | weitere Käuferkennung |
| `debitor_number` | string(20) | nein | `""` | — | DATEV-Debitorennummer (nur Export, nicht im XML) |
| `customer_number` | string(50) | nein | `""` | — | Ihre eigene Kundennummer (siehe unten; nur PDF, nicht im XML) |
| `branch_id` | string | nein | `""` | — | Filial-/Standortkennung |
| `trading_name` | string(255) | nein | `""` | BT-45 | Handelsname |
| `legal_registration_id` | string(64) | nein | `""` | BT-47 | Handelsregisternummer |
| `contact` | Objekt | nein | — | BG-9 | Ansprechpartner (wie oben) |

### Kundennummer — `buyer.customer_number`

Die Nummer, unter der der Käufer in **Ihrem** System geführt wird. Sie
erscheint auf dem PDF als „Kundennummer" und steht nicht im XML (EN 16931
kennt dafür kein Feld — für eine Käuferkennung im XML nutzen Sie `gln`
(BT-44) oder `identifier` (BT-46)).

Die Nummer ist zugleich die **Identität des Kunden**: Rechnungen mit
derselben Kundennummer landen auf demselben Kundendatensatz, auch wenn
sich der Firmenname zwischenzeitlich geändert hat. Umgekehrt gilt: ändern
Sie die Nummer eines bestehenden Kunden, übernehmen wir das — bereits
ausgestellte Rechnungen behalten die Nummer, die zum Zeitpunkt der
Ausstellung galt.

Eindeutig ist sie **je Mandant**: zwei Mandanten dürfen dieselbe Nummer
vergeben, innerhalb eines Mandanten bezeichnet sie genau einen Kunden.

Lassen Sie das Feld weg, vergibt Factora weiterhin selbst eine Nummer aus
dem Firmennamen (`API-…`). Schicken Sie die Nummer für einen Käufer, den
wir bereits unter einer solchen `API-…`-Nummer führen, ersetzt Ihre Nummer
die generierte — es entsteht kein zweiter Datensatz.

## Lieferadresse — `shipping` (BG-13/15)

Optional; fehlt sie, gilt die Käuferadresse. Wird sie angegeben, sind
`name`, `street`, `zip`, `city` Pflicht.

| Feld | Typ | Pflicht | Default | BT |
|---|---|---|---|---|
| `name` | string(255) | ja | — | BT-71 |
| `street` | string(255) | ja | — | BT-73 |
| `zip` | string(20) | ja | — | BT-74 |
| `city` | string(255) | ja | — | BT-75 |
| `country` | string(2) | nein | `"DE"` | BT-80 |
| `country_subdivision` | string(100) | nein | `""` | BT-79 |
| `address_line2` | string(255) | nein | `""` | BT-76 |
| `address_line3` | string(255) | nein | `""` | BT-165 |
| `gln` | string | nein | `""` | — |

## Positionen — `items[]`

Pflichtfelder je Position: `description`, `quantity`, `unit`,
`unit_price_net`, `vat_rate`.

| Feld | Typ | Pflicht | Default | BT | Bemerkung |
|---|---|---|---|---|---|
| `description` | string(500) | ja | — | BT-154 | Bezeichnung |
| `quantity` | decimal(…,3) | ja | — | BT-129 | Menge |
| `unit` | string(10) | ja | — | BT-130 | Einheit (UNCL 5272: HUR Stunde, KGM kg, C62 Stück …) |
| `unit_price_net` | decimal | ja | — | BT-146 | Nettoeinzelpreis |
| `vat_rate` | decimal(5,2) | ja | — | BT-152 | Steuersatz in % (19.00 / 7.00 / 0.00) |
| `tax_category_code` | string | nein | `""` | BT-151 | Kategorie (S/AA/Z/E/AE/K/G/O) — siehe [Steuerlogik](tax-logic.md) |
| `tax_exemption_reason_code` | string | nein | `""` | BT-121 | VATEX-Code (Pflicht bei Befreiung) |
| `tax_exemption_reason_text` | string | nein | `""` | BT-120 | Befreiungstext |
| `sku` | string(50) | nein | `""` | BT-155 | Artikelnummer Verkäufer |
| `gtin` | string | nein | `""` | BT-157 | GTIN (13/14 Stellen, GS1-Prüfziffer) |
| `buyer_item_id` | string(100) | nein | `""` | BT-156 | Artikelnummer Käufer |
| `discount_percent` | decimal(5,2) | nein | — | BT-139 | Rabatt in % |
| `discount_amount` | decimal | nein | — | BT-138 | Rabatt absolut |
| `discount_base_amount` | decimal | nein | — | BT-137 | Rabatt-Basis |
| `discount_reason` | string | nein | `""` | — | Rabattgrund (Freitext) |
| `discount_reason_code` | string(16) | nein | `""` | BT-140 | Rabattgrund-Code (UNTDID 5189) |
| `charge_amount` | decimal | nein | — | BT-141 | Zuschlag absolut (BG-28) |
| `charge_base_amount` | decimal | nein | — | BT-142 | Zuschlag-Basis |
| `charge_percent` | decimal(5,2) | nein | — | BT-143 | Zuschlag in % |
| `charge_reason` | string(255) | nein | `""` | BT-144 | Zuschlagsgrund |
| `charge_reason_code` | string(16) | nein | `""` | BT-145 | Zuschlagsgrund-Code (UNTDID 7161) |
| `period_start` | date | nein | — | BT-134 | Leistungszeitraum Beginn |
| `period_end` | date | nein | — | BT-135 | Leistungszeitraum Ende |
| `line_note` | string(1000) | nein | `""` | BT-127 | Positionshinweis |
| `object_identifier` | string(200) | nein | `""` | BT-128 | Objektkennung |
| `order_line_reference` | string(50) | nein | `""` | BT-132 | Bestellpositions-Referenz |
| `line_accounting_cost` | string(200) | nein | `""` | BT-133 | Kostenstelle (Position) |
| `item_description` | string(1000) | nein | `""` | BT-154 | erweiterte Beschreibung |
| `classification_code` | string(100) | nein | `""` | BT-158 | Klassifizierung (z. B. eCl@ss) |
| `classification_scheme` | string(10) | nein | `""` | BT-158-1 | Klassifizierungsschema (UNCL 7143) |
| `country_of_origin` | string(2) | nein | `""` | BT-159 | Ursprungsland |
| `attribute_name` | string(200) | nein | `""` | BT-160 | Merkmalsname |
| `attribute_value` | string(200) | nein | `""` | BT-161 | Merkmalswert |
| `gross_price` | decimal | nein | — | BT-148 | Bruttoeinzelpreis vor Positionsrabatt |
| `price_discount` | decimal | nein | — | BT-147 | Preisnachlass (Brutto − Netto) |
| `price_base_quantity` | decimal(…,3) | nein | — | BT-149 | Preisbasis-Menge |
| `price_base_quantity_unit` | string(3) | nein | `""` | BT-150 | Einheit der Preisbasis |

## Anhänge — `attachments[]`

Bis zu **10 eingebettete Anhänge**, **max. 15 MB** je Datei und gesamt. Pro
Anhang ist **entweder** `content_base64` **oder** `external_url` anzugeben.
Der MIME-Typ wird gegen eine Whitelist geprüft und der Dateiinhalt
zusätzlich anhand der „Magic Bytes" verifiziert.

**Erlaubte MIME-Typen:** PDF, PNG, JPEG, CSV, ODS (OpenDocument), XLSX, XLS.

| Feld | Typ | Pflicht | Default | BT | Bemerkung |
|---|---|---|---|---|---|
| `filename` | string(255) | ja | — | — | Dateiname |
| `mime_type` | string(100) | ja | — | BT-125-1 | aus Whitelist |
| `type` | string | nein | `"OTHER"` | — | `INVOICE_IMAGE` / `DELIVERY_NOTE` / `OTHER` |
| `content_base64` | string | bedingt | `""` | BT-125 | eingebettete Datei (XOR `external_url`) |
| `external_url` | string(500) | bedingt | `""` | BT-124 | externer Link (XOR `content_base64`) |
| `document_reference` | string(200) | nein | `""` | BT-122 | Dokumentkennung |
| `description` | string(255) | nein | `""` | BT-123 | Beschreibung |
| `delivery_note_number` | string | nein | `""` | — | bei `type=DELIVERY_NOTE` |

## Zusatzfelder — `extended_fields`

Frei verwendbares Objekt für weitere EN-16931-Felder. Werte aus dem Aufruf
überschreiben etwaige am API-Schlüssel hinterlegte Voreinstellungen.

## Antwort

**Erfolg (Live):** HTTP `201`. **Sandbox:** HTTP `200` mit
`meta.sandbox = true`.

```json
{
  "valid": true,
  "data": {
    "id": 12345,
    "invoice_number": "RE-2026-001",
    "status": "final",
    "total": "1190.00",
    "pdf_base64": "JVBERi0xLjQK…",
    "xml_base64": "PD94bWwgdmVy…"
  },
  "errors": [],
  "meta": {}
}
```

| Feld | Typ | Bemerkung |
|---|---|---|
| `data.id` | integer | interne Rechnungs-ID |
| `data.invoice_number` | string | übernommene Rechnungsnummer (BT-1) |
| `data.status` | string | `final` (Live) bzw. Sandbox-Status |
| `data.total` | string | Bruttobetrag |
| `data.pdf_base64` | string | ZUGFeRD-PDF/A-3 (Base64) |
| `data.xml_base64` | string | XRechnung-XML (Base64) |

Fehlerfälle und Fehlercodes: siehe [API-Überblick](api-overview.md#fehler--und-antwortstruktur).

## Wichtige Kombinationsregeln

1. Genau eines von `seller_snapshot` / `mandant_id`.
2. Bei befreiten Kategorien (E, AE, K, G, O) Befreiungstext/-code angeben
   (AE/K/G werden automatisch gefüllt).
3. AE/K erfordern eine Käufer-USt-IdNr.; AE nur für B2B/B2G.
4. `validation_totals` müssen zur Positionsberechnung passen.
5. GLN/GTIN, falls angegeben, müssen gültige GS1-Prüfziffern haben.
6. Anhänge: `content_base64` XOR `external_url`.
7. `prepaid_amount` (BT-113) muss ≤ Bruttobetrag sein, sonst 400.

## Bereits beglichene Belege

Läuft die Zahlung außerhalb des Belegs (z. B. Plattform-Abrechnung,
Gutschriftverfahren nach §14 Abs. 2 UStG), ist bei Erstellung nichts mehr
fällig. Empfohlene Kombination:

- `invoice_header.prepaid_amount` = Bruttobetrag → XML trägt BT-113 und
  BT-115 = `0.00` (kein offener Posten beim Empfänger-Import); das PDF
  zeigt „Bereits gezahlt" in der Summenübersicht und ersetzt Bankblock,
  Zahlungs-QR und Fälligkeitszeile durch einen Beglichen-Hinweis.
- `payment_means_code` **nicht** `58`/`59` setzen — z. B. `1` (Instrument
  not defined), `68` (Online payment service) oder `97` (Clearing between
  partners). BR-DE-23-a erzwingt nur bei `58` eine IBAN (BG-17).
- `iban`/`bic` im `seller_snapshot` weglassen, wenn kein Konto auf dem
  Beleg erscheinen soll (Achtung: Tenant-Stammdaten füllen Snapshot-Lücken —
  BG-17 entfällt nur, wenn beide keine IBAN tragen).
- Optional `payment_terms` mit eigenem Hinweistext (BT-20), sonst rendert
  das PDF den Standard-Beglichen-Hinweis.
