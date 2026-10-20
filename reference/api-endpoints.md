# Endpoints

Alle Pfade relativ zur Basis-URL `https://console.factora.software/api/v1`.
Authentifizierung, Antwort-Hülle und Fehlercodes siehe
[API-Überblick](api-overview.md).

## Schema & interaktive Referenz

Die API ist vollständig per **OpenAPI 3.1** beschrieben (Titel „Factora API",
Version 1.0.0). Bei Abweichungen zwischen Prosa und Schema ist das **Schema
maßgeblich**.

| Ressource | Pfad | Zugriff |
|---|---|---|
| OpenAPI-Schema (3.1) | `GET /api/schema/` | öffentlich |
| Swagger-UI | `GET /api/docs/` | öffentlich |
| ReDoc | `GET /api/redoc/` | öffentlich |
| Scalar-Referenz | `https://console.factora.software/docs` | öffentlich |
| Vollschema | `GET /api/schema/full/` · `GET /api/docs/full/` | authentifiziert |

## Vollständige Endpunktliste

| Methode | Pfad | Zweck | Scope | Sandbox |
|---|---|---|---|---|
| POST | `/invoices/atomic/` | Rechnung in einem Aufruf erstellen, finalisieren, PDF+XML zurückgeben | `invoices:write` | ✓ |
| POST | `/invoices/preview/` | Summen live berechnen (ohne Speicherung) | `invoices:read` | ✓ |
| POST | `/invoices/convert/` | EN-16931-JSON → geprüftes XML (ohne Speicherung) | `invoices:write` | ✓ |
| POST | `/e-invoice-obligation/check/` | Prüfen, ob ein Vorgang in DE eine E-Rechnung sein muss (ohne Speicherung, nicht abgerechnet) | `invoices:read` | ✓ |
| POST | `/parties/check/` | Stammdaten einer Partei vorab prüfen: USt-IdNr. (Format + VIES), Leitweg-ID, elektronische Adresse (ohne Speicherung, nicht abgerechnet) | `invoices:read` | ✓ |
| GET | `/invoices/` | Rechnungen auflisten (paginiert, filterbar) | `invoices:read` | — |
| POST | `/invoices/` | Rechnung als Entwurf anlegen | `invoices:write` | — |
| GET | `/invoices/{id}/` | Einzelne Rechnung abrufen | `invoices:read` | — |
| POST | `/invoices/{id}/finalize/` | Entwurf finalisieren (PDF/XML erzeugen) | `invoices:write` | — |
| POST | `/invoices/{id}/send/` | Finalisierte Rechnung per E-Mail versenden | `invoices:write` | — |
| GET | `/invoices/{id}/pdf/` | ZUGFeRD-PDF herunterladen | `invoices:read` | — |
| GET | `/invoices/{id}/xml/` | XRechnung-XML herunterladen | `invoices:read` | — |
| GET | `/invoices/{id}/xml/cii/` | CII/XRechnung-XML — finalisiert: das archivierte XML (= `/xml/`, B-AP91); Entwurf: **entwertete Vorschau**, keine gültige E-Rechnung (BT-24 `FACTORA-DRAFT-INVALID#…`, Notiz „ENTWURF — KEINE GÜLTIGE RECHNUNG“; B-AP131) | `invoices:read` | — |
| GET | `/invoices/{id}/xml/ubl/` | UBL/Peppol-XML eines Entwurfs als **entwertete Vorschau** (wie `/xml/cii/`, B-AP131); finalisiert → `409` (kein UBL-Artefakt archiviert, kein Re-Rendern; B-AP91) | `invoices:read` | — |
| POST | `/invoices/{id}/dispatch/` | Rechnung zum Transport einreihen (E-Mail/Peppol) | `invoices:write` | — |
| GET | `/invoices/{id}/dispatch-log/` | Versandhistorie abrufen | `invoices:read` | — |
| GET | `/customers/` | Kunden auflisten | `customers:read` | — |
| POST | `/customers/` | Kunden anlegen | `customers:write` | — |
| GET | `/mandanten/` | Mandanten auflisten | `mandanten:read` | — |
| POST | `/mandanten/` | Mandant anlegen | `mandanten:write` | — |
| GET | `/mandanten/{id}/` | Mandant abrufen | `mandanten:read` | — |
| PATCH | `/mandanten/{id}/` | Mandant ändern (partiell) | `mandanten:write` | — |
| DELETE | `/mandanten/{id}/` | Mandant löschen (nur ohne Rechnungen/Kunden) | `mandanten:write` | — |
| POST | `/mandanten/bulk/` | Bis zu 200 Mandanten anlegen (alles-oder-nichts) | `mandanten:write` | — |
| POST | `/mandanten/{id}/logo/` | Kopf-Logo hochladen (multipart `logo`) | `mandanten:write` | — |
| DELETE | `/mandanten/{id}/logo/` | Kopf-Logo entfernen | `mandanten:write` | — |
| POST | `/mandanten/{id}/letterhead/` | Briefpapier-PDF hochladen (multipart `letterhead`) | `mandanten:write` | — |
| DELETE | `/mandanten/{id}/letterhead/` | Briefpapier entfernen | `mandanten:write` | — |
| POST | `/mandanten/{id}/footer-logos/` | Fußzeilen-Logo hinzufügen (multipart `image`, `position`) | `mandanten:write` | — |
| PATCH | `/mandanten/{id}/footer-logos/{logo_id}/` | Fußzeilen-Logo verschieben (`position`, `sort_order`) | `mandanten:write` | — |
| DELETE | `/mandanten/{id}/footer-logos/{logo_id}/` | Fußzeilen-Logo entfernen | `mandanten:write` | — |
| GET | `/exports/datev/` | DATEV-Buchungsdaten als CSV | `exports:read` | — |
| GET | `/exports/invoices/` | Rechnungsarchiv (PDF + XML) als ZIP mit Index und Manifest | `exports:read` | — |
| GET | `/webhooks/` | Webhooks auflisten | `webhooks:read` | — |
| POST | `/webhooks/` | Webhook anlegen | `webhooks:write` | — |
| GET | `/webhooks/{id}/` | Webhook abrufen | `webhooks:read` | — |
| PATCH | `/webhooks/{id}/` | Webhook ändern | `webhooks:write` | — |
| DELETE | `/webhooks/{id}/` | Webhook deaktivieren | `webhooks:write` | — |
| POST | `/webhooks/{id}/test/` | Test-Webhook senden | `webhooks:write` | — |
| GET | `/webhooks/{id}/deliveries/` | Zustellungen eines Webhooks auflisten | `webhooks:read` | — |

## Rechnungen

### Atomic Invoice — `POST /invoices/atomic/`

Der zentrale Endpunkt für die ERP-Integration: ein einziger Aufruf erstellt,
finalisiert und prüft die Rechnung und liefert das fertige **ZUGFeRD-PDF**
und die **XRechnung-XML** (beide Base64) zurück. Erfordert im Live-Betrieb
einen Idempotenz-Schlüssel. Das **vollständige Feldschema** steht unter
[Atomic Invoice](atomic-invoice.md).

### Preview — `POST /invoices/preview/`

Berechnet aus übergebenen Positionen die Summen exakt (Dezimal) inklusive
Aufschlüsselung je Steuersatz. Keine Speicherung, auch mit Sandbox-Schlüssel
und bei inaktivem Abo nutzbar.

Nachlässe und Zuschläge auf Dokumentebene (`header_charges`, BG-20/21) gehen
ein wie bei [Atomic](atomic-invoice.md):
dieselben Felder, `amount` Pflicht und nie aus `base_amount` × `percentage`
berechnet (sonst `400`, `code=business`). Netto = Positionen − Nachlässe +
Zuschläge, die Steuer je Satz und Kategorie einmal gerundet.

```json
{
  "items": [
    {"quantity": "1", "unit_price_net": "12218.72", "vat_rate": "19.00", "tax_category_code": "S"},
    {"quantity": "1", "unit_price_net": "3093.57", "vat_rate": "19.00", "tax_category_code": "S"}
  ],
  "header_charges": [
    {"is_charge": false, "amount": "610.94", "base_amount": "12218.72", "percentage": "5.00",
     "reason_code": "95", "reason_text": "Rabatt", "vat_category_code": "S", "vat_rate": "19.00"}
  ]
}
```

→ `subtotal` 14701.35, `tax_amount` 2793.26, `total` 17494.61. Wer diese
Werte als `validation_totals` an Atomic schickt, sendet `tax_category_code`
an den Positionen mit: dann liegen Positionen und Nachlass in derselben Zeile
der Steueraufschlüsselung, und die Vorschau liefert centgenau die Summen, die
Atomic bucht. Ohne Kategorie wird nichts abgeleitet, und die Vorschau rundet
Positionen und Nachlass getrennt (höchstens 1 Cent Unterschied je Satz).

### Convert — `POST /invoices/convert/`

Rendert EN-16931-Rechnungsdaten ohne Speicherung in geprüftes XML
(CII/XRechnung oder UBL, je nach Rechnungstyp) und gibt KoSIT-Befunde zurück.
Ideal, um die eigene Datenabbildung zu testen.

**Abrechnung:** Jede gültige Live-Konvertierung (`valid=true`) ist eine
abrechenbare Einheit wie eine finalisierte Rechnung — selbes Kontingent, selbe
Staffel. `400`, `valid=false`, `503` und die Sandbox sind kostenfrei. Ein
optionaler `Idempotency-Key` (UUID) lässt eine Wiederholung mit unverändertem
Body einmal zählen; gespeichert wird dabei nichts von der Rechnung. Convert
steht einem Konto erst nach Bestätigung der AGB 2.0 zur Verfügung (bei der
Registrierung oder in der Console); ohne Bestätigung antwortet es mit `403`
(`code = agb_confirmation_required`), bevor etwas gerendert wird. Wie jeder
schreibende Live-Aufruf braucht Convert ein aktives Abo; ohne antwortet es mit
`403`.

### Stammdaten vorab prüfen — `POST /parties/check/`

Prüft die Stammdaten einer Partei, bevor die erste Rechnung an sie geht. Es
wird nichts angelegt, nichts gespeichert und nichts abgerechnet. Mindestens
eine der drei Angaben ist nötig.

```json
{
  "vat_id": "DE123456789", "country": "DE",
  "leitweg_id": "04011000-1234512345-06",
  "electronic_address": {"scheme": "0204", "value": "04011000-1234512345-06"}
}
```

| Block | Antwort |
|---|---|
| `vat_id` | `format_valid` (Aufbau + Länderpräfix passt zu `country`), `format_message`, `vies_status` = `VALID` \| `INVALID` \| `UNVERIFIED`, `vies_reason` |
| `leitweg_id` | `structure_valid`, `checksum_valid` (Prüfziffer nach ISO/IEC 7064 Mod 97-10, Leitweg-ID-Formatspezifikation 2.0.2; `null`, wenn schon der Aufbau falsch ist), `message` |
| `electronic_address` | `valid` für die Schemata `0204` (Leitweg-ID), `9930` (DE-USt-IdNr.), `EM` (E-Mail), `0088` (GLN); `null` für andere Schemata — die prüft Factora nicht inhaltlich |

`UNVERIFIED` heißt: VIES hat nicht entschieden — nicht erreichbar
(`vies_unreachable`), Nummer außerhalb der EU (`not_checked`) oder Format
falsch (`format_invalid`). Es heißt **nie** „ungültig“.

Die Antwort enthält bewusst **keine Steuerkategorie**: eine registrierte
USt-IdNr. sagt nichts über die steuerliche Behandlung einer Rechnung aus.

Dieselbe Leitweg-ID-Prüfung läuft auch beim Anlegen einer Rechnung: ein
falscher Aufbau ist dort ein Fehler, eine falsche Prüfziffer eine Warnung.

### E-Rechnungspflicht prüfen — `POST /e-invoice-obligation/check/`

Beantwortet **vor** der Rechnungserstellung, ob ein Vorgang in Deutschland
eine E-Rechnung sein muss (§ 14 Abs. 2 UStG, Ausnahmen nach §§ 33, 34, 34a
UStDV, Übergang nach § 27 Abs. 38 UStG). Ihr System liefert die Fakten,
Factora liefert das Urteil. Es wird nichts angelegt, nichts gespeichert und
nichts abgerechnet.

```json
{
  "seller":   {"established_in_de": true, "small_business": false, "prior_year_turnover_over_800k": true},
  "buyer":    {"is_business": true, "established_in_de": true},
  "document": {"transmission_date": "2027-03-02", "supply_date": "2027-02-28",
               "gross_total": "1190.00", "currency": "EUR", "passenger_ticket": false},
  "lines":    [{"tax_category": "S"}, {"tax_category": "E", "ustg_section_4_number": "8"}]
}
```

| Feld | Pflicht | Bedeutung |
|---|---|---|
| `seller.established_in_de` | ja, `null` erlaubt | Sitz, Geschäftsleitung oder beteiligte Betriebsstätte im Inland. Das Adressland allein ist **keine** Ansässigkeit. |
| `seller.small_business` | ja | Kleinunternehmer nach § 19 UStG |
| `seller.prior_year_turnover_over_800k` | nein | nur für Umsätze 2027 nötig (Gesamtumsatz im Vorjahr > 800.000 €) |
| `buyer.is_business` | ja, `null` erlaubt | Empfänger bezieht die Leistung als Unternehmer für sein Unternehmen |
| `buyer.established_in_de` | ja, `null` erlaubt | wie beim Verkäufer |
| `document.transmission_date` | ja | Tag, an dem die Rechnung an den Empfänger übermittelt wird. Die Übergangsfristen hängen an der Übermittlung, nicht am Rechnungsdatum. |
| `document.supply_date` | ja, `null` erlaubt | Leistungsdatum; bei Abschlagsrechnungen das der künftigen Leistung. Das Rechnungsdatum ersetzt es nicht. |
| `document.passenger_ticket` | nein | Fahrausweis für die Personenbeförderung |
| `lines[].tax_category` | ja | `S`, `Z`, `E`, `AE`, `K`, `G`, `O` oder `UNDETERMINED`, solange die Kategorie noch nicht entschieden ist |
| `lines[].ustg_section_4_number` | bei `E` | Nummer des § 4 UStG (`8`, `9a`, `11` …). Nur Nr. 8–29 sind von der Pflicht ausgenommen. |

`null` heißt „nicht bekannt“. Das Ergebnis ist dann `undetermined` mit
Begründung — Factora rät nicht. Ein fehlender Pflichtschlüssel ergibt `400`.

**Antwort** (`data`):

| Feld | Werte |
|---|---|
| `required` | `yes` · `no` · `undetermined` |
| `reasons` | Liste aus `{code, message}`, z. B. `DOMESTIC_B2B`, `SMALL_AMOUNT`, `TRANSITION_UNTIL_2026`, `BUYER_TYPE_UNKNOWN` |
| `consent_needed_if_e_invoice` | `true`: eine E-Rechnung braucht die Zustimmung des Empfängers · `false`: ohne Zustimmung zulässig · `null`: offen |
| `basis` | Rechtsstand, auf dem die Antwort beruht |

`no` heißt „keine Pflicht“, nicht „verboten“: Zwischen zwei inländischen
Unternehmern ist eine E-Rechnung immer zulässig.

Nicht abgedeckt: Vorgaben für Rechnungen an öffentliche Auftraggeber
(E-Rechnungsverordnung des Bundes, Landesrecht). Sie gelten unabhängig von
§ 14 UStG; ein `no` sagt darüber nichts.

### Klassischer Lebenszyklus

Statt des Atomic-Wegs lässt sich eine Rechnung auch schrittweise führen:
anlegen (`POST /invoices/`) → finalisieren (`/finalize/`) → herunterladen
(`/pdf/`, `/xml/`) → versenden (`/send/`) oder einreihen (`/dispatch/`).
Nachlässe und Zuschläge auf Dokumentebene (`header_charges`, BG-20/21) kennt
dieser Weg nicht: `POST /invoices/` lehnt ein mitgeschicktes, nicht leeres
`header_charges` mit `400` ab (`code=schema`, `field=header_charges`), statt
es still zu verwerfen. Dafür Atomic oder Convert verwenden.
Listen sind paginiert und filterbar; per `mandant`-Filter lässt sich auf
einen Mandanten einschränken.

## Mandanten

CRUD über `/mandanten/`. Der **Bulk-Endpunkt** legt bis zu 200 Mandanten in
einer Transaktion an (**alles oder nichts** — schlägt eine Zeile fehl, z. B.
durch einen doppelten Namen, wird nichts angelegt; Antwort 409). Ein Mandant
kann nicht gelöscht werden, solange Rechnungen oder Kunden an ihm hängen
(409, GoBD).

**Branding je Mandant** (Logo, Briefpapier, Farbe, Schrift, Fußtext und bis zu
fünf **Fußzeilen-Logos**) lässt sich vollständig über die API pflegen — oder
von Hand in der Console; beide Wege prüfen dieselben Regeln. Farbe
(`primary_color`), Schrift (`font_choice`: `helvetica`, `times`, `courier`),
`logo_position`, `footer_text` und die Briefpapier-Einstellungen setzt
`PATCH /mandanten/{id}/`. Die Dateien laufen als `multipart/form-data` über
eigene Unterpfade:

```bash
curl -X POST https://console.factora.software/api/v1/mandanten/42/logo/ \
  -H "Authorization: Bearer $FACTORA_API_KEY" \
  -F "logo=@logo.png"
```

- `logo/` — Feld `logo`, lesbare Bilddatei (PNG, JPEG, …) bis 5 MB; ersetzt
  ein vorhandenes Logo. `DELETE` entfernt es.
- `letterhead/` — Feld `letterhead`, PDF bis 5 MB. `DELETE` entfernt es.
- `footer-logos/` — Feld `image` (wie beim Logo), optional `position`
  (`left`/`center`/`right`, Standard `left`); höchstens fünf je Mandant.
  `PATCH …/footer-logos/{logo_id}/` ändert `position`/`sort_order`,
  `DELETE` entfernt ein Logo.

Jede Antwort ist der vollständige Mandant. Eine Datei, die keine lesbare
Bilddatei bzw. kein PDF ist oder zu groß ist, wird mit 400 abgelehnt (das
vorhandene Logo bleibt dann unverändert). Branding setzt einen Tarif mit
Branding voraus (sonst 403); Sandbox-Schlüssel sind für diese Pfade nicht
zugelassen. Das Logo erscheint nur auf Rechnungen, die mit `mandant_id`
erzeugt werden — ein Mandant ohne eigenes Logo zeigt keines.

Die Mandanten-Antwort trägt den Stand lesend: `logo` (URL oder `null`),
`has_letterhead` und `footer_logos` (Liste aus `id`, `url`, `position`,
`sort_order`, in Zeichenreihenfolge). Fußzeilen-Logos erscheinen auf jeder
Rechnungsseite in der Fußzeile; der Fußtext rückt daneben oder darüber, ein
Briefpapier mit eigener Fußzeile unterdrückt beides.

## Kunden

Auflisten und Anlegen über `/customers/`. Auf dem Mandanten-Pfad werden
Kunden pro Mandant getrennt geführt.

## Exporte

`GET /exports/datev/` und `GET /exports/invoices/` — siehe [Exporte](exports.md).

## Webhooks

CRUD, Testversand und Zustellhistorie über `/webhooks/` — siehe
[Webhooks](webhooks.md).
