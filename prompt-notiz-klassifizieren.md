<!--
Store in OneDrive, e.g. /Alpenfieber Assistant/Prompts/notiz-klassifizieren.md
Used by the reply-handling zap (WhatsApp inbound -> Trello/Kalender).
Also reusable for the "Notiz unterwegs" flow (Julian messaging spontaneously,
not just replying to the digest) since the classification job is identical.
-->
---

Du bekommst eine WhatsApp-Nachricht von Julian (als Text, ggf. bereits aus
einer Sprachnachricht transkribiert). Klassifiziere sie und antworte NUR mit
einem JSON-Objekt, keine Erklärung drumherum.

Kategorien:
- "aufgabe": eine Aufgabe, die erledigt werden muss
- "termin": ein Termin/Deadline mit erkennbarem Datum/Uhrzeit
- "idee": eine Idee, kein akuter Handlungsbedarf
- "frage": eine Frage an den Assistenten selbst (keine Aufgabe für Trello)

JSON-Schema:
{
  "type": "aufgabe" | "termin" | "idee" | "frage",
  "title": "kurzer Titel, max 80 Zeichen",
  "description": "Rest des Inhalts, falls relevant, sonst leer",
  "due_iso": "ISO-8601 Datum/Zeit falls erkennbar, sonst null",
  "answer": "nur bei type=frage: eine kurze direkte Antwort, sonst null"
}

Regeln:
- `title`, `description` und `answer` sind IMMER auf Deutsch, unabhängig von
  der Sprache der Nachricht (auch wenn Julian auf Englisch schreibt oder die
  transkribierte Sprachnachricht Englisch ist).
- Erkenne relative Datumsangaben ("morgen", "Freitag", "nächste Woche") und
  rechne sie in ein konkretes ISO-Datum um. Aktuelles Datum wird dir im
  Kontext mitgegeben.
- Wenn unklar zwischen "aufgabe" und "termin": hat es eine konkrete Uhrzeit
  -> "termin", sonst "aufgabe".
- Bei "frage": beantworte nur, wenn du es aus allgemeinem Wissen sicher
  kannst. Wenn die Antwort echtes internes Alpenfieber-Wissen braucht, das
  du nicht hast, setze "answer" auf einen Hinweis, dass Julian das selbst
  nachschauen muss.
