# Rechnungstypen

Factora unterstützt die in Deutschland und der EU gebräuchlichen
Rechnungsarten. Der Rechnungstyp wird über den **TypeCode (BT-3,
UNTDID 1001)** gesteuert und bestimmt die rechtliche Einordnung sowie die
Struktur des erzeugten Dokuments.

## Unterstützte TypeCodes

| TypeCode | Rechnungsart | Bedeutung |
|---|---|---|
| **380** | Rechnung | Standard-Rechnung (Voreinstellung) |
| **384** | Korrekturrechnung | Berichtigung einer bereits gestellten Rechnung |
| **386** | Abschlagsrechnung | Voraus-/Teilzahlung (auch kumulierte Abschläge) |
| **389** | Gutschrift (Gutschriftverfahren / Self-Billing) | Der Leistungsempfänger stellt die Rechnung im Namen des Leistenden aus (§14 Abs. 2 UStG) |

Über die öffentliche API wird der Typ im Feld `invoice_header.invoice_type`
gesetzt — entweder als TypeCode (`"380"`) oder als semantisches Kürzel. Ohne
Angabe gilt `380` (Standard-Rechnung).

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
- **Kumulierte Abschläge** werden unterstützt: bereits gestellte Abschläge
  werden fortlaufend aufsummiert geführt, sodass die Folgerechnung den
  korrekten kumulierten Stand trägt. Das deckt das im Bauwesen übliche
  Vorgehen nach **VOB/B §16** ab.

### Schlussrechnung

Schließt ein Projekt ab und verrechnet die zuvor geleisteten Abschläge gegen
den Gesamtbetrag. Die Schlussrechnung wird als reguläre Rechnung (TypeCode
380) geführt; der Bezug zu den Abschlägen wird über die kumulierte
Abschlagsführung hergestellt.

### Korrekturrechnung (384)

Berichtigt eine bereits gestellte Rechnung. Über das Feld
`invoice_header.parent_invoice` wird die ursprüngliche Rechnung referenziert
(BG-3 / BT-25 / BT-26). Im UBL/Peppol-Format wird eine Korrektur als
**CreditNote** abgebildet.

### Gutschrift / Self-Billing (389)

Das **Gutschriftverfahren** nach §14 Abs. 2 UStG: nicht der Leistende,
sondern der **Leistungsempfänger** stellt die Rechnung aus (und vergibt
damit auch die Rechnungsnummer). Anwendungsfall z. B. Abrechnung durch den
Auftraggeber.

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
  **Korrekturrechnung (384)**.
- **Gutschrift im umsatzsteuerlichen Sinn (Self-Billing)** → das ist der
  **TypeCode 389**.

Factora trennt beide Fälle sauber über den TypeCode.
