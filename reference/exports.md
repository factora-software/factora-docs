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
| Kodierung | Windows-1252 (CP1252) — DATEV-kompatibel |
| Dateiname | `datev_{date_from}_{date_to}.csv` |
| Format | DATEV-Buchungsdaten (Buchungsstapel) |
| Umfang | alle finalisierten Rechnungen im Zeitraum (Entwürfe ausgeschlossen), kontogebunden je Konto |

Jede Rechnung wird als Buchungszeile abgebildet; bei gemischten Steuersätzen
erfolgt eine Aufteilung je Satz. Enthalten sind u. a. Betrag, Soll/Haben-
Kennzeichen, Konto und Gegenkonto, Buchungsschlüssel (BU), Belegdatum sowie
die Debitorennummer (sofern am Käufer hinterlegt).

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
