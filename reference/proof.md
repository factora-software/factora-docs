# Signierter Prüfnachweis (F-AP136)

Zu jeder gültig geprüften Rechnung liefert Factora einen **signierten Nachweis**
in `data.proof` — bei Atomic (live und Sandbox), bei Convert (nur wenn KoSIT das
Dokument akzeptiert hat) und beim Inbound-Check (gültig oder ungültig). Ein Key
ohne konfigurierten Signierschlüssel ändert nichts: das Feld fehlt dann.

## Was der Nachweis belegt

> „Dokument mit dem SHA-256 *X* wurde am *Y* mit dem Regelwerk *Z* geprüft;
> Ergebnis *gültig/ungültig*, Befunde nach `tax_relevance`.“

Mehr nicht. Er enthält **keine Rechnungsdaten**: nur Nachweis-ID, Hash des
ausgelieferten XML, Prüfzeitpunkt, Regelwerk (Validator-Image, Szenario,
Regelsatz-Artefakte wie in `data.validation.provenance`), Ergebnis, Anzahl der
Befunde je `tax_relevance`, Umgebung (`live`/`sandbox`) und ein Pseudonym des
Mandanten (`t_…`, nicht umkehrbar).

## Was er nicht belegt

- **Keine Rechtsgültigkeit der Forderung** und keine steuerliche Richtigkeit
  der Angaben — geprüft wird die Form gegen das Regelwerk, nicht der Sachverhalt.
- **Keine Archivierung.** Factora archiviert nicht und speichert Rechnungen nur bis zum Monatsexport (AGB 2.2, § 4 Abs. 6); der
  Nachweis ersetzt die Aufbewahrung nicht. Er ist nur zusammen mit der Datei
  aussagekräftig.
- **Kein Beleg für Sandbox-Dokumente.** Ein Sandbox-Nachweis trägt
  `env: "sandbox"`; das XML wird dort nie ausgeliefert.
- Beim Inbound-Check mit PDF bezieht sich der Hash auf das **eingebettete XML**,
  nicht auf die PDF-Datei.

## Format

Kompaktes **JWS** (RFC 7515), Algorithmus **EdDSA / Ed25519** (RFC 8037).
Gewählt, weil es ein offener Standard mit Bibliotheken in jeder Sprache ist,
die Signatur klein bleibt (64 Byte) und kein Aushandeln von Parametern nötig
ist. Header: `alg`, `typ: factora-proof+jws`, `kid`.

Die öffentlichen Schlüssel (JWKS) stehen unter
`https://console.factora.software/.well-known/factora-proof-keys.json` und
`/api/public/proof/keys/`. Bei einer Rotation bekommt der neue Schlüssel eine
neue `kid`; die alten bleiben im Set, damit ältere Nachweise prüfbar bleiben.

## Prüfen ohne Login

- Seite: `https://console.factora.software/pruefung` (XML und Nachweis hochladen).
- API: `POST /api/public/proof/verify/` (multipart: `file` = XML, `proof` = JWS;
  der Nachweis darf auch als letzter XML-Kommentar `<!-- factora-proof:JWS -->`
  eingebettet sein). Antwort `status`: `verified` (Signatur gültig, Hash passt),
  `modified` (Signatur gültig, Datei weicht ab) oder `invalid` (kein gültiger
  Nachweis). Es wird nichts gespeichert; Limit 30 Anfragen/Minute je IP.

Die Prüfung ist auch offline möglich: SHA-256 des XML bilden, JWS mit dem
öffentlichen Schlüssel zur `kid` prüfen, `sha256` im Payload vergleichen.

## Console

„Nachweise“ listet die Nachweise der Live-Aufrufe (Atomic, Convert) — Nachweis-ID,
Prüfzeitpunkt, Ergebnis, Hash — und lädt den Nachweis als `.jws`. Die Tabelle
enthält keine Rechnungsdaten. Sandbox-Läufe und der Inbound-Check hinterlassen
keine Zeile (beide speichern nichts).

## Betrieb (Produktion)

Ohne Schlüssel stellt Factora keine Nachweise aus. Nötig sind in der
Server-`.env` (nur der Betreiber setzt sie):

- `PROOF_SIGNING_KEY` — Ed25519-Seed, base64url, 32 Byte (z. B. mit
  `cryptography`: `Ed25519PrivateKey.generate().private_bytes(Raw, Raw, NoEncryption())`).
- `PROOF_SIGNING_KID` — Kennung, bei Rotation hochzählen (`factora-proof-2`).
- `PROOF_RETIRED_PUBLIC_KEYS` — bei Rotation: JSON-Liste der öffentlichen JWKs
  der alten Schlüssel.

Dazu die nginx-Änderung (Location für `/.well-known/factora-proof-keys.json`
und Pfad `pruefung`) und die Migration `api.0030`.
