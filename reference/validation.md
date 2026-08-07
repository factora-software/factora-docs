# Validierung — XSD & Schematron

Factora prüft die erzeugte Rechnungs-XML gegen den **KoSIT-Validator** —
denselben Prüfdienst, den auch die öffentliche Verwaltung einsetzt. Die
Prüfung umfasst das **XSD-Schema** und die **Schematron-Geschäftsregeln**
(EN 16931, XRechnung BR-DE, Peppol). Diese Seite beschreibt den Aufbau, die
geprüften Regelsätze und die Struktur eines Befunds.

Formate: [Rechnungsformate](formats.md) (CII) und [Peppol](peppol.md) (UBL).

## Aufbau

Die Validierung läuft gegen **zwei getrennte KoSIT-Validator-Dienste** per
HTTP:

| Dienst | Geltungsbereich |
|---|---|
| XRechnung-Validator | XRechnung 3.0 + EN 16931 + CII |
| Peppol-Validator | Peppol BIS Billing 3.0 |

Welcher Dienst greift, entscheidet die `CustomizationID` des Dokuments. Der
KoSIT-Report (Schematron-`failed-assert` bzw. `successful-report`) wird
geparst und in eine strukturierte Befundliste übersetzt.

## Geprüfte Regelsätze

Die deutschen Meldungen zu den Regel-Codes sind in eigenen Katalogen
hinterlegt:

| Regelsatz | Inhalt |
|---|---|
| `BR-DE-*` | XRechnung-CIUS (21 Regeln) |
| `BR-*` / `BR-CO-*` | EN-16931-Basis, Rechen- und Querprüfungen |
| `BR-S/E/Z/O/AE/IC-*`, `BR-CL-*` | Steuerkategorie- und Codelisten-Regeln |
| `PEPPOL-EN16931-R*` | Peppol-spezifische Regeln |
| `UBL-*` | UBL-Strukturregeln |

## Struktur eines Befunds

Das Validierungsergebnis trägt `is_valid`, `errors`, `warnings` und eine
strukturierte Befundliste. Jeder Befund enthält:

| Feld | Bedeutung |
|---|---|
| `severity` | `error`, `warning` oder `fatal` |
| `message` | Klartext-Meldung |
| `rule` | Regel-Code (z. B. `BR-DE-15`) |
| `bt` | betroffener Geschäftsbegriff (z. B. `BT-31`) |
| `location` | XPath im Dokument |
| `test` | zugrunde liegender Schematron-Test |

In der API-Antwort erscheinen KoSIT-Befunde mit dem Fehler-Code `kosit` und
liefern `rule`, `bt` und `location` mit — siehe
[API-Überblick](api-overview.md#fehler--und-antwortstruktur).

Beispiel — Befund bei fehlender Bestellreferenz (gekürzt):

```json
{
  "severity": "error",
  "rule": "BR-DE-15",
  "bt": "BT-10",
  "message": "Bestellreferenz fehlt",
  "location": "/ubl:Invoice/cbc:BuyerReference"
}
```

## Geschäftsregeln vs. KoSIT

Die EN-16931- und BR-DE-Geschäftsregeln werden bereits im Erstellungs- und
Finalisierungspfad geprüft. Die vollständige XSD- und
Schematron-Validierung erfolgt anschließend gegen den KoSIT-Dienst.

## Hinweise

- Maßgeblich ist die Version des eingesetzten Validator-Dienstes; eine im
  Anwendungscode geführte Versionsangabe gibt es bewusst nicht.
- Die deutschen Lösungshinweise je Regel stammen aus den Meldungskatalogen von
  Factora, nicht aus dem KoSIT-Report selbst.

Welche Formate amtlich abgenommen sind, steht unter
[Rechnungsformate](formats.md#validierung--amtliche-prüfung-kosit).
