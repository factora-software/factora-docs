# Rechnungsformate

Factora erzeugt elektronische Rechnungen nach der europäischen Norm
**EN 16931**. Aus denselben Rechnungsdaten werden je nach Bedarf
verschiedene Ausgabeformate generiert.

## Unterstützte Formate

| Format | Syntax | Datei | Einsatzzweck |
|---|---|---|---|
| **XRechnung** | CII (UN/CEFACT) | XML | Amtliches deutsches B2B-/B2G-Format |
| **ZUGFeRD** | CII, eingebettet | PDF/A-3 | Menschen- und maschinenlesbare Hybrid-Rechnung |
| **Peppol / UBL** | UBL (OASIS) | XML | Internationaler Versand über das Peppol-Netzwerk |
| **UBL CreditNote** | UBL | XML | Korrekturrechnung im Peppol-Format |

### XRechnung (CII)

Die **XRechnung** ist der deutsche CIUS (Core Invoice Usage Specification)
der EN 16931 und Pflichtformat für Rechnungen an öffentliche Auftraggeber
(B2G). Factora erzeugt die XRechnung in **CII-Syntax** (UN/CEFACT
Cross-Industry Invoice). Unterstützt werden u. a. Leitweg-ID (B2G),
Reverse-Charge-Hinweise, Abschläge und Korrekturbezüge.

### ZUGFeRD (PDF/A-3)

**ZUGFeRD** ist eine Hybrid-Rechnung: ein optisch lesbares **PDF/A-3** mit
**eingebetteter CII-XML**. Der Empfänger kann das PDF wie gewohnt ansehen,
während Buchhaltungssysteme die strukturierten Daten direkt auslesen. Die
XML entspricht inhaltlich der XRechnung. Zusätzliche Anhänge (z. B.
Lieferschein, Leistungsnachweis) können in das PDF/A-3 eingebettet werden.

### Peppol / UBL

Für den internationalen Versand über das **Peppol**-Netzwerk erzeugt Factora
**UBL** (Universal Business Language) nach **Peppol BIS Billing 3.0**.
Korrekturrechnungen werden im UBL-Format als **CreditNote** abgebildet.

## Validierung & amtliche Prüfung (KoSIT)

Jede finalisierte Rechnung wird vor der Auslieferung gegen das amtliche
Regelwerk geprüft. Factora nutzt dafür den offiziellen **KoSIT-Validator**
(Koordinierungsstelle für IT-Standards) — denselben Prüfdienst, den auch
die öffentliche Verwaltung einsetzt.

| Format / Fall | Status |
|---|---|
| XRechnung 3.0 (CII) | **Amtlich geprüft** (KoSIT) |
| Peppol/UBL Standardrechnung (Billing 3.0) | **Amtlich geprüft** (Peppol-Validator) |
| Reverse Charge §13b (AE), inländisch — CII & UBL | **Amtlich geprüft** (KoSIT) |
| Self-Billing (389) — CII / XRechnung | **Amtlich geprüft** (KoSIT) |
| Self-Billing (389) — Peppol/UBL | **Technisch vorbereitet** — korrektes Peppol-Self-Billing-Profil wird erzeugt; amtliche Abnahme über einen Peppol-Self-Billing-Validator steht aus |

Ergebnisse der Prüfung fließen in die [Antwort-Hülle](api-overview.md#fehler--und-antwortstruktur)
der API ein: KoSIT-Befunde erscheinen als strukturierte Fehler mit Regel
(`rule`), Geschäftsbegriff (`bt`) und Fundstelle (`location`).

## Format-Auswahl über die API

- Der **Atomic-Endpunkt** liefert die finalisierte Rechnung direkt als
  **ZUGFeRD-PDF** und **XRechnung-XML** (beides Base64) zurück.
- Zu einer gespeicherten Rechnung lassen sich die XML-Varianten gezielt
  abrufen: **CII/XRechnung** und **UBL/Peppol**.
- Der **Convert-Endpunkt** rendert EN-16931-Daten ohne Speicherung in
  geprüftes XML — nützlich zum Testen der eigenen Datenabbildung.

Details siehe [Endpoints](api-endpoints.md).
