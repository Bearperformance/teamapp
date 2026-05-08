# CLAUDE.md — Bear Performance Team-App

Anweisungen für jeden Claude-Code-Aufruf in diesem Repository.

---

## Projekt-Kontext

**Bear Performance** — Trainings- und Anwesenheits-Management für Jugend-Fußballteams (U10–U18).

- **Architektur:** Single-File HTML-App (`index.html`, ~2500 Zeilen). Vanilla JS, kein Build-Schritt.
- **Backend:** Firebase Firestore. Projekt-ID: `bear-performance`. Collections: `coaches`, `teams`, `players`, `trainings`, `attendance`, `longstatus`.
- **Hosting:** GitHub Pages. Push auf `main` deployt automatisch.
- **Pages-URL:** https://bearperformance.github.io/teamapp/
- **Repo:** Bearperformance/teamapp
- **Test-Umgebung:** iOS Safari (mobile-first). Cache-Bust-Parameter `?v=N` zwingend für Verify-Checks.

**Sprache der User-Kommunikation:** Deutsch. Code-Kommentare können Englisch oder Deutsch sein.

---

## Workflow-Regeln (zwingend)

### A — Vor jedem Patch

1. **CONTEXT.md zuerst lesen** — enthält das Funktions-/Zeilen-Inventar der `index.html`.
2. **`git pull` ausführen**, falls länger nicht im Repo gewesen.
3. **Niemals die ganze `index.html` mit `Read` laden** — erst `Grep` für gezielten Sprung, dann `Read` mit Zeilen-Range.

### B — Beim Editieren

4. **Patches nur via `Edit` (str_replace)** — niemals Voll-Rewrite via `Write`.
5. **Pro Funktion ein Edit**, mehrere kleine sind sauberer als einer großer.
6. **Versionsnummer im Header genau dann hochsetzen, wenn ein Code-Patch im selben Edit-Lauf existiert.** Versions-Bump allein ist verboten.

### C — Nach jedem Patch (Verifikation)

7. **`verifier`-Subagent aufrufen** (siehe `.claude/agents/verifier.md`). Wartet auf dessen ✅, bevor weitergegangen wird.
8. **Bei ❌ vom Verifier:** Patch zurückrollen, Ursache analysieren, neu versuchen.

### D — Deployment

9. **Wenn alles ✅:** `deployer`-Subagent aufrufen (siehe `.claude/agents/deployer.md`).
10. **Niemals selbst `git push --force` oder `git reset --hard`** ausführen.

### E — Kommunikation mit dem User

11. **Code geht in die Datei**, nicht in die Chat-Antwort.
12. **Bei Mehrdeutigkeit nachfragen**, statt zu raten — User-Korrekturen sind teurer als Klarstellungs-Fragen.
13. **Keine Selbst-Entschuldigungs-Schleifen.** Fehler benennen, fixen, weiter.
14. **Tabellarische Verifikations-Reports** sind das Standard-Format nach jedem Patch.

---

## Wiederkehrende Bug-Muster (zur Vermeidung)

| Muster | Detail |
|---|---|
| **Kurze Aliase existieren nicht** | `showL`, `hideL`, `fbDel` sind nirgends definiert. Nur `showLoading`, `hideLoading`, `fbDelete`. Bei jedem Edit prüfen, dass keine Kurzform reinrutscht. |
| **Login-Robustheit** | `doLogin`-Body komplett in try/catch. `loadAll()` mit `Promise.race`-Timeout (12 s). Setup-Calls (`setupAllTeamsFromExcel`, `setupU14IfNeeded`, `setupU17Training`) NICHT-blockierend in `setTimeout` NACH dem Login. |
| **DEV-Bypass-Konstante** | `const DEV_BYPASS_LOGIN = true` füllt Login-Felder automatisch und fügt einen roten Banner ein. **Vor Launch:** auf `false` setzen — sonst bleibt der Banner für Endnutzer sichtbar. |
| **Spieler-Duplikate** | `setupAllTeamsFromExcel` und `setupTeamPlayers` können bei Namenskollision (Sonderzeichen, Whitespace) Duplikate erzeugen. `cleanupDuplicatePlayers(true)` läuft auto-silent nach Setup-Calls beim Admin-Login. Manueller Trigger: Admin-Button „🧹 Duplikate bereinigen". |
| **iOS Safari Caching** | Nach Deploy IMMER mit `?v=<Versionsnummer>` an die URL anhängen für ehrliche Verify-Tests. |

---

## Versions-Disziplin

- Aktueller Stand: nach **v9.7-delete-fix** (per Memory dokumentiert).
- v9.4 wurde übersprungen (kaputte Syntax-Fehler-Version aus früherem Chat).
- Versionsnummern strikt aufsteigend, nie wiederverwenden.
- Versions-Label im Header: `<span style="...">vX.Y</span>` — eindeutig per `grep '>vX\.Y<'` identifizierbar.

---

## Tools-Erwartung

- **`node --check`** — für JS-Syntax-Validierung des extrahierten `<script>`-Inhalts.
- **`gh` CLI** — für GitHub-Pages-Build-Status (`gh run list`, `gh run view`).
- **`git`** — Standard-Workflow: pull → edit → add → commit → push.
