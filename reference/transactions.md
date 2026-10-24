# Vorgänge — Rechnung aus Ereignissen (F-AP138)

Ein **Vorgang** (`transaction`) sammelt die Fakten eines Geschäftsvorfalls.
Aus den Fakten erzeugt Factora auf Abruf die Rechnung — über denselben Pfad
wie `POST /api/v1/invoices/atomic/` (gleiche Regeln, Summen, KoSIT-Prüfung,
Abrechnung).

## Was gespeichert wird — und was nicht

| Gespeichert (Fakten) | Nicht gespeichert |
|---|---|
| `service.completed`: Leistung, Menge, Einheit, Preis, Steuersatz/-kategorie | die Rechnung (XML, PDF, Summen) |
| `service.accepted`: wer, wann, Nachweis-Referenz | Rechnungskopf, Käufer, Verkäufer, Zahlungsdaten |
| `charge.agreed`: Dokumentrabatt/-zuschlag | Personendaten, die nicht in der Rechnung stehen müssen |
| Zeitpunkt, **wann** eine Rechnung zurückgegeben wurde | der Inhalt der zurückgegebenen Rechnung |

**Einschränkung bis zum Zustandslos-Umbau (F-ST04):** Der Vorgang selbst
hält keine Rechnung. Die Atomic-Pipeline darunter legt die finalisierte
Rechnung aber weiterhin an, und der Idempotenz-Speicher hält die Antwort für
die Wiederholung vor. „Nicht gespeichert“ gilt hier für den Vorgang, nicht
für das Gesamtsystem.

Fakten sind unveränderlich (append-only). Die Aufbewahrung der Rechnung
(GoBD) bleibt beim Kunden — Factora gibt sie zurück, mehr nicht.

## Endpunkte (Bearer-Key, Live)

- `POST /api/v1/transactions/{id}/events/` — Fakt anhängen (`Idempotency-Key`
  Pflicht). Body: `{"type": "...", "data": {...}}`. Der Vorgang entsteht mit
  dem ersten Fakt. `{id}` wählt der Integrator (1–64 Zeichen `A–Z a–z 0–9 . _ : -`).
  - `service.completed`: Felder einer atomic-Position plus `service_id`.
  - `service.accepted`: `service_id`, `accepted_by`, `accepted_at`, optional `evidence_ref`.
  - `charge.agreed`: Felder eines `header_charges`-Eintrags plus `charge_id`.
- `GET /api/v1/transactions/{id}/` — Fakten, `status`, `missing`.
- `POST /api/v1/transactions/{id}/invoice/` — Rechnung erzeugen. Body wie
  atomic, **ohne** `items` und `header_charges` (die kommen aus den Fakten);
  `validation_totals` prüft wie bei atomic die Summen. Antwort wie atomic.
  Nur bei `ready`, sonst 409. Danach nimmt der Vorgang keine Fakten mehr an.

## Status — deterministische Regel

- `open` — Fakten fehlen. `missing` nennt sie, z. B.
  „Abnahme fehlt für Leistung S2 (Inbetriebnahme).“ oder „Keine abgeschlossene Leistung erfasst.“
- `ready` — mindestens eine Leistung, und jede ist abgenommen.
- `invoiced` — eine Rechnung wurde zurückgegeben.

Kein Raten, keine KI: gleiche Fakten ergeben immer denselben Status.

Doppelte Fakten (gleicher Typ und gleiche `service_id`/`charge_id`) sind 409.
Die Console zeigt unter „Vorgänge“ Liste und Detail (Zeitleiste, Ampel).
