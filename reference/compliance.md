# Compliance & Archivierung

Factora ist auf die deutschen Aufbewahrungs- und Nachvollziehbarkeits­
anforderungen (GoBD) ausgelegt. Finalisierte Rechnungen sind unveränderlich,
werden verschlüsselt archiviert und jede Statusänderung wird protokolliert.

## Unveränderlichkeit finalisierter Rechnungen (GoBD)

Sobald eine Rechnung finalisiert ist, sind ihre rechnungsrelevanten Daten
gesperrt. Geschützte (eingefrorene) Felder sind u. a.:

- Rechnungsnummer, Rechnungsdatum, Fälligkeitsdatum
- Käufer/Empfänger-Snapshot
- Netto-, Steuer- und Bruttobeträge
- Leitweg-ID (B2G)
- Steuerkategorie, Steuersatz, Befreiungsgründe je Position

Ein Schreibversuch auf eine finalisierte Rechnung wird abgelehnt. Erlaubt
sind nur definierte **Statusübergänge** (z. B. finalisiert → versendet →
bezahlt → storniert).

**Löschen** ist ausschließlich im Entwurfsstatus möglich. Eine finalisierte
Rechnung kann nicht gelöscht, sondern nur über eine
[Korrekturrechnung](invoice-types.md) berichtigt werden.

## Verschlüsseltes Archiv

Bei der Finalisierung wird die Rechnung in einem verschlüsselten Archiv
abgelegt. Dabei werden die Metadaten (Nummer, Datum, Empfänger, Positionen,
Beträge) erfasst und über einen **SHA-256-Integritätshash** gesichert, sodass
nachträgliche Veränderungen erkennbar wären. Der Archivstatus jeder Rechnung
ist nachvollziehbar (ausstehend / archiviert / fehlgeschlagen).

## Änderungsprotokoll (Audit-Log)

Jede Statusänderung einer Rechnung — insbesondere der Übergang vom Entwurf
zur finalisierten Rechnung — wird mit Zeitpunkt und auslösendem Akteur in
einem Änderungsprotokoll festgehalten.

## Normprüfung (EN 16931 / KoSIT)

Vor der Auslieferung durchläuft jede Rechnung die fachliche Prüfung gegen das
EN-16931-Regelwerk und — je nach Format — den amtlichen **KoSIT-Validator**.
Befunde werden strukturiert zurückgegeben (Regel, Geschäftsbegriff,
Fundstelle). Welche Formate amtlich abgenommen sind, steht unter
[Rechnungsformate](formats.md).

## Datenschutz & Betrieb

- Betrieb und Datenhaltung in **Deutschland**.
- **DSGVO**-konform mit Auftragsverarbeitungsvertrag (AVV).
- Tägliche Sicherung der Daten.
- Sensible Felder (z. B. Bankverbindung) werden verschlüsselt gespeichert.

## Datenexport

- Vollständiger Konto-Datenexport auf Anforderung.
- **DATEV-Export** der Buchungsdaten für die Finanzbuchhaltung — siehe
  [Exporte](exports.md).
