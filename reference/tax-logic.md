# Steuerlogik

Factora bildet die umsatzsteuerliche Behandlung einer Rechnung normkonform
nach EN 16931 ab. Jede Position trägt eine **Steuerkategorie (BT-151,
UNTDID 5305)** und einen **Steuersatz (BT-152)**; daraus ergeben sich
Pflichtangaben wie Befreiungsgründe und die dokumentweite Steueraufstellung.

## Steuerkategorien

| Code | Bezeichnung | Satz | Anwendung | Befreiungsgrund nötig |
|---|---|---|---|---|
| **S** | Regelsteuersatz | 19 % oder 7 % | Normale Inlandsumsätze (auch ermäßigt besteuerte, 7 %) | nein |
| **Z** | Nullsatz | 0 % | Nullsatz-Umsätze | nein |
| **E** | Steuerbefreit | 0 % | u. a. §19 Kleinunternehmer | ja (Text) |
| **AE** | Reverse Charge | 0 % | §13b UStG (Steuerschuld beim Empfänger) | ja (auto) |
| **K** | Innergemeinschaftliche Lieferung | 0 % | §4 Nr. 1b UStG (EU → EU) | ja (auto) |
| **G** | Ausfuhrlieferung | 0 % | §4 Nr. 1a UStG (Export Drittland) | ja (auto) |
| **O** | Nicht steuerbar | 0 % | §3a Abs. 2 UStG (Leistung außerhalb) | individuell |

### Automatische Ableitung

Wird keine Kategorie ausdrücklich gesetzt, leitet Factora sie ab
(erste zutreffende Regel gewinnt):

1. **Ausdrückliche Vorgabe** (z. B. `AE`, `K`, `G`) → wird übernommen.
2. **Kleinunternehmer-Profil** → `E`.
3. **Steuersatz > 0** → `S`.
4. **Steuersatz = 0** → `Z`.

## Steuersätze

Unterstützt werden die deutschen Sätze **19 %**, **7 %** und **0 %**. Der
Satz wird je Position im Feld `vat_rate` als Prozentwert übergeben
(`19.00`, `7.00`, `0.00`). Für EU-B2C-Käufer (`customer_type: "B2C"`,
Käuferland ≠ Verkäuferland, EU) sind zusätzlich die Sätze des
**Bestimmungslandes** gültig (OSS-Fernverkauf, §3c UStG — z. B. NL 21 %/9 %);
die deutschen Sätze bleiben daneben erlaubt (unterhalb der
10.000-€-Schwelle wird weiter deutsch fakturiert).

## Befreiungsgründe (BT-120 / BT-121)

Für befreite bzw. steuerfreie Kategorien sind ein **Befreiungstext (BT-120)**
und ggf. ein **VATEX-Code (BT-121)** erforderlich. Factora füllt diese bei
den deterministischen Fällen automatisch:

| Kategorie | Befreiungstext (BT-120) | VATEX-Code (BT-121) |
|---|---|---|
| **AE** | „Steuerschuldnerschaft des Leistungsempfängers (§13b UStG / Art. 196 MwStSystRL)" | `VATEX-EU-AE` |
| **K** | „Innergemeinschaftliche Lieferung (§4 Nr. 1b UStG)" | `VATEX-EU-IC` |
| **G** | „Steuerfreie Ausfuhrlieferung (§4 Nr. 1a UStG)" | `VATEX-EU-G` |
| **E** (§19) | „Kleinunternehmer gemäß §19 UStG" | *(leer)* |
| **O** | individuell zu setzen | *(leer)* |

Eigene Texte/Codes können je Position übergeben werden und haben Vorrang vor
den Voreinstellungen.

## Reverse Charge §13b (AE)

Die Steuerschuld geht auf den Leistungsempfänger über (z. B. Bauleistungen
zwischen Unternehmern). Factora unterstützt **inländische und
grenzüberschreitende** Reverse-Charge-Fälle.

**Pflichtprüfungen:**

- Käufer muss eine **USt-IdNr.** besitzen (BR-AE-02).
- Nur für **B2B/B2G**, nicht für Endverbraucher (B2C).
- Steuersatz muss **0 %** sein.
- Befreiungstext (BT-120) ist Pflicht (wird automatisch gesetzt).

## Innergemeinschaftliche Lieferung (K) — §4 Nr. 1b

EU-grenzüberschreitende Lieferung an einen Unternehmer mit USt-IdNr.

**Pflichtprüfungen:**

- Käufer in einem **EU-Mitgliedstaat** (nicht im Inland).
- Käufer-**USt-IdNr.** vorhanden.
- **VIES-Prüfung** der USt-IdNr.: ungültig → blockiert; nicht erreichbar →
  nicht-blockierende Warnung mit späterer Wiederholung.
- Hinweis auf fehlenden **Gelangensnachweis** (§17a UStDV) als Warnung.

## Ausfuhrlieferung (G) — §4 Nr. 1a

Lieferung an einen Empfänger **außerhalb der EU** (Drittland). Steuersatz
0 %, Befreiungstext automatisch.

## Nicht steuerbar (O) — §3a Abs. 2

Leistung mit Ort außerhalb des deutschen Steuergebiets, an einen Empfänger
im Drittland. Befreiungstext individuell.

## Kleinunternehmer §19 UStG (E)

Bei Kleinunternehmer-Profil:

- Steuersatz **0 %**, Netto = Brutto (keine USt ausgewiesen).
- Kategorie **E**, Befreiungstext „Kleinunternehmer gemäß §19 UStG" ist
  Pflicht.
- VATEX-Code (BT-121) muss **leer** bleiben.
- Dokument-Hinweis: „Gemäß §19 UStG wird keine Umsatzsteuer berechnet."

## Steuerlicher Vertreter (BG-11/12)

Tritt der Verkäufer über einen **Fiskalvertreter** auf (z. B. ausländischer
Verkäufer mit Vertreter in DE), trägt die Rechnung dessen Angaben
(BT-62 bis BT-69). Wird der Name gesetzt, werden USt-IdNr. und Land zu
Pflichtfeldern. Die Angaben können tenantweit hinterlegt oder je
API-Rechnung übergeben werden.

## Grenzüberschreitende Einordnung

Aus Verkäufer-/Käuferland, USt-IdNr. und Kundentyp leitet Factora die
zutreffende Behandlung ab:

- **Gleiches Land** → `S` (Regelsteuer).
- **EU-grenzüberschreitend, B2B mit USt-IdNr.** → `AE` (Reverse Charge).
- **EU-grenzüberschreitend, B2C** → `S`; gültig sind deutsche Sätze
  **und** die Sätze des Bestimmungslandes (OSS-Fernverkauf, §3c UStG).
- **Drittland-Export** → `G`.
- **Nordirland (XI, Brexit-Protokoll):** Warenlieferungen EU-ähnlich
  (AE/S), Dienstleistungen als Export (`G`).

## Dokumentweite Steueraufstellung (BG-23)

Zusätzlich zur positionsbezogenen Steuer (BG-27/28) erzeugt Factora die
zusammengefasste **Steueraufstellung je Satz und Kategorie** (BG-23). Die
Summen werden aus den Positionen exakt (Dezimal, keine Float-Rundung)
berechnet und gegen die mitgelieferten Kontrollsummen (`validation_totals`)
geprüft.

## Prüfregeln — was eine Rechnung blockiert

Folgende Steuer-Prüfungen verhindern die Finalisierung (HTTP 400):

| Fehlercode | Kategorie | Bedingung |
|---|---|---|
| `K_VAT_ID_MISSING` | K | Käufer ohne USt-IdNr. |
| `K_COUNTRY_NOT_EU` | K | Käufer nicht in der EU |
| `K_COUNTRY_DOMESTIC` | K | Käufer im selben Land wie Verkäufer |
| `K_VAT_ID_INVALID` | K | USt-IdNr. laut VIES ungültig |
| `AE_CUSTOMER_TYPE_B2C` | AE | Reverse Charge an Endverbraucher |
| `AE_VAT_ID_MISSING` | AE | Käufer ohne USt-IdNr. (BR-AE-02) |
| `AE_BT120_MISSING` | AE | Befreiungstext fehlt (BR-AE-10) |
| `G_COUNTRY_NOT_THIRD` | G | Käufer innerhalb der EU |
| `O_COUNTRY_NOT_THIRD` | O | Käufer innerhalb der EU |
| `E_KU_BT120_MISSING` | E | Kleinunternehmer ohne Befreiungstext |
| `E_KU_BT121_SET` | E | Kleinunternehmer mit gesetztem VATEX-Code |

Nicht-blockierende Warnungen (HTTP 200, protokolliert):

| Warncode | Kategorie | Bedingung |
|---|---|---|
| `K_VAT_ID_UNCHECKED` | K | VIES-Dienst nicht erreichbar |
| `K_DELIVERY_PROOF_MISSING` | K | Gelangensnachweis noch nicht erfasst |

## Einfrieren bei Finalisierung (GoBD)

Steuerkategorie, Satz und Befreiungsgründe werden bei der Finalisierung als
Snapshot **eingefroren**, damit XML und PDF jederzeit reproduzierbar bleiben.
Siehe [Compliance & Archivierung](compliance.md).
