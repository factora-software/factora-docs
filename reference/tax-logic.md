# Steuerlogik

Factora bildet die umsatzsteuerliche Behandlung einer Rechnung normkonform
nach EN 16931 ab. Jede Position trägt eine **Steuerkategorie (BT-151,
UNTDID 5305)** und einen **Steuersatz (BT-152)**; daraus ergeben sich
Pflichtangaben wie Befreiungsgründe und die dokumentweite Steueraufstellung.

## Steuerkategorien

| Code | Bezeichnung | Satz | Anwendung | Befreiungsgrund nötig |
|---|---|---|---|---|
| **S** | Regelsteuersatz | 19 % oder 7 % | Normale Inlandsumsätze (auch ermäßigt besteuerte, 7 %) | nein |
| **Z** | Nullsatz | 0 % | Nullsatz-Umsätze | nein |
| **E** | Steuerbefreit | 0 % | u. a. §19 Kleinunternehmer | ja (Text) |
| **AE** | Reverse Charge | 0 % | §13b UStG (Steuerschuld beim Empfänger) | ja (auto) |
| **K** | Innergemeinschaftliche Lieferung | 0 % | §4 Nr. 1b UStG (EU → EU) | ja (auto) |
| **G** | Ausfuhrlieferung | 0 % | §4 Nr. 1a UStG (Export Drittland) | ja (auto) |
| **O** | Nicht steuerbar | 0 % | §3a Abs. 2 UStG (Leistung außerhalb) | individuell |

### Automatische Ableitung

Die Entscheidung erfolgt in dieser Reihenfolge:

1. **Ausdrückliche Vorgabe** (`tax_category_code`, z. B. `AE`, `K`, `G`)
   → wird übernommen; die fachlichen Prüfungen bei der Finalisierung bleiben
   bestehen. Zulässig sind genau die sieben Codes der Tabelle oben; jeder
   andere Code wird **abgelehnt** und landet nicht in BT-151.
2. **Rechtsgrundlage aus dem Katalog** (am Kunden oder Mandanten
   hinterlegt) → sofern sie für den Vorgang tragen kann: sie füllt eine
   Lücke, überstimmt aber keine eindeutige Ableitung aus den Tatsachen.
   `K` und `G` verlangen in jedem Fall den vollständig deklarierten
   Warensachverhalt (Schritt 3). Eine **stillgelegte** Rechtsgrundlage, die
   die Position entschieden hätte, macht sie **unbestimmt** — es wird nicht
   ersatzweise abgeleitet.
3. **Ableitung aus den Tatsachen** → Kleinunternehmer-Profil ergibt `E`,
   ein positiver Steuersatz sonst `S`. Bei 0 % können vollständig deklarierte
   grenzüberschreitende Warenlieferungen `K` oder `G` ergeben; alles andere
   bleibt bei 0 % unbestimmt.

**0 % allein ergibt niemals `Z`, auch nicht im Inland.** Ohne tragfähige
Angabe, Rechtsgrundlage oder Ableitung bleibt die Kategorie unbestimmt;
der Entwurf ist möglich, die Finalisierung wird abgelehnt.

## Steuersätze

Der Satz wird je Position im Feld `vat_rate` als Prozentwert übergeben
(`19.00`, `7.00`, `0.00`).

**Welche Sätze gültig sind, entscheidet das Land des Verkäufers** —
`seller_snapshot.country` beziehungsweise das Land des Mandanten:

| Verkäufer sitzt in | gültige Sätze |
|---|---|
| **Deutschland** (Vorgabe) | 19 %, 7 %, 0 % |
| **EU-Mitgliedstaat** | Normal- und ermäßigte Sätze dieses Landes plus 0 % (z. B. AT: 20 %, 13 %, 10 %, 0 %) |
| **außerhalb der EU** | wird abgelehnt — für diese Länder liegen uns keine Sätze vor |

0 % ist überall möglich: befreite, Reverse-Charge- und innergemeinschaftliche
Positionen tragen ihn.

Für EU-B2C-Käufer (`customer_type: "B2C"`, Käuferland ≠ Verkäuferland, EU)
sind **zusätzlich** die Sätze des **Bestimmungslandes** gültig
(OSS-Fernverkauf, §3c UStG — z. B. NL 21 %/9 %); die Sätze des
Verkäuferlandes bleiben daneben erlaubt (unterhalb der 10.000-€-Schwelle
wird weiter im eigenen Land fakturiert).

> Ein Verkäufer außerhalb der EU wird mit einer Geschäftsregel-Meldung
> abgelehnt („Verkäuferland CH wird nicht unterstützt …"), nicht stillschweigend
> durchgelassen. Bis August 2026 entfiel für jeden Nicht-DE-Verkäufer jede
> Satzprüfung; wer dort ein anderes Land eintrug, konnte jeden Satz zwischen
> 0 und 30 % ansetzen.

## Befreiungsgründe (BT-120 / BT-121)

Für befreite bzw. steuerfreie Kategorien sind ein **Befreiungstext (BT-120)**
und ggf. ein **VATEX-Code (BT-121)** erforderlich. Factora füllt diese bei
den deterministischen Fällen automatisch:

| Kategorie | Befreiungstext (BT-120) | VATEX-Code (BT-121) |
|---|---|---|
| **AE** | „Steuerschuldnerschaft des Leistungsempfängers (§13b UStG / Art. 196 MwStSystRL)" | `VATEX-EU-AE` |
| **K** | „Innergemeinschaftliche Lieferung (§4 Nr. 1b UStG)" | `VATEX-EU-IC` |
| **G** | „Steuerfreie Ausfuhrlieferung (§4 Nr. 1a UStG)" | `VATEX-EU-G` |
| **E** (§19) | „Kleinunternehmer gemäß §19 UStG" | *(leer)* |
| **O** | „Nicht steuerbarer Umsatz, Leistungsort im Ausland (§3a Abs. 2 UStG / Art. 44 MwStSystRL)" | `VATEX-EU-O` |

Eigene Texte/Codes können je Position übergeben werden und haben Vorrang vor
den Voreinstellungen.

**`O` trägt auf der Position keinen Steuersatz.** BR-O-05, BR-O-06 und
BR-O-07 verbieten bei „nicht steuerbar" den Steuersatz auf der Position
(BT-152) sowie an Rabatt und Zuschlag (BT-96/103). `O` bedeutet dort *Satz
nicht vorhanden*, nicht *Satz null*; `0.00 %` wäre die Aussage „besteuert, mit
null Prozent", und die trifft auf einen nicht steuerbaren Umsatz nicht zu.

In der **Steueraufstellung** bleibt BT-119 dagegen stehen: BR-48 erlaubt zwar,
ihn bei einer nicht steuerbaren Rechnung wegzulassen, BR-DE-14 verlangt ihn
aber — und `xrechnung` ist das Standardprofil.

**`O` und USt-IdNr. schließen sich aus.** BR-O-02: eine Rechnung mit einer
`O`-Position darf weder BT-31 (Verkäufer) noch BT-63 noch BT-48 (Käufer)
tragen. Der Verkäufer weist sich dann über die Steuernummer (BT-32) aus.

**`S` und `Z` dürfen keinen Befreiungsgrund tragen** (BR-S-10, BR-Z-10). Ein
mitgeschickter Text oder Code wird dort nicht übernommen — 0 % ist kein
Befreiungsgrund, sondern ein Steuersatz. Wer einen steuerfreien Umsatz meint,
setzt `E`, `AE`, `K`, `G` oder `O`.

Umgekehrt wird eine Kategorie **ohne** Grund abgelehnt statt durchgereicht
(BR-E-10, BR-AE-10, BR-IC-10, BR-G-10, BR-O-10). Für `AE`/`K`/`G`/`O` füllt
Factora den Grund selbst; bei `E` außerhalb §19 nicht — §4 UStG kennt zwei
Dutzend Befreiungen, und welche gemeint ist, weiß nur der Aufrufer.

## Reverse Charge §13b (AE)

Die Steuerschuld geht auf den Leistungsempfänger über (z. B. Bauleistungen
zwischen Unternehmern). Factora unterstützt **inländische und
grenzüberschreitende** Reverse-Charge-Fälle.

**Pflichtprüfungen:**

- Käufer muss eine **USt-IdNr.** besitzen (BR-AE-02).
- Nur für **B2B/B2G**, nicht für Endverbraucher (B2C).
- Steuersatz muss **0 %** sein.
- Befreiungstext (BT-120) ist Pflicht (wird automatisch gesetzt).

## Innergemeinschaftliche Lieferung (K) — §4 Nr. 1b

EU-grenzüberschreitende Lieferung an einen Unternehmer mit USt-IdNr.

**Pflichtprüfungen:**

- Käufer in einem **EU-Mitgliedstaat** (nicht im Inland).
- Käufer-**USt-IdNr.** vorhanden.
- **VIES-Prüfung** der USt-IdNr.: ungültig → blockiert; nicht erreichbar →
  nicht-blockierende Warnung mit späterer Wiederholung.
- Hinweis auf fehlenden **Gelangensnachweis** (§17a UStDV) als Warnung.

## Ausfuhrlieferung (G) — §4 Nr. 1a

Lieferung an einen Empfänger **außerhalb der EU** (Drittland). Steuersatz
0 %, Befreiungstext automatisch.

## Nicht steuerbar (O) — §3a Abs. 2

Leistung mit Ort außerhalb des deutschen Steuergebiets, an einen Empfänger
im Drittland. Befreiungstext individuell.

## Kleinunternehmer §19 UStG (E)

Bei Kleinunternehmer-Profil:

- Steuersatz **0 %**, Netto = Brutto (keine USt ausgewiesen).
- Kategorie **E**, Befreiungstext „Kleinunternehmer gemäß §19 UStG" ist
  Pflicht.
- VATEX-Code (BT-121) muss **leer** bleiben.
- Dokument-Hinweis: „Gemäß §19 UStG wird keine Umsatzsteuer berechnet."

## Steuerlicher Vertreter (BG-11/12)

Tritt der Verkäufer über einen **Fiskalvertreter** auf (z. B. ausländischer
Verkäufer mit Vertreter in DE), trägt die Rechnung dessen Angaben
(BT-62 bis BT-69). Wird der Name gesetzt, werden USt-IdNr. und Land zu
Pflichtfeldern. Die Angaben können tenantweit hinterlegt oder je
API-Rechnung übergeben werden.

## Grenzüberschreitende Einordnung

Über eine Grenze entscheidet nicht das Länderpaar allein, sondern **was
geliefert wird**. Deshalb trägt jede Position das optionale Feld
`items[].supply_type` mit den Werten `goods` (Warenlieferung) oder `service`
(Dienstleistung).

Factora leitet daraus **nur `K` und `G`** ab, und nur aus einem vollständig
deklarierten Warensachverhalt:

| Verkäufer → Käufer | Voraussetzungen | Kategorie |
|---|---|---|
| gleiches Land | positiver Steuersatz | `S`; bei 0 % **keine Ableitung** |
| EU → EU, B2B/B2G | `supply_type: goods` · formal gültige USt-IdNr. des Käufers mit passendem Länderpräfix · `shipping.country` in einem anderen Mitgliedstaat (BT-80) · `delivery_date` oder Abrechnungszeitraum (BT-72/BG-14) | `K` (§6a UStG) |
| EU → EU, B2C | positiver Steuersatz | `S`; zusätzlich gültig sind die Sätze des Bestimmungslandes (OSS-Fernverkauf, §3c UStG); bei 0 % **keine Ableitung** |
| EU → Drittland | `supply_type: goods` · `shipping.country` außerhalb der EU | `G` (§6 UStG) |
| alles übrige | — | **keine Ableitung** — die Position bleibt unbestimmt |

Die `S`-Zeilen ergeben sich bereits aus dem positiven Satz (Schritt 3 der
Ableitung); die Länderprüfung leitet ausschließlich `K` und `G` ab.

Die Lieferangaben sind nicht willkürlich gewählt: **BR-IC-11** verlangt bei
`K` das Lieferdatum (BT-72) oder den Abrechnungszeitraum (BG-14),
**BR-IC-12** das Lieferland (BT-80). Ohne sie wäre die Rechnung ohnehin
formal unzulässig.

### Was Factora nicht behauptet

**Die Ableitung kennzeichnet die Rechnung — sie ersetzt keinen Nachweis.**
`supply_type`, BT-80 und BT-72/BG-14 sind Ihre Angaben. Dass die Ware
tatsächlich ins übrige Gemeinschaftsgebiet gelangt ist, weisen Sie nach §17b
UStDV über die Gelangensbestätigung nach; die Ausfuhr belegen Sie nach §8
UStDV. Beides bleibt bei Ihnen und außerhalb dieser Ableitung.

### `AE` und `O` werden nie automatisch vergeben

§3a Abs. 2 UStG ist die B2B-Grundregel, aber Abs. 3 bis 8 nehmen ganze Klassen
heraus — Grundstücksleistungen, Eintrittsberechtigungen, Restaurantleistungen,
Personenbeförderung, kurzfristige Vermietung. Welcher Fall bei Ihnen vorliegt,
steht in keinem Feld, das wir halten. Eine Dienstleistung ohne gesetzte
Kategorie bleibt bei 0 % ohne anwendbare Rechtsgrundlage deshalb unbestimmt;
für `AE` und `O` erklären Sie die Kategorie oder die passende Rechtsgrundlage.

### Unbestimmt bleibt unbestimmt

Reichen die Angaben nicht, setzen wir **keine** Ersatzkategorie. Insbesondere
nicht `Z`: `Z` heißt „mit 0 % besteuert" und ist eine steuerliche Aussage über
Ihren Umsatz — „wir wissen es nicht" ist keine. Ein Dokument, das die
Formprüfung besteht und etwas Falsches behauptet, ist schlimmer als eine
Ablehnung, weil niemand es bemerkt.

Praktisch heißt das: der Entwurf darf die Kategorie offen lassen, die
**Finalisierung wird abgelehnt** — mit einer Meldung, die sagt, welche Angabe
fehlt:

```json
{ "field": "items[0]",
  "message": "Steuerkategorie konnte aus den übergebenen Angaben nicht bestimmt werden (BT-151). Bitte tax_category_code selbst setzen — oder für eine grenzüberschreitende Warenlieferung supply_type, shipping.country und delivery_date bzw. den Abrechnungszeitraum mitgeben." }
```

Das betrifft auch **inländische Nullsatz-Positionen ohne ausreichende
Angaben**. Ein echter Nullsatz-Umsatz benötigt eine ausdrückliche Kategorie
`Z` oder eine anwendbare Rechtsgrundlage; der Satz allein erklärt ihn nicht.

> Eine ausdrücklich gesetzte Kategorie gewinnt immer. Ein **positiver
> Steuersatz** wird nie in `K`/`G` umgedeutet — diese Kategorien verlangen
> 0 % (BR-IC-*, BR-G-*).

### Die USt-IdNr. muss eine sein

Für `K` wird das Format nach EN 16931 geprüft und das Länderpräfix gegen das
Käuferland gehalten; eine beliebige Zeichenkette genügt nicht.

**Taxonomie und Verifikation sind getrennte Zustände.** Das VIES-Ergebnis
ändert die Kategorie in **keinem** Fall — weder ein Ausfall noch ein
Negativverdikt. Der Sachverhalt bestimmt, was die Position ist; die
Nummernprüfung entscheidet, ob das Dokument entstehen darf:

| VIES | Kategorie | Ergebnis |
|---|---|---|
| `valid` | `K` | Rechnung entsteht |
| nicht erreichbar | `K` | Rechnung entsteht, Warnung `K_VAT_ID_UNCHECKED` |
| „nicht registriert" | `K` | **Ablehnung** mit `K_VAT_ID_INVALID` |

Die mittlere Zeile ist der Grund für die Trennung: hinge die Kategorie am
VIES-Zustand, ergäbe derselbe Payload je nach Tagesform in Brüssel ein anderes
Dokument.

### Nordirland (XI, Brexit-Protokoll)

Warenlieferungen folgen dem EU-Pfad (`K`), Dienstleistungen dem Drittlandpfad
— und werden dort aus demselben Grund wie oben nicht automatisch eingeordnet.
`supply_type` der Position schlägt immer die Kundenangabe `ni_supply_type`;
fehlt er, gilt die Kundenangabe, und bei `mixed` bleibt die Position
unbestimmt.

## Dokumentweite Steueraufstellung (BG-23)

Zusätzlich zur positionsbezogenen Steuer (BG-27/28) erzeugt Factora die
zusammengefasste **Steueraufstellung je Satz und Kategorie** (BG-23). Die
Summen werden aus den Positionen exakt (Dezimal, keine Float-Rundung)
berechnet und gegen die mitgelieferten Kontrollsummen (`validation_totals`)
geprüft.

## Prüfregeln — was eine Rechnung blockiert

Die Prüfung läuft auf **jedem schreibenden Weg**: `POST /api/v1/invoices/atomic/`
(live und Sandbox) und `POST /api/v1/invoices/{id}/finalize/`. Sie greift, bevor
ein Dokument entsteht — bei einem Treffer wird nichts gespeichert und nichts
archiviert. Sie ist Teil der Finalisierungsoperation selbst, nicht des
Endpunkts (B-AP90): auch interne Wege, die eine Rechnung final oder
freigegeben setzen, laufen durch dieselbe Prüfung.

Jeder Fehler kommt mit **seinem** Code im `errors[]` der Antwort, nicht als
Sammelcode, und mit `field: "items[n]"` auf die betroffene Position:

```json
{
  "valid": false,
  "data": null,
  "errors": [
    {
      "code": "K_VAT_ID_INVALID",
      "severity": "error",
      "field": "items[0]",
      "message": "USt-IdNr ATU12345678 ist laut VIES nicht gültig."
    }
  ],
  "meta": {}
}
```

Folgende Steuer-Prüfungen verhindern die Finalisierung (HTTP 400):

| Fehlercode | Kategorie | Bedingung |
|---|---|---|
| `K_VAT_ID_MISSING` | K | Käufer ohne USt-IdNr. |
| `K_COUNTRY_NOT_EU` | K | Käufer nicht in der EU |
| `K_COUNTRY_DOMESTIC` | K | Käufer im selben Land wie Verkäufer |
| `K_VAT_ID_INVALID` | K | USt-IdNr. laut VIES ungültig |
| `AE_CUSTOMER_TYPE_B2C` | AE | Reverse Charge an Endverbraucher |
| `AE_VAT_ID_MISSING` | AE | Käufer ohne USt-IdNr. (BR-AE-02) |
| `AE_BT120_MISSING` | AE | Befreiungstext fehlt (BR-AE-10) |
| `G_COUNTRY_NOT_THIRD` | G | Käufer innerhalb der EU |
| `O_COUNTRY_NOT_THIRD` | O | Käufer innerhalb der EU |
| `E_KU_BT120_MISSING` | E | Kleinunternehmer ohne Befreiungstext |
| `E_KU_BT121_SET` | E | Kleinunternehmer mit gesetztem VATEX-Code |

Nicht-blockierende Warnungen (HTTP 200/201, in `meta.warnings`):

| Warncode | Kategorie | Bedingung |
|---|---|---|
| `K_VAT_ID_UNCHECKED` | K | VIES-Dienst nicht erreichbar |
| `K_DELIVERY_PROOF_MISSING` | K | Gelangensnachweis noch nicht erfasst |

```json
{"meta": {"warnings": [
  {"code": "K_VAT_ID_UNCHECKED", "severity": "warning", "field": "items[0]",
   "message": "USt-IdNr konnte nicht bei VIES geprüft werden — Service nicht erreichbar."}
]}}
```

### VIES ist bewusst fail-open

`K_VAT_ID_INVALID` heißt: VIES hat geantwortet und die Nummer ist **nicht
registriert**. Nur das blockiert. Ist VIES nicht erreichbar — Wartung,
Netzfehler, ein einzelner Mitgliedstaat offline — entsteht die Rechnung
trotzdem und Sie bekommen `K_VAT_ID_UNCHECKED`. Ein Ausfall in Brüssel legt
Ihre Fakturierung nicht still.

Das Ergebnis wird am Kunden gespeichert und für 24 Stunden wiederverwendet.
Es hängt an der geprüften Nummer: sobald Sie eine andere USt-IdNr. schicken,
wird neu geprüft — auch innerhalb der 24 Stunden.
Eine Plattform, die denselben Käufer täglich abrechnet, löst also nicht je
Rechnung einen VIES-Aufruf aus.

## Einfrieren bei Finalisierung (GoBD)

Steuerkategorie, Satz und Befreiungsgründe werden bei der Finalisierung als
Snapshot **eingefroren**, damit XML und PDF jederzeit reproduzierbar bleiben.
Siehe [Compliance & Archivierung](compliance.md).
