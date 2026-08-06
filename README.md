# Factora — Entwicklerdokumentation

Dokumentation zur **Factora E-Rechnungs-API**: XRechnung, ZUGFeRD und Peppol
nach EN 16931, aus einem Fremdsystem heraus erzeugt, validiert und versendet.

- **Basis-URL:** `https://console.factora.software/api/v1`
- **Interaktive Referenz (OpenAPI 3.1):** <https://console.factora.software/docs>
- **Python-SDK:** <https://github.com/factora-software/factora-python>

Die Dokumentation ist auf Deutsch, weil sich die zugrunde liegenden Standards
(XRechnung-CIUS, BR-DE-Regeln, DATEV-Export) auf den deutschen Rechtsraum
beziehen.

---

## Einstieg

| Wenn du … | dann hier |
|---|---|
| zum ersten Mal integrierst | [api/integration-guide.md](api/integration-guide.md) |
| eine Rechnung in **einem** POST erzeugen willst | [reference/atomic-invoice.md](reference/atomic-invoice.md) |
| wissen willst, welches Feld welchem BT entspricht | [api/en16931-field-reference.md](api/en16931-field-reference.md) |
| einen Endpunkt nachschlagen willst | [reference/api-endpoints.md](reference/api-endpoints.md) |
| Authentifizierung, Idempotency, Fehlerformat brauchst | [reference/api-overview.md](reference/api-overview.md) |
| kaufmännische Fragen hast (Tarif, Sandbox, Limits) | [api/faq.md](api/faq.md) |

## Inhalt

### `api/` — Leitfäden

| Datei | Inhalt |
|---|---|
| [integration-guide.md](api/integration-guide.md) | Integrationsleitfaden von Key-Erstellung bis Versand (auch als PDF) |
| [en16931-field-reference.md](api/en16931-field-reference.md) | Vollständige Feldreferenz EN 16931 ↔ Factora-JSON (auch als PDF) |
| [faq.md](api/faq.md) | Onboarding, Tarif, Sandbox, Abrechnung |
| [postman/](api/postman) | E2E-Collection + Environment-Template |

### `reference/` — Nachschlagewerk

| Datei | Inhalt |
|---|---|
| [api-overview.md](reference/api-overview.md) | Auth, Antwort-Hülle, Fehlercodes, Idempotency |
| [api-endpoints.md](reference/api-endpoints.md) | Alle Endpunkte im Überblick |
| [atomic-invoice.md](reference/atomic-invoice.md) | Der Atomic-Endpunkt im Detail |
| [invoice-types.md](reference/invoice-types.md) | Rechnung, Korrektur, Abschlag, Gutschrift |
| [formats.md](reference/formats.md) | XRechnung, ZUGFeRD, UBL, CII |
| [tax-logic.md](reference/tax-logic.md) | Steuerkategorien, Reverse Charge, OSS |
| [compliance.md](reference/compliance.md) | GoBD, Aufbewahrung, Validierung |
| [exports.md](reference/exports.md) | DATEV-Export |
| [webhooks.md](reference/webhooks.md) | Ereignisse und Zustellung |

### `samples/en16931/` — Beispieldokumente

Vollständig befüllte Beispiele: [maximal_input.json](samples/en16931/maximal_input.json)
(Request), [maximal_cii.xml](samples/en16931/maximal_cii.xml) und
[maximal_ubl.xml](samples/en16931/maximal_ubl.xml) (erzeugte Dokumente). Alle
Daten sind erfunden.

---

## Schnellstart

```bash
pip install factora
```

```python
from factora import FactoraClient

client = FactoraClient(api_key="fa_test_…")
result = client.create_atomic_invoice(payload, idempotency_key="…")
```

Den vollständigen Payload mit allen Pflichtfeldern zeigt der
[Integrationsleitfaden](api/integration-guide.md); die Bedeutung jedes Feldes
steht in der [Feldreferenz](api/en16931-field-reference.md).

Sandbox-Keys (`fa_test_…`) sind kostenlos und erzeugen keine abrechenbaren
Rechnungen.

---

## Hinweise

**Diese Dokumentation wird aus dem Produkt-Repository gespiegelt.** Änderungen
gehören dorthin, nicht hierher — ein Pull Request gegen dieses Repo würde beim
nächsten Abgleich überschrieben. Fehler und Lücken bitte als
[Issue](https://github.com/factora-software/factora-docs/issues) melden.

Die PDF-Fassungen werden aus denselben Quellen erzeugt wie die Markdown-Dateien
und sind inhaltlich identisch.

## Lizenz

[MIT](LICENSE) — die Code-Beispiele dürfen frei übernommen werden.
