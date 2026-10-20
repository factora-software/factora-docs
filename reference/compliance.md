# Compliance & Aufbewahrung

Factora erzeugt und prüft Rechnungen und gibt sie zurück. **Factora archiviert
Ihre Rechnungen nicht:** die Aufbewahrung nach GoBD, § 147 AO und § 14b UStG
liegt bei Ihnen (AGB § 20 Abs. 5, AVV § 10 Abs. 3). Finalisierte Rechnungen
sind unveränderlich, jede Statusänderung wird protokolliert, und bis zum Ende
der Abruffrist nach AGB § 20 können Sie jede Rechnung abrufen und für Ihr
eigenes Archiv exportieren ([Exporte](exports.md)).

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

## Verschlüsselte Ablage bis zum Vertragsende

Bei der Finalisierung wird die Rechnung verschlüsselt abgelegt, damit Sie sie
abrufen und exportieren können. Diese Ablage ist kein Archiv im Sinne der
GoBD: sie endet mit der Abruffrist nach AGB § 20, danach wird gelöscht. Dabei werden die Metadaten (Nummer, Datum, Empfänger, Positionen,
Beträge) erfasst und über einen **SHA-256-Integritätshash** gesichert, sodass
nachträgliche Veränderungen erkennbar wären. Der Ablagestatus jeder Rechnung
ist nachvollziehbar (`archive_status`: `pending` / `archived` / `failed`).

## Änderungsprotokoll (Audit-Log)

Jede Statusänderung einer Rechnung — insbesondere der Übergang vom Entwurf
zur finalisierten Rechnung — wird mit Zeitpunkt und auslösendem Akteur in
einem Änderungsprotokoll festgehalten — für die Dauer der Ablage bei Factora.
Ihre eigene Verfahrensdokumentation ersetzt es nicht.

## Normprüfung (EN 16931 / KoSIT)

Vor der Auslieferung durchläuft jede Rechnung die fachliche Prüfung gegen das
EN-16931-Regelwerk und — je nach Format — den amtlichen **KoSIT-Validator**.
Befunde werden strukturiert zurückgegeben (Regel, Geschäftsbegriff,
Fundstelle). Welche Formate amtlich abgenommen sind, steht unter
[Rechnungsformate](formats.md).

## Betrieb, Datenexport und Ausstieg

Hosting, Datensicherung, Datenexport, Löschung nach Kündigung und
Notfall-Kontakt stehen mit Belegen an einer Stelle:
[factora.software/vertrauen](https://factora.software/vertrauen/). Die
Export-Endpunkte selbst beschreibt [Exporte](exports.md).
