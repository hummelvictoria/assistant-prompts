# Alpenfieber-Assistent: alle Agenten

Ein Unterordner pro Agent. Jeder enthält seinen Prompt, seinen Code und eine
eigene README mit den Zap-Schritten.

| Ordner | Was er tut | Zap | Läuft |
|---|---|---|---|
| `morgen-digest/` | sortierte Übersicht aus Mails, Terminen, Trello | Zap 1 | Mo–Fr 07:30 |
| `tagesabschluss/` | Kurzbilanz, offene Karten dringend auf morgen verschieben | Zap 3 | Mo–Fr 17:30 |
| `telefon-agent/` | Sprachaufnahmen (Telefonate, Notizen) auswerten und anlegen | Zap 2, Path A | bei WhatsApp-Audio |
| `notiz-textpfad/` | Textnachrichten als Aufgabe/Termin/Idee/Frage verarbeiten | Zap 2, Path B | bei WhatsApp-Text |
| `anmeldungen/` | Anmelde-Mails beantworten, erfassen, Nachfass-Karte anlegen | Zap 4 | bei neuer Mail |

Außerdem: `JULIAN-SETUP.md` ist die Anleitung für Julian (WhatsApp-Vorlagen,
Mail-Weiterleitung, Kalender-Sync, Aufnahmen schicken).

## Was ins GitHub-Repo kommt

Genau die fünf Prompt-Dateien, jede aus ihrem Agentenordner:

- `morgen-digest/prompt-morgen-digest.md`
- `tagesabschluss/prompt-tagesabschluss.md`
- `telefon-agent/prompt-anruf-extraktion.md`
- `notiz-textpfad/prompt-notiz-klassifizieren.md`
- `anmeldungen/prompt-anmeldung-verarbeiten.md`

Alles andere (`.js`, `.gs`) wird in Zapier bzw. Google eingefügt, nicht auf
GitHub gelegt. Die Prompt-Dateien enthalten nur den reinen Prompt-Text, du
kannst sie unverändert hochladen.

## Was bei Meta, Google und den Konten nötig ist

- **Zwei WhatsApp-Vorlagen**: `morgen_digest` und `tagesabschluss`. Ohne "Approved" kann der Assistent nicht von sich aus schreiben. **Text und Kategorie sind noch offen:** Meta stuft Vorlagen, die nur aus `{{1}}` bestehen, als Marketing ein, und für jede Variable braucht es einen Beispielwert. Die Angaben "Utility, nur `{{1}}`" in den Agenten-READMEs gelten deshalb nicht mehr, bis das entschieden ist.
- **Anthropic:** API Key und hinterlegte Zahlungsmethode.
- **OpenAI:** Key für die Transkription der Sprachaufnahmen (Telefon-Agent).
- **Trello:** Token mit **Julians** Konto autorisiert. Alles, was der Assistent anlegt oder verschiebt, liegt auf **Julians eigenem Board**, nicht im ON-STUDIO-Board. Dessen Kurzlink kommt als `board_id` in die Trello-Schritte.
- **Zapier:** bezahlter Plan (Code-Schritte brauchen mehr als 1 Sekunde, die WhatsApp-App ist Premium).
- **Apps Script:** `morgen-digest/apps-script.gs` auf script.google.com, `SHARED_SECRET` setzen.

## Zap 2 in einem Bild

```
WhatsApp: New Message Received
└─ Paths
   ├─ Path A  (Nachrichtentyp = audio)  → telefon-agent/
   └─ Path B  (Always run)              → notiz-textpfad/
```

## Getestet und nicht getestet

Alle Code-Dateien habe ich mit erfundenen Claude-Antworten ausgeführt
(Randfälle wie falsche Typen, fehlende Felder, leere Listen). Gegen die
echten Dienste (Anthropic, Trello, Zapier, WhatsApp, Gmail) lief nichts. Die
Prompts sind ungetestet, die Qualität der Ergebnisse zeigt sich erst beim
ersten echten Lauf.

## Offene Entscheidungen

- Gmail-Suchbegriff für Anmeldungen festlegen (`anmeldungen/README.md`).
- Sollen reine Infos aus Aufnahmen abgelegt werden? Aktuell nicht.
- Tagesabschluss: Beim ersten Test `dry_run` auf `ja` lassen und die Kandidatenliste prüfen, erst dann auf `nein` stellen.
