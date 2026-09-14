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

## Rechnungsarchiv als ZIP — `GET /api/v1/exports/invoices/`

Liefert alle finalisierten Rechnungen eines Zeitraums als ZIP — je Rechnung
die bei der Finalisierung archivierte **PDF** und **XML**, dazu ein Index und
ein Manifest. Gedacht für Archivübernahme, Betriebsprüfung und Systemwechsel;
für den laufenden Abruf einzelner Rechnungen sind `/invoices/{id}/pdf/` und
`/invoices/{id}/xml/` der richtige Weg.

### Parameter

| Parameter | Pflicht | Format | Bemerkung |
|---|---|---|---|
| `date_from` | ja | `YYYY-MM-DD` | Beginn des Zeitraums |
| `date_to` | ja | `YYYY-MM-DD` | Ende des Zeitraums |

Erforderlicher Scope: `exports:read`. Höchstens **1.000 Rechnungen** je
Abruf; enthält der Zeitraum mehr, antwortet der Endpunkt mit `400`
(`export_too_large`) und nennt die tatsächliche Anzahl.

### Inhalt des ZIP

| Datei | Inhalt |
|---|---|
| `rechnungen/<Nr>.pdf` | archivierte ZUGFeRD-PDF der Rechnung |
| `rechnungen/<Nr>.xml` | archiviertes XRechnung-/CII-XML der Rechnung |
| `index.csv` | Übersicht aller Rechnungen (Semikolon, UTF-8 mit BOM, Excel-kompatibel) |
| `index.json` | derselbe Index, maschinenlesbar (`export_info`, `invoices[]`) |
| `manifest.json` | Vollständigkeit: welche Dateien fehlen und warum |
| `README.txt` | Beschreibung des Archivs |

Das ZIP trägt ausschließlich die bei der Finalisierung archivierten Dateien;
nichts wird beim Export neu erzeugt.

### Vollständigkeit und Manifest

Kann eine Datei nicht aufgenommen werden — etwa weil zu einer Rechnung kein
archiviertes Dokument vorliegt oder der Speicher nicht lesbar ist —, wird das
ZIP trotzdem geliefert. Dann gilt:

- Der Pfad der fehlenden Datei bleibt in `index.csv` und `index.json`
  **leer** (`pdf_file` bzw. `xml_file`). **Der Index behauptet keinen Pfad,
  den das ZIP nicht enthält.**
- `manifest.json` nennt jede Lücke mit Rechnung, Datei und Grund:

```json
{
  "format": "GoBD-Export v1.0",
  "created_at": "2026-09-14T10:15:00+00:00",
  "invoice_count": 2,
  "file_count": 3,
  "missing_count": 1,
  "complete": false,
  "missing": [
    {
      "invoice_number": "RE-2026-0002",
      "file": "rechnungen/RE-2026-0002.pdf",
      "kind": "pdf",
      "reason": "Für diese finalisierte Rechnung ist kein PDF archiviert."
    }
  ]
}
```

- Der Antwort-Header **`X-Factora-Export-Gaps`** trägt dieselbe Anzahl
  (`0` = vollständig), damit ein Client die Lücke erkennt, ohne das ZIP zu
  öffnen.

Der HTTP-Status bleibt `200` — das ZIP ist gültig, nur nicht vollständig.
Ein vollständiger Export hat `complete: true`, `missing: []` und den Header
`X-Factora-Export-Gaps: 0`.

### Fehler

| HTTP | Bedingung | Code |
|---|---|---|
| 400 | `date_from`/`date_to` fehlt oder nicht im Format `YYYY-MM-DD` | `schema` |
| 400 | mehr als 1.000 Rechnungen im Zeitraum | `export_too_large` |

## Weitere Exporte

- **Rechnungsdokumente**: ZUGFeRD-PDF und XRechnung-/UBL-XML lassen sich pro
  Rechnung über die Rechnungs-Endpunkte herunterladen (siehe
  [Endpoints](api-endpoints.md)); für viele Rechnungen auf einmal das
  ZIP-Archiv oben.
- **Konto-Datenexport**: vollständiger Export der eigenen Kontodaten auf
  Anforderung (siehe [Compliance & Archivierung](compliance.md)).
