# Rechnungstypen

Factora unterstützt die in Deutschland gebräuchlichen Rechnungsarten. Der
Rechnungstyp wird über den **TypeCode (BT-3, UNTDID 1001)** gesteuert und
bestimmt die rechtliche Einordnung sowie die Struktur des erzeugten Dokuments.

## Unterstützte TypeCodes

| TypeCode | Rechnungsart | Bedeutung |
|---|---|---|
| **380** | Rechnung | Standard-Rechnung (Voreinstellung) |
| **381** | Korrektur als Minderung | Berichtigt eine Rechnung, indem sie den Betrag mindert (Credit Note), z. B. ein Storno |
| **384** | Korrekturrechnung | Berichtigt eine Rechnung, indem sie sie ersetzt |
| **386** | Abschlagsrechnung | Voraus-/Teilzahlung (auch kumulierte Abschläge) |
| **389** | Gutschrift (Gutschriftverfahren / Self-Billing) | Der Leistungsempfänger stellt die Rechnung im Namen des Leistenden aus (§ 14 Abs. 2 UStG) |

Über die öffentliche API wird der Typ im Feld `invoice_header.invoice_type`
gesetzt — als TypeCode (`"380"`) oder als internes Kürzel (`standard`,
`abschlag`, `abschlag_kumuliert`, `schlussrechnung`, `korrektur`,
`gutschrift`). Ohne Angabe gilt `380`. **Ein anderer Wert wird mit `400`
abgelehnt** — Factora macht aus einem unbekannten Code nie still eine
Standard-Rechnung.

## Die Rechnungsarten im Detail

### Standard-Rechnung (380)

Die normale Ausgangsrechnung. Enthält Verkäufer, Käufer, Positionen,
Steueraufstellung und Zahlungsangaben. Erzeugt XRechnung-XML und
ZUGFeRD-PDF.

### Abschlagsrechnung (386)

Für Voraus- oder Teilzahlungen vor vollständiger Leistungserbringung —
typisch bei längeren Projekten.

- **Einzelne Abschläge** werden als Abschlagsrechnung mit TypeCode 386
  ausgewiesen.
- **Kumulierte Abschläge** (`abschlag_kumuliert`, VOB/B § 16): jeder Abschlag
  nennt den Gesamtstand und zieht die vorigen Abschläge ab; der abgezogene
  Betrag wird beim Finalisieren eingefroren.
- Ein Abschlag kann nicht an eine bereits finalisierte Schlussrechnung
  gehängt werden (`400`) — deren abgesetzte Teilentgelte stehen fest.

### Schlussrechnung (`schlussrechnung`, TypeCode 380)

Schließt ein Projekt ab und setzt die bereits gezahlten Teilentgelte vom
Gesamtbetrag ab (§ 14 Abs. 5 UStG). Über die API gilt:

- `prepaid_amount` (BT-113) ist **Pflicht**: die Summe der vereinnahmten
  Teilentgelte. Wurde nichts vorab gezahlt, `0` angeben. Fehlt das Feld,
  antwortet die API mit `400` (`bt: BT-113`) — sonst stünde die volle Steuer
  ein zweites Mal auf der Rechnung.
- Die einzelnen Teilentgelte (Nummer, Datum, Entgelt und Steuer je Satz)
  gehören in `notes` (BT-22). So erlaubt es das BMF-Schreiben vom 15.10.2026
  (Rn. 3) bis 30.06.2030; alternativ als eingebettete Anlage, auf die `notes`
  verweist (Rn. 4).
- Kumulierte Abschläge zählen hier nicht automatisch mit — sie tragen ihren
  Stand selbst.

### Korrekturrechnung (384) und Minderung (381)

Berichtigt eine bereits gestellte Rechnung. Eine Korrektur **muss** die
ursprüngliche Rechnung nennen (BG-3 / BT-25 / BT-26, § 31 Abs. 5 UStDV) —
fehlt der Bezug, antwortet die API mit `400` (`bt: BT-25`). Zwei Wege, genau
einer davon:

- `parent_invoice` (nur Atomic) — ID einer Rechnung, die in Factora liegt;
  Nummer und Datum kommen aus ihr.
- `preceding_invoice_number` + `preceding_invoice_date` — Nummer und Datum
  einer Rechnung, die woanders ausgestellt wurde. Beide Felder gehören
  zusammen.

Der Code bleibt in beiden Syntaxen, was Sie senden: **384** ist eine
ersetzende Rechnung (XRechnung TypeCode 384, Peppol/UBL `Invoice` 384),
**381** eine Minderung (XRechnung TypeCode 381, Peppol/UBL `CreditNote`).

### Gutschrift / Self-Billing (389)

Das **Gutschriftverfahren** nach § 14 Abs. 2 UStG: nicht der Leistende,
sondern der **Leistungsempfänger** stellt die Rechnung aus (und vergibt
damit auch die Rechnungsnummer). Anwendungsfall z. B. Abrechnung durch den
Auftraggeber. Das Dokument trägt in BG-1 den Text „Gutschrift“
(§ 14 Abs. 4 Nr. 10 UStG).

> **Hinweis zum Reifegrad:** Im XRechnung/CII-Pfad ist der TypeCode 389
> erzeugbar und amtlich (KoSIT) geprüft. Im internationalen Peppol/UBL-Pfad
> wird das eigenständige **Peppol-Self-Billing-Profil** (BIS Self-Billing 3.0)
> erzeugt und gegen das offizielle OpenPeppol-Schematron validiert; die
> Prüfszenarien dafür pflegt Factora selbst, da weder KoSIT noch OpenPeppol
> eine fertige Validator-Konfiguration für Self-Billing ausliefert
> (siehe [Rechnungsformate](formats.md)).

## Abgrenzung: Begriff „Gutschrift"

In der Praxis wird „Gutschrift" doppeldeutig verwendet:

- **Kaufmännische Gutschrift / Stornierung** einer Rechnung → das ist eine
  **Korrektur als Minderung (381)** oder eine **Korrekturrechnung (384)**.
- **Gutschrift im umsatzsteuerlichen Sinn (Self-Billing)** → das ist der
  **TypeCode 389**.

Factora trennt beide Fälle über den TypeCode.
