# Exporte

## DATEV-Export — `GET /api/v1/exports/datev/`

Erzeugt die **Buchungsdaten** der finalisierten Rechnungen eines Zeitraums
als CSV im DATEV-Format — direkt importierbar in die Finanzbuchhaltung bzw.
durch den Steuerberater.

### Parameter

| Parameter | Pflicht | Format | Bemerkung |
|---|---|---|---|
| `date_from` | ja | `YYYY-MM-DD` | Beginn des Zeitraums |
| `date_to` | ja | `YYYY-MM-DD` | Ende des Zeitraums |

Erforderlicher Scope: `exports:read`.

### Ausgabe

| Eigenschaft | Wert |
|---|---|
| Medientyp | `text/csv; charset=windows-1252` |
| Kodierung | Windows-1252 (CP1252) mit sicherer Transliteration — DATEV-kompatibel |
| Trennzeichen | Semikolon (`;`) |
| Text-Quoting | nur bei Bedarf (`QUOTE_MINIMAL`) |
| Dezimaltrennzeichen | Komma (deutsches Format, z. B. `595,00`) |
| Belegdatum | `DDMM` (z. B. `1806`) |
| Dateiname | `datev_{date_from}_{date_to}.csv` |
| Format | DATEV-Buchungsstapel, **118 Spalten** |
| Umfang | alle finalisierten Rechnungen im Zeitraum (Entwürfe ausgeschlossen), kontogebunden je Konto |

Die Datei beginnt mit einer Kopfzeile aus exakt 118 Spaltennamen, gefolgt von
einer Buchungszeile je Rechnung. Die ersten Spalten:

| # | Spaltenname |
|---|---|
| 1 | Umsatz (ohne Soll/Haben-Kz) |
| 2 | Soll/Haben-Kennzeichen |
| 3 | WKZ Umsatz |
| 4 | Kurs |
| 5 | Basis-Umsatz |
| 6 | WKZ Basis-Umsatz |
| 7 | Konto |
| 8 | Gegenkonto (ohne BU-Schlüssel) |
| 9 | BU-Schlüssel |
| 10 | Belegdatum |

Weitere Spalten umfassen u. a. Belegfeld 1/2, Buchungstext, Beleginfo 1–8 und
die KOST-Felder. Bei gemischten Steuersätzen erfolgt eine Aufteilung je Satz.

**Debitorennummer.** Das Feld *Gegenkonto* nutzt die am Kunden hinterlegte
Debitorennummer. Fehlt sie (Altbestand), greift ein synthetischer Fallback
(`10000` + Kunden-ID), damit bestehende Exporte unverändert bleiben.

> **Zur Versionsangabe.** Der Export schreibt die 118-spaltige
> Buchungsstapel-Feldreihe samt Spalten-Kopfzeile. Eine separate
> **EXTF-Metadatenzeile** (Formatname, Versionsnummer, Berater-/Mandanten-Nr.,
> WJ-Beginn, SKR) wird **nicht** ausgegeben — es steckt also kein
> maschinenlesbarer Versionsmarker in der Datei. Eine Versionsangabe bezieht
> sich auf das Feld-Layout, nicht auf eine im Export hinterlegte Formatversion.

### Fehler

| HTTP | Bedingung | Code |
|---|---|---|
| 400 | `date_from`/`date_to` fehlt | `schema` |
| 400 | Datum nicht im Format `YYYY-MM-DD` | `schema` |

## Weitere Exporte

- **Rechnungsdokumente**: ZUGFeRD-PDF und XRechnung-/UBL-XML lassen sich pro
  Rechnung über die Rechnungs-Endpunkte herunterladen (siehe
  [Endpoints](api-endpoints.md)).
- **Konto-Datenexport**: vollständiger Export der eigenen Kontodaten auf
  Anforderung (siehe [Compliance & Archivierung](compliance.md)).
