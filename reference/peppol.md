# Peppol BIS 3.0 & EAS-Codes

Für den Peppol-Transport erzeugt Factora **UBL** (Universal Business Language)
nach **Peppol BIS Billing 3.0**. Diese Seite beschreibt die emittierten
Customization- und Profile-Kennungen, die Behandlung des Gutschriftverfahrens
(TypeCode 389) sowie die EAS-Codes der Teilnehmer-Endpunkte.

Grundlagen zu den Formaten: [Rechnungsformate](formats.md).

## Customization & Profile (Billing)

Der UBL-Writer setzt im Kopf standardmäßig:

| Element | Wert |
|---|---|
| `cbc:CustomizationID` | `urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0` |
| `cbc:ProfileID` | `urn:fdc:peppol.eu:2017:poacc:billing:01:1.0` |

Ein explizit gesetztes `invoice_header.customization_id` überschreibt die
Customization.

## Gutschriftverfahren / Self-Billing (TypeCode 389)

Bei `InvoiceTypeCode = 389` (Gutschriftverfahren nach § 14 Abs. 2 UStG — der
Käufer stellt im Namen des Verkäufers aus) wechselt der Writer automatisch auf
das **Peppol-BIS-Self-Billing-3.0-Profil**:

| Element | Wert (bei 389) |
|---|---|
| `cbc:CustomizationID` | `urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:selfbilling:3.0` |
| `cbc:ProfileID` | `urn:fdc:peppol.eu:2017:poacc:selfbilling:01:1.0` |

Das ist nötig, weil das Standard-Billing-Profil den TypeCode 389 ablehnt.

> **Zum Abnahmestatus:** Der **UBL**-Self-Billing-Pfad (389) ist bislang nicht
> amtlich abgenommen — das korrekte Profil wird erzeugt, die Prüfung über einen
> Peppol-Self-Billing-Validator steht aus. Der CII-Pfad (XRechnung) ist
> KoSIT-geprüft. Siehe die Übersicht unter
> [Rechnungsformate](formats.md#validierung--amtliche-prüfung-kosit).

## InvoiceTypeCodes

| Code | Bedeutung | Wurzelelement |
|---|---|---|
| `380` | Rechnung | `<Invoice>` |
| `381` | Gutschrift (Korrektur) | `<CreditNote>` |
| `389` | Gutschriftverfahren / Self-Billing | `<Invoice>` (Self-Billing-Profil) |

## EAS-Codes (EndpointID)

Die Teilnehmer-Endpunkte (`cbc:EndpointID` mit `schemeID`) werden nach
folgender Priorität gesetzt:

| Priorität | Quelle | `schemeID` |
|---|---|---|
| 1 | explizite Peppol-Teilnehmerkennung + Schema | wie gesetzt |
| 2 | deutsche USt-IdNr. (beginnt mit `DE`) | `9930` |
| 3 | E-Mail (Rückfallebene) | `EM` |

`EM` ist **kein** gültiger Peppol-EAS-Code — daher wird eine deutsche
USt-IdNr. bevorzugt, damit das UBL Peppol-valide bleibt. Zusätzlich wird eine
**GLN**, sofern vorhanden, als `cac:PartyIdentification` mit `schemeID = 0088`
ausgegeben.

Beispiel — EndpointID mit deutscher USt-IdNr. (EAS 9930):

```xml
<cbc:EndpointID schemeID="9930">DE123456789</cbc:EndpointID>
```

## Hinweise

- Ohne Peppol-Teilnehmerkennung, deutsche USt-IdNr. **und** E-Mail wird keine
  EndpointID emittiert.
- Die `EM`-Rückfallebene ist nicht Peppol-konform — für den Peppol-Versand
  sollte eine Teilnehmerkennung oder eine deutsche USt-IdNr. hinterlegt sein.

Siehe auch [Validierung](validation.md) und [Endpoints](api-endpoints.md).
