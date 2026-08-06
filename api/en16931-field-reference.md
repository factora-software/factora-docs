# Factora — EN 16931 Feld-Referenz

_Alle 164 unterstützten Business Terms · XRechnung (CII) + Peppol (UBL)_

Alle **164 Geschäftsfelder (Business Terms)** der EN 16931, die Factora unterstützt. Jedes Feld wird sowohl in **XRechnung (CII)** als auch in **Peppol BIS 3.0 (UBL)** ausgegeben und ist gegen den KoSIT-Prüfstandard getestet.

Die Spalte **Pfad im Request** zeigt, an welcher Stelle das Feld im JSON-Body von `POST /api/v1/invoices/atomic/` gesetzt wird (Kopfdaten unter `invoice_header.*`). `(automatisch berechnet)` = von Factora aus den Positionen ermittelt (z. B. Summen). `—` = wird von Factora gesetzt oder ist (noch) nicht als Einzelfeld der atomic-API verfügbar — etwa die Dokument-Nachlässe und -Zuschläge (BG-20/21), die XML-seitig unterstützt, aber nur über die App bzw. das Datenmodell pflegbar sind.

> Automatisch generiert aus apps/invoices/standards/en16931.py · Stand: 2026-05-22. Nicht von Hand bearbeiten — bei Änderungen docs/api/build_field_reference.py neu ausführen.

## Dokumentenkopf

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-1 | Rechnungsnummer | Invoice number | invoice_header.invoice_number |
| BT-2 | Rechnungsdatum | Invoice issue date | invoice_header.invoice_date |
| BT-3 | Rechnungsart (Code) | Invoice type code | invoice_header.invoice_type |
| BT-5 | Währung | Invoice currency code | invoice_header.currency |
| BT-6 | Steuer-Buchungswährung | VAT accounting currency code | invoice_header.vat_accounting_currency |
| BT-7 | Steuerstichtag (Leistungsdatum) | Value added tax point date | invoice_header.tax_point_date |
| BT-8 | Steuerstichtag-Code | VAT point date code | invoice_header.tax_point_date_code |
| BT-9 | Fälligkeitsdatum | Payment due date | invoice_header.payment_due_date |
| BT-10 | Käuferreferenz (Leitweg-ID) | Buyer reference | invoice_header.buyer_reference |
| BT-11 | Projektreferenz | Project reference | invoice_header.project_reference |
| BT-12 | Vertragsreferenz | Contract reference | invoice_header.contract_reference |
| BT-13 | Bestellnummer | Purchase order reference | invoice_header.order_reference |
| BT-14 | Auftragsnummer (Verkäufer) | Sales order reference | invoice_header.sales_order_reference |
| BT-15 | Wareneingangsmeldungs-Referenz | Receiving advice reference | invoice_header.receiving_advice_reference |
| BT-16 | Lieferavis-Referenz | Despatch advice reference | invoice_header.delivery_note_id |
| BT-17 | Ausschreibungs-/Losreferenz | Tender or lot reference | invoice_header.tender_lot_reference |
| BT-18 | Objektkennung | Invoiced object identifier | invoice_header.invoiced_object_id |
| BT-19 | Buchungsreferenz des Käufers | Buyer accounting reference | invoice_header.accounting_cost_code |
| BT-20 | Zahlungsbedingungen | Payment terms | invoice_header.payment_terms |

## BG-1 — Rechnungshinweise

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-21 | Hinweis-Betreffcode | Invoice note subject code | invoice_header.note_subject_code |
| BT-22 | Rechnungshinweis (Freitext) | Invoice note | invoice_header.notes |

## BG-2 — Prozesssteuerung

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-23 | Geschäftsprozess-Typ | Business process type | (automatisch berechnet) |
| BT-24 | Spezifikationskennung | Specification identifier | invoice_header.customization_id |

## BG-3 — Vorausgehende Rechnung

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-25 | Nummer der Vorrechnung | Preceding invoice number | invoice_header.parent_invoice |
| BT-26 | Datum der Vorrechnung | Preceding invoice issue date | invoice_header.parent_invoice |

## BG-4 — Verkäufer

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-27 | Name des Verkäufers | Seller name | seller_snapshot.name |
| BT-28 | Handelsname des Verkäufers | Seller trading name | seller_snapshot.trading_name |
| BT-29 | Kennung des Verkäufers | Seller identifier | seller_snapshot.identifier |
| BT-30 | Registernummer des Verkäufers (HRB) | Seller legal registration id | seller_snapshot.legal_registration_id |
| BT-31 | USt-IdNr. des Verkäufers | Seller VAT identifier | seller_snapshot.vat_id |
| BT-32 | Steuernummer des Verkäufers | Seller tax registration id | seller_snapshot.tax_number |
| BT-33 | Zusätzliche rechtliche Angaben (Verkäufer) | Seller additional legal info | seller_snapshot.additional_legal_info |
| BT-34 | Elektronische Adresse des Verkäufers | Seller electronic address | — |

## BG-5 — Verkäufer-Anschrift

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-35 | Verkäufer Adresszeile 1 | Seller address line 1 | seller_snapshot.street |
| BT-36 | Verkäufer Adresszeile 2 | Seller address line 2 | seller_snapshot.address_line2 |
| BT-162 | Verkäufer Adresszeile 3 | Seller address line 3 | seller_snapshot.address_line3 |
| BT-37 | Verkäufer Ort | Seller city | seller_snapshot.city |
| BT-38 | Verkäufer PLZ | Seller post code | seller_snapshot.zip |
| BT-39 | Verkäufer Region/Bundesland | Seller country subdivision | seller_snapshot.country_subdivision |
| BT-40 | Verkäufer Land | Seller country code | seller_snapshot.country |

## BG-6 — Verkäufer-Kontakt

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-41 | Verkäufer Ansprechpartner | Seller contact point | seller_snapshot.contact.name |
| BT-42 | Verkäufer Telefon | Seller contact telephone | seller_snapshot.contact.phone |
| BT-43 | Verkäufer E-Mail | Seller contact email | seller_snapshot.contact.email |

## BG-7 — Käufer

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-44 | Name des Käufers | Buyer name | buyer.name |
| BT-45 | Handelsname des Käufers | Buyer trading name | buyer.trading_name |
| BT-46 | Kennung des Käufers | Buyer identifier | buyer.identifier |
| BT-47 | Registernummer des Käufers | Buyer legal registration id | buyer.legal_registration_id |
| BT-48 | USt-IdNr. des Käufers | Buyer VAT identifier | buyer.vat_id |
| BT-49 | Elektronische Adresse des Käufers | Buyer electronic address | buyer.contact.email |

## BG-8 — Käufer-Anschrift

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-50 | Käufer Adresszeile 1 | Buyer address line 1 | buyer.street |
| BT-51 | Käufer Adresszeile 2 | Buyer address line 2 | buyer.address_line2 |
| BT-163 | Käufer Adresszeile 3 | Buyer address line 3 | buyer.address_line3 |
| BT-52 | Käufer Ort | Buyer city | buyer.city |
| BT-53 | Käufer PLZ | Buyer post code | buyer.zip |
| BT-54 | Käufer Region/Bundesland | Buyer country subdivision | buyer.country_subdivision |
| BT-55 | Käufer Land | Buyer country code | buyer.country |

## BG-9 — Käufer-Kontakt

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-56 | Käufer Ansprechpartner | Buyer contact point | buyer.contact.name |
| BT-57 | Käufer Telefon | Buyer contact telephone | buyer.contact.phone |
| BT-58 | Käufer E-Mail | Buyer contact email | buyer.contact.email |

## BG-10 — Zahlungsempfänger

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-59 | Name des Zahlungsempfängers | Payee name | invoice_header.payee_name |
| BT-60 | Kennung des Zahlungsempfängers | Payee identifier | invoice_header.payee_identifier |
| BT-61 | Registernummer des Zahlungsempfängers | Payee legal registration id | invoice_header.payee_legal_id |

## BG-11 — Steuervertreter

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-62 | Name des Steuervertreters | Tax representative name | seller_snapshot.tax_representative.name |
| BT-63 | USt-IdNr. des Steuervertreters | Tax representative VAT id | seller_snapshot.tax_representative.vat_id |

## BG-12 — Steuervertreter-Anschrift

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-64 | Steuervertreter Adresszeile 1 | Tax rep. address line 1 | seller_snapshot.tax_representative.street |
| BT-65 | Steuervertreter Adresszeile 2 | Tax rep. address line 2 | seller_snapshot.tax_representative.address_line2 |
| BT-164 | Steuervertreter Adresszeile 3 | Tax rep. address line 3 | seller_snapshot.tax_representative.address_line3 |
| BT-66 | Steuervertreter Ort | Tax rep. city | seller_snapshot.tax_representative.city |
| BT-67 | Steuervertreter PLZ | Tax rep. post code | seller_snapshot.tax_representative.zip |
| BT-68 | Steuervertreter Region | Tax rep. country subdivision | seller_snapshot.tax_representative.country_subdivision |
| BT-69 | Steuervertreter Land | Tax rep. country code | seller_snapshot.tax_representative.country |

## BG-13 — Lieferinformationen

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-70 | Name des Warenempfängers | Deliver to party name | shipping.name |
| BT-71 | Standortkennung der Lieferung (GLN) | Deliver to location identifier | shipping.gln |
| BT-72 | Lieferdatum | Actual delivery date | invoice_header.delivery_date |

## BG-14 — Abrechnungszeitraum

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-73 | Abrechnungszeitraum Beginn | Invoicing period start | invoice_header.billing_start_date |
| BT-74 | Abrechnungszeitraum Ende | Invoicing period end | invoice_header.billing_end_date |

## BG-15 — Lieferanschrift

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-75 | Lieferadresse Zeile 1 | Deliver to address line 1 | shipping.street |
| BT-76 | Lieferadresse Zeile 2 | Deliver to address line 2 | shipping.address_line2 |
| BT-165 | Lieferadresse Zeile 3 | Deliver to address line 3 | shipping.address_line3 |
| BT-77 | Lieferadresse Ort | Deliver to city | shipping.city |
| BT-78 | Lieferadresse PLZ | Deliver to post code | shipping.zip |
| BT-79 | Lieferadresse Region | Deliver to country subdivision | shipping.country_subdivision |
| BT-80 | Lieferadresse Land | Deliver to country code | shipping.country |

## BG-16 — Zahlungsanweisungen

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-81 | Zahlungsart (Code) | Payment means type code | invoice_header.payment_means_code |
| BT-82 | Zahlungsart (Text) | Payment means text | invoice_header.payment_means_text |
| BT-83 | Verwendungszweck | Remittance information | invoice_header.remittance_information |

## BG-17 — Überweisung

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-84 | IBAN | Payment account identifier (IBAN) | seller_snapshot.iban |
| BT-85 | Kontoinhaber | Payment account name | invoice_header.payment_account_name |
| BT-86 | BIC | Payment service provider id (BIC) | seller_snapshot.bic |

## BG-18 — Kartenzahlung

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-87 | Kartennummer | Card primary account number | invoice_header.card_pan |
| BT-88 | Karteninhaber | Card holder name | invoice_header.card_holder |

## BG-19 — SEPA-Lastschrift

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-89 | SEPA-Mandatsreferenz | Mandate reference identifier | invoice_header.mandate_reference |
| BT-90 | Gläubiger-ID (SEPA) | Bank assigned creditor identifier | invoice_header.creditor_reference |
| BT-91 | Belastetes Konto (IBAN) | Debited account identifier | invoice_header.debited_account |

## BG-20 — Nachlässe (Dokument)

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-92 | Nachlass-Betrag | Allowance amount | — |
| BT-93 | Nachlass-Basisbetrag | Allowance base amount | — |
| BT-94 | Nachlass-Prozentsatz | Allowance percentage | — |
| BT-95 | Nachlass USt-Kategorie | Allowance VAT category code | — |
| BT-96 | Nachlass USt-Satz | Allowance VAT rate | — |
| BT-97 | Nachlass-Grund | Allowance reason | — |
| BT-98 | Nachlass-Grund (Code) | Allowance reason code | — |

## BG-21 — Zuschläge (Dokument)

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-99 | Zuschlag-Betrag | Charge amount | — |
| BT-100 | Zuschlag-Basisbetrag | Charge base amount | — |
| BT-101 | Zuschlag-Prozentsatz | Charge percentage | — |
| BT-102 | Zuschlag USt-Kategorie | Charge VAT category code | — |
| BT-103 | Zuschlag USt-Satz | Charge VAT rate | — |
| BT-104 | Zuschlag-Grund | Charge reason | — |
| BT-105 | Zuschlag-Grund (Code) | Charge reason code | — |

## BG-22 — Summen

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-106 | Summe der Positionsnettobeträge | Sum of line net amounts | (automatisch berechnet) |
| BT-107 | Summe Nachlässe (Dokument) | Sum of allowances on document | (automatisch berechnet) |
| BT-108 | Summe Zuschläge (Dokument) | Sum of charges on document | (automatisch berechnet) |
| BT-109 | Gesamtbetrag netto | Invoice total without VAT | (automatisch berechnet) |
| BT-110 | Gesamtbetrag USt | Invoice total VAT amount | (automatisch berechnet) |
| BT-111 | USt-Gesamtbetrag (Buchungswährung) | Invoice total VAT in accounting currency | invoice_header.tax_amount_accounting |
| BT-112 | Gesamtbetrag brutto | Invoice total with VAT | (automatisch berechnet) |
| BT-113 | Bereits gezahlter Betrag | Paid amount | invoice_header.prepaid_amount (optional; sonst automatisch aus Abschlags-/Kumulativ-Kette) |
| BT-114 | Rundungsbetrag | Rounding amount | invoice_header.rounding_amount |
| BT-115 | Zahlbetrag | Amount due for payment | (automatisch berechnet: BT-112 − BT-113 + BT-114) |

## BG-23 — USt-Aufschlüsselung

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-116 | USt-Basisbetrag (Kategorie) | VAT category taxable amount | (automatisch berechnet) |
| BT-117 | USt-Betrag (Kategorie) | VAT category tax amount | (automatisch berechnet) |
| BT-118 | USt-Kategorie | VAT category code | (automatisch berechnet) |
| BT-119 | USt-Satz (Kategorie) | VAT category rate | (automatisch berechnet) |
| BT-120 | USt-Befreiungsgrund (Text) | VAT exemption reason text | items[].tax_exemption_reason_text |
| BT-121 | USt-Befreiungsgrund (Code) | VAT exemption reason code | items[].tax_exemption_reason_code |

## BG-24 — Anhänge

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-122 | Anhang-Referenz | Supporting document reference | attachments[].document_reference |
| BT-123 | Anhang-Beschreibung | Supporting document description | attachments[].description |
| BT-124 | Externer Anhang-Link | External document location | attachments[].external_url |
| BT-125 | Eingebetteter Anhang (Datei) | Attached document (binary) | attachments[].content_base64 |

## BG-25 — Rechnungsposition

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-126 | Positionsnummer | Invoice line identifier | — |
| BT-127 | Positionshinweis | Invoice line note | items[].line_note |
| BT-128 | Positions-Objektkennung | Invoice line object identifier | items[].object_identifier |
| BT-129 | Menge | Invoiced quantity | items[].quantity |
| BT-130 | Mengeneinheit | Invoiced quantity unit of measure | items[].unit |
| BT-131 | Positionsnettobetrag | Invoice line net amount | — |
| BT-132 | Referenzierte Bestellposition | Referenced purchase order line | items[].order_line_reference |
| BT-133 | Buchungsreferenz (Position) | Invoice line buyer accounting ref | items[].line_accounting_cost |

## BG-26 — Positions-Leistungszeitraum

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-134 | Positions-Leistungszeitraum Beginn | Invoice line period start | items[].period_start |
| BT-135 | Positions-Leistungszeitraum Ende | Invoice line period end | items[].period_end |

## BG-27 — Positions-Nachlässe

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-136 | Positions-Nachlass Betrag | Line allowance amount | items[].discount_amount |
| BT-137 | Positions-Nachlass Basisbetrag | Line allowance base amount | items[].discount_base_amount |
| BT-138 | Positions-Nachlass Prozentsatz | Line allowance percentage | items[].discount_percent |
| BT-139 | Positions-Nachlass Grund | Line allowance reason | items[].discount_reason |
| BT-140 | Positions-Nachlass Grund (Code) | Line allowance reason code | items[].discount_reason_code |

## BG-28 — Positions-Zuschläge

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-141 | Positions-Zuschlag Betrag | Line charge amount | items[].charge_amount |
| BT-142 | Positions-Zuschlag Basisbetrag | Line charge base amount | items[].charge_base_amount |
| BT-143 | Positions-Zuschlag Prozentsatz | Line charge percentage | items[].charge_percent |
| BT-144 | Positions-Zuschlag Grund | Line charge reason | items[].charge_reason |
| BT-145 | Positions-Zuschlag Grund (Code) | Line charge reason code | items[].charge_reason_code |

## BG-29 — Preisdetails

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-146 | Nettoeinzelpreis | Item net price | items[].unit_price_net |
| BT-147 | Einzelpreis-Rabatt | Item price discount | items[].price_discount |
| BT-148 | Bruttoeinzelpreis | Item gross price | items[].gross_price |
| BT-149 | Preisbasismenge | Item price base quantity | items[].price_base_quantity |
| BT-150 | Preisbasismenge-Einheit | Item price base quantity unit | items[].price_base_quantity_unit |

## BG-30 — Positions-USt

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-151 | USt-Kategorie (Position) | Invoiced item VAT category code | items[].tax_category_code |
| BT-152 | USt-Satz (Position) | Invoiced item VAT rate | items[].vat_rate |

## BG-31 — Artikelinformationen

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-153 | Artikelname | Item name | items[].description |
| BT-154 | Artikelbeschreibung | Item description | items[].item_description |
| BT-155 | Artikelnummer (Verkäufer) | Seller's item identifier | items[].sku |
| BT-156 | Artikelnummer (Käufer) | Buyer's item identifier | items[].buyer_item_id |
| BT-157 | GTIN/EAN | Standard item identifier (GTIN) | items[].gtin |
| BT-158 | Artikelklassifizierung | Item classification identifier | items[].classification_code |
| BT-159 | Ursprungsland | Item country of origin | items[].country_of_origin |

## BG-32 — Artikelmerkmale

| BT | Feld (deutsch) | EN 16931 (englisch) | Pfad im Request |
| --- | --- | --- | --- |
| BT-160 | Artikelmerkmal (Name) | Item attribute name | items[].attribute_name |
| BT-161 | Artikelmerkmal (Wert) | Item attribute value | items[].attribute_value |
