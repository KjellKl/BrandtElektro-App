# SPEC.md · Elektro Brandt & Söhne

Quelle: Meeting 1 und 2 mit dem Inhaber Thomas Brandt (K4).

## Entitäten

8 Entitäten aus dem Grundmodell, dazu der Wartungsvertrag aus der neuen Anforderung (Meeting 2).

| Entität | Schlüssel | Attribute |
|---|---|---|
| Kunde | kunden_nr | name, adresse, kundentyp (Privat, Firma), stammkunde (ja, nein), kunde_seit |
| Auftrag | auftrag_nr | auftragsart (Installation, Reparatur, Notdienst, Wartung), beschreibung, eingegangen_am, faellig_am, termin, protokoll, status (offen, eingeplant, erledigt, abgerechnet) |
| Mitarbeiter | mitarbeiter_nr | name, qualifikation (Lehrling, Geselle, Meister), rolle (Inhaber, Planung, Büro, Monteur) |
| Einsatz | einsatz_id | datum, beginn, stunden, notdienst (ja, nein), zuschlag_erlassen (ja, nein) |
| Material | material_nr | bezeichnung, einheit, einkaufspreis, lagerbestand |
| Angebot | angebot_nr | betrag, erstellt_am, unterschrieben_kunde (ja, nein), unterschrieben_inhaber (ja, nein) |
| Rechnung | rechnungs_nr | rechnungsdatum, faellig_am, betrag, rabatt_gewaehrt (ja, nein), status (offen, 1. Mahnung, 2. Mahnung, Anwalt, bezahlt) |
| Werkzeug | werkzeug_nr | bezeichnung, standort (Werkstatt, Bus), entnommen_am |
| Wartungsvertrag (neu, Meeting 2) | vertrag_nr | jahrespauschale, beginn, letzte_pruefung_notlicht, letzte_pruefung_stromkasten |

Der Schlüssel identifiziert jede Entität eindeutig. Verweise auf andere Entitäten (z. B. zu welchem Kunden ein Auftrag gehört) stehen nicht als Attribut in der Tabelle, sondern unter Beziehungen. Fremdschlüssel kommen erst im Relationenmodell im Sprint.

## Beziehungen

Notation (min,max): Die Angabe steht neben der Entität und sagt, an wie vielen Beziehungen dieser Art eine einzelne Entität mindestens und höchstens beteiligt ist.

- **R1** Kunde (0,n) ─ erteilt ─ (1,1) Auftrag → **1:n**
  Ein Kunde kann beliebig viele Aufträge erteilen, jeder Auftrag gehört zu genau einem Kunden.
- **R2** Mitarbeiter (0,n) ─ wird eingesetzt bei ─ (0,m) Auftrag → **n:m**
  Ein Mitarbeiter arbeitet an vielen Aufträgen, an einem Auftrag können mehrere Mitarbeiter arbeiten. Aufgelöst über die Entität Einsatz, die die Beziehungsattribute trägt (datum, beginn, stunden, notdienst, zuschlag_erlassen).
- **R3** Auftrag (0,n) ─ verbraucht ─ (0,m) Material → **n:m**
  Ein Auftrag verbraucht viele Materialien, ein Material wird in vielen Aufträgen verbaut. Beziehungsattribute (Materialverbrauch): menge, erfasst_von, erfasst_am. Jeder erfasste Verbrauch senkt den Lagerbestand und landet auf der Rechnung (Nachr. 20).
- **R4** Auftrag (0,1) ─ hat ─ (1,1) Angebot → **1:1**
  Ein Auftrag hat höchstens ein Angebot (Pflicht ab 1.500 €, siehe BR4), jedes Angebot gehört zu genau einem Auftrag.
- **R5** Auftrag (0,1) ─ wird abgerechnet mit ─ (0,1) Rechnung → **1:1**
  Ein Auftrag bekommt höchstens eine Rechnung. Prüfaufträge aus einem Wartungsvertrag bekommen keine eigene Rechnung. Jede Rechnung gehört entweder zu genau einem Auftrag (R5) oder zu genau einem Wartungsvertrag (R9).
- **R6** Mitarbeiter (0,n) ─ hat im Bus ─ (0,1) Werkzeug → **1:n**
  Ein Mitarbeiter kann mehrere Werkzeuge haben, ein Werkzeug ist bei höchstens einem Mitarbeiter. Ohne Mitarbeiter liegt es in der Werkstatt.
- **R7** Kunde (0,n) ─ schließt ab ─ (1,1) Wartungsvertrag → **1:n**
  Nur Firmenkunden. Ein Kunde kann mehrere Wartungsverträge haben, jeder Vertrag gehört zu genau einem Kunden.
- **R8** Wartungsvertrag (0,n) ─ erzeugt ─ (0,1) Auftrag → **1:n**
  Aus einem Wartungsvertrag entstehen Prüfaufträge und die daraus folgenden Reparaturaufträge. Ein Auftrag gehört zu höchstens einem Wartungsvertrag, normale Aufträge zu keinem.
- **R9** Wartungsvertrag (0,n) ─ wird abgerechnet mit ─ (0,1) Rechnung → **1:n**
  Pro Vertragsjahr gibt es eine Rechnung über die Jahrespauschale.

## Business Rules

- **BR1 Lehrlinge nie allein:** Wenn ein Lehrling für einen Einsatz eingeplant wird, dann muss beim selben Auftrag zur selben Zeit ein Geselle oder Meister mit eingeplant sein (Nachr. 2).
- **BR2 Materialrabatt für Firmenkunden:** Wenn ein Firmenkunde Material über 5.000 € auf einem Auftrag hat, dann schlägt die App 20 % Rabatt vor, nur auf das Material und nie auf die Arbeit. Ob der Rabatt gewährt wird, entscheidet der Inhaber (Nachr. 4, 5).
- **BR3 Notdienstzuschlag:** Wenn ein Einsatz nach 18 Uhr oder am Wochenende stattfindet, dann kommen 80 € Anfahrtspauschale dazu und der Stundensatz wird mit 1,5 multipliziert. Nur der Inhaber kann den Zuschlag im Einzelfall erlassen (Nachr. 6, 7, 18).
- **BR4 Angebot ab 1.500 €:** Wenn ein Auftrag 1.500 € oder mehr kosten wird, dann braucht er vorher ein schriftliches Angebot, das Kunde und Inhaber unterschrieben haben. Ohne beide Unterschriften beginnt die Arbeit nicht (Nachr. 9).
- **BR5 Mahnstufen:** Wenn eine Rechnung nach 21 Tagen nicht bezahlt ist, dann geht die 1. Mahnung raus. Nach 35 Tagen folgt die 2. Mahnung, nach 49 Tagen geht der Fall an den Anwalt. Das Zahlungsziel ist 14 Tage (Nachr. 17).

**Rechengrundlage für jede Rechnung** (keine Wenn-dann-Regel, sondern die feste Formel, auf der BR2 und BR3 aufsetzen): Rechnungsbetrag = Stunden × Stundensatz + verbautes Material zum Einkaufspreis mit 30 % Aufschlag (Nachr. 3, 20).

## Widersprüche

- **W1 Notdienstzuschlag: fest oder verhandelbar?**
  - Aussage A (Nachr. 7): „Das ist fest so, daran rütteln wir nicht.“
  - Aussage B (Nachr. 18): Für langjährige Stammkunden macht Brandt doch eine Ausnahme.
  - Was gilt: Der Zuschlag wird immer automatisch berechnet (BR3). Nur der Inhaber kann ihn im Einzelfall manuell erlassen.
  - Annahme: Stammkunden bekommen den Erlass nicht automatisch. Der Inhaber entscheidet jedes Mal selbst.
- **W2 Wer plant die Einsätze?**
  - Aussage A (Nachr. 2): „Hab ich im Kopf, meine Frau manchmal auch.“
  - Aussage B (Nachr. 8): „Markus macht die Planung mit mir.“
  - Was gilt: Inhaber und Markus planen gemeinsam (Rolle Planung). Das Büro sieht die Planung nur (Nachr. 10).

## Rollen und Rechte

- Monteure sehen nur Einsatzort und Aufgabe, keine Löhne und keine Preise (Nachricht 8)
- Büro erfasst Aufträge, schreibt Rechnungen und Mahnungen, hält Kundenkontakt (Nachricht 8, 16)
- Planung (Inhaber und Markus) teilt die Einsätze ein und sieht alles (Nachricht 8, 10)
- Nur der Inhaber unterschreibt Angebote, entscheidet über Rabatte und erlässt Notdienstzuschläge (Nachricht 5, 9, 18)

## Neue Anforderung (Meeting 2)

Wartungsverträge für Firmenkunden (Nachricht 11, 14 bis 16):

- Notbeleuchtung wird jährlich geprüft, der Stromkasten alle 2 Jahre
- Der Kunde zahlt eine Jahrespauschale für Prüfen und Protokoll
- Spätestens 14 Tage vor einer fälligen Prüfung erinnert die App das Büro und die Planung, damit ein Termin mit dem Kunden vereinbart werden kann
- Jede Prüfung bekommt ein Protokoll, damit sichtbar ist, was beim letzten Mal gemacht wurde
- Defekte, die bei einer Prüfung auffallen, werden als eigener Reparaturauftrag angelegt

**Klärung zur Abrechnung:** Brandt spricht von einer „Jahrespauschale“, sagt aber auch „nach jedem Termin automatisch eine Rechnung“ (Nachr. 14). Annahme: Die Pauschale wird einmal im Jahr über den Wartungsvertrag abgerechnet, Prüftermine bekommen keine eigene Rechnung. Reparaturen, die bei einer Prüfung auffallen, sind ein neuer Auftrag und werden extra abgerechnet (Nachr. 15). Brandt will das noch mit Markus klären, siehe offene Fragen.

## Prioritäten

**v1**

1. Aufträge erfassen statt Zettel
2. Einsätze planen: wer hat welchen Auftrag, wer ist wo
3. Abrechnung mit Materialverbrauch, Zuschlägen, Rabatt und Mahnstufen
4. Angebote ab 1.500 €

**Später**

5. Wartungsverträge mit Erinnerung
6. Werkzeug-Übersicht

## Offene Fragen und Annahmen

- Stundensatz: Brandt nennt keinen Betrag (Nachr. 17). Annahme: Der Stundensatz ist in der App einstellbar, eventuell getrennt nach Qualifikation
- Rabattgrenze 5.000 €: auf Einkaufspreis oder Preis mit Aufschlag? Annahme: Einkaufspreis
- Notdienst: Ab wann morgens gilt wieder der normale Satz, zählen Feiertage? Annahme: 18 bis 7 Uhr, Feiertage wie Wochenende
- Wartungsvertrag: Abrechnung pro Jahr ist noch mit Markus zu bestätigen (Nachr. 15)
