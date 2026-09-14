# Factora — Funktions- & API-Referenz

Vollständige technische Referenz der Factora E-Invoicing-Plattform: was die
Plattform kann, welche Rechnungsformate sie erzeugt, welche Steuerlogik
greift und wie die öffentliche REST-API im Detail funktioniert — inklusive
aller Felder, Rechnungstypen und Endpoints.

> Diese Referenz ist format- und integrationsneutral. Sie beschreibt die
> Plattform allgemein, nicht einen einzelnen Anwendungsfall.

## Überblick

Factora ist eine E-Invoicing-Plattform für Deutschland und die EU. Sie nimmt
Rechnungsdaten in einem strukturierten Format entgegen und erzeugt daraus
gesetzeskonforme elektronische Rechnungen nach **EN 16931** — als
**XRechnung** (CII), **ZUGFeRD** (PDF/A-3 mit eingebetteter XML) und
**Peppol/UBL**. Jede finalisierte Rechnung wird gegen das amtliche
Regelwerk geprüft, revisionssicher (GoBD) archiviert und kann per DATEV
exportiert werden.

Es gibt zwei Wege, Factora zu nutzen:

- **Web-Anwendung** — Rechnungen im Browser erfassen, finalisieren, versenden.
- **Öffentliche REST-API** — Rechnungserstellung direkt aus einem Fremdsystem
  (ERP, Warenwirtschaft, Abrechnungssoftware). Ein einziger Aufruf erzeugt
  eine fertige, geprüfte Rechnung als PDF und XML.

## Inhaltsverzeichnis

### Plattform & Engine

- [Rechnungstypen](invoice-types.md) — alle unterstützten Rechnungsarten und
  ihre TypeCodes (Rechnung, Korrektur, Abschlag, kumulierte Abschläge,
  Schlussrechnung, Gutschrift/Self-Billing).
- [Rechnungsformate](formats.md) — XRechnung (CII), ZUGFeRD (PDF/A-3),
  Peppol/UBL; Profile, Prüfung und amtliche Abnahme (KoSIT).
- [Peppol BIS 3.0 & EAS-Codes](peppol.md) — Customization-/Profile-Kennungen,
  Gutschriftverfahren (389), EAS-Codes der Teilnehmer-Endpunkte.
- [Validierung](validation.md) — XSD und Schematron, geprüfte Regelsätze,
  Aufbau eines Befunds.
- [Steuerlogik](tax-logic.md) — alle USt-Kategorien, Steuersätze,
  Befreiungsgründe, §13b Reverse Charge, innergemeinschaftliche Lieferung,
  Ausfuhr, §19 Kleinunternehmer, grenzüberschreitende Logik, Prüfregeln.
- [Compliance & Archivierung](compliance.md) — GoBD-Unveränderlichkeit,
  verschlüsseltes Archiv, Änderungsprotokoll, EN-16931-Validierung.

### Öffentliche API

- [Schnellstart](../api/quickstart.md) — vom leeren Account zum ersten
  authentifizierten Aufruf.
- [API-Überblick](api-overview.md) — Basis-URL, Authentifizierung,
  API-Schlüssel, Sandbox/Live, Idempotenz, Limits, Antwort-Hülle,
  Fehlercodes, Scopes.
- [Sandbox & Testing](sandbox.md) — kostenlose Trockenläufe, zugelassene
  Endpunkte, Verhalten des Dry-Runs.
- [Endpoints](api-endpoints.md) — vollständige Liste aller Endpunkte mit
  Methode, Zweck, erforderlichem Scope und Verhalten.
- [Atomic Invoice — vollständiges Feldschema](atomic-invoice.md) — jedes Feld
  des Ein-Aufruf-Rechnungsendpunkts (Kopf, Verkäufer, Käufer, Positionen,
  Summen, Anhänge) mit Typ, Pflicht/Optional, Default und EN-16931-Bezug.
- [Webhooks](webhooks.md) — Ereignistypen, Signaturprüfung, Zustellung &
  Wiederholungslogik.
- [Exporte](exports.md) — DATEV-Export (Buchungsdaten), Rechnungsarchiv als ZIP.

## Konventionen in dieser Referenz

- **BT-/BG-Codes** verweisen auf die Geschäftsbegriffe (Business Terms /
  Groups) der Norm **EN 16931**.
- **TypeCodes** (z. B. 380, 384) stammen aus **UNTDID 1001**,
  **Steuerkategorien** aus **UNTDID 5305**, **Maßeinheiten** aus
  **UN/ECE Rec. 20 / UNCL 5272**.
- Geldbeträge werden als Dezimal-Strings mit 2 Nachkommastellen übertragen
  (z. B. `"1190.00"`), Mengen mit 3 (z. B. `"10.500"`), Sätze mit 2
  (z. B. `"19.00"`). Datumsangaben sind **ISO 8601** (`YYYY-MM-DD`).

## Interaktive Doku

Die maschinenlesbare OpenAPI-3.1-Spezifikation mit Ausprobier-Funktion ist
öffentlich erreichbar unter **`console.factora.software/api/docs/`**.
