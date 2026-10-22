<p align="center">
  <img src="assets/pengu-brief.png" alt="Pengu-Brief logo" width="150">
</p>

<h1 align="center">Pengu-Brief</h1>

<p align="center">
  <b>Boil projects down to the essentials – for you and your AI agents.</b><br>
  <b>Projekte auf das Wesentliche eindampfen – für dich und deine KI-Agenten.</b>
</p>

<p align="center">
  <a href="#english">English</a> · <a href="#deutsch">Deutsch</a>
</p>

<table>
  <tr>
    <td><img src="docs/screenshot-en-run.png" alt="Summarize tab" width="400"></td>
    <td><img src="docs/screenshot-en-preview.png" alt="Preview tab" width="400"></td>
  </tr>
  <tr>
    <td><img src="docs/screenshot-en-agent.png" alt="AI agent tab" width="400"></td>
    <td><img src="docs/screenshot-light-preview.png" alt="Light theme" width="400"></td>
  </tr>
</table>

---

# English

Pengu-Brief turns a project folder into a **compact Markdown overview** – structure, entry points, classes and function signatures, dependencies. Give that file to a (local) LLM or an AI coding agent so it does not have to read every file again at the start of each session.

It ships as a **single AppImage** with three faces: a GUI, a command line tool and an **MCP server**. Everything runs offline.

## Why?

Local LLMs are limited by context size and speed, and long prompts are the slow part. Instead of letting an agent crawl through hundreds of files, hand it a one-page map of the project. A typical overview is only a few hundred to a few thousand tokens.

## Features

- **GUI** with dark/light theme and a **German/English switch** (flag button, top right)
- **Command line** for scripts and agents with shell access
- **MCP server** (`--mcp`) so agents can call a `summarize_folder` tool directly
- **One-click agent setup**: writes a short instruction block into `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` or `.cursorrules` – existing text stays untouched
- **Code understanding**
  - Python via AST: module docstrings, function signatures, classes with their public methods
  - JavaScript, TypeScript, Go, Rust, Java, Kotlin, C#: names of top-level declarations
  - Other text files are counted in the statistics
- **Project metadata**: README, `package.json`, `pyproject.toml`, `requirements.txt`, Makefile, Dockerfile / docker-compose, `go.mod`, `Cargo.toml`, `pom.xml`, `composer.json`, `Gemfile`
- **Sensible ignores**: `.git`, `node_modules`, virtual environments, build output, lock files, binaries, files over 1 MB and patterns from your `.gitignore` (simple matching, no `!` negation)
- **Optional LLM introduction** through any OpenAI-compatible API (Ollama, LM Studio, llama.cpp server). Only the generated overview is sent, never your source files.
- Remembers your settings in `~/.config/pengu-brief/config.json`

## Quick start

1. Download `Pengu-Brief-x86_64.AppImage` from the [Releases](../../releases) page, or [build it yourself](#build-from-source).
2. Make it executable and start it:
   ```bash
   chmod +x Pengu-Brief-x86_64.AppImage
   ./Pengu-Brief-x86_64.AppImage
   ```
3. Choose a **project folder** and an **output folder**, then click **Summarize**. The result opens in the *Preview* tab and is saved as `PROJECT_SUMMARY.md` (or the file name you pick, e.g. `CLAUDE.md`).

Example output (shortened):

```markdown
# Project: my-app

_Generated automatically with Pengu-Brief: 11 files, approx. 87 lines._

## Languages
- Python: 6 files, 50 lines
- Markdown: 2 files, 20 lines

## Project info
- **README.md:** Invoice manager for freelancers: create customers, send invoices, track payments.
- **requirements.txt:** flask, sqlalchemy, pydantic, pytest
- **Makefile-Targets:** `make run`, `make test`

## Entry points
- `main.py`

## Modules and interfaces
- `app/models.py` (13 lines) - Database models.
    - class Customer: full_name, outstanding_amount
    - class Invoice: total_cents, mark_paid, is_overdue
- `app/services/billing.py` (10 lines) - Billing logic: totals, taxes and reminders.
    - class BillingService: create, send_reminder
    - def calculate_tax(net_cents: int, rate: float=0.2) -> int
```

## Use it with an AI coding agent

**Option 1 – agent file (easiest).** Put the AppImage into your project folder and run:

```bash
./Pengu-Brief-x86_64.AppImage --init-agent .            # writes AGENTS.md
./Pengu-Brief-x86_64.AppImage --init-agent . --file CLAUDE.md --lang en
```

(or use the *AI agent* tab in the GUI). This adds a clearly marked block to the file that tells the agent the tool exists, how to call it, and to read the overview before opening many files. Running it again updates the block instead of duplicating it. The agent needs permission to run shell commands.

| Agent | Typical file |
|---|---|
| Claude Code | `CLAUDE.md` |
| Codex, Copilot and many others | `AGENTS.md` |
| Gemini CLI | `GEMINI.md` |
| Cursor | `AGENTS.md` or `.cursorrules` |

File names are conventions of the respective tools – check their documentation.

**Option 2 – pre-generated file.** Create the overview once and let the agent load it as its context file:

```bash
./Pengu-Brief-x86_64.AppImage . -o CLAUDE.md
```

Regenerate it when the project structure changes noticeably.

**Option 3 – MCP server.** Print the configuration and paste it into `.mcp.json` (Claude Code) or `mcp.json` (Cursor):

```bash
./Pengu-Brief-x86_64.AppImage --mcp-config
```

```json
{
  "mcpServers": {
    "pengu-brief": {
      "command": "/path/to/Pengu-Brief-x86_64.AppImage",
      "args": ["--mcp"],
      "env": { "APPIMAGE_EXTRACT_AND_RUN": "1" }
    }
  }
}
```

The server talks JSON-RPC over stdio and offers one tool, `summarize_folder`:

| Parameter | Type | Description |
|---|---|---|
| `path` | string | Project folder (default: current directory) |
| `max_depth` | integer | Depth of the directory tree (default 4) |
| `max_files` | integer | Max. code files with details (default 80) |
| `max_chars` | integer | Truncate the answer after this many characters (default 40000) |
| `lang` | `"de"` \| `"en"` | Language of the headings (default: automatic) |
| `output_file` | string | Optionally also write the result to this `.md`/`.txt` file |

## Tree depth and code files

Both sliders control how long the overview gets:

- **Tree depth** – how many folder levels are expanded in the *Structure* section. Deeper levels are collapsed into a line like `... (12 files)`.
- **Code files** – for how many code files the *Modules and interfaces* section lists classes and functions. Files are taken in alphabetical path order; the rest is summarized as `... 35 more code files without details`.

Defaults (depth 4, 80 files) suit small and medium projects. For large projects try depth 2–3 and 40–80 files. Aim for an overview below roughly 5,000 tokens – the Preview tab shows the size.

## Command line

```text
./Pengu-Brief-x86_64.AppImage [FOLDER] [options]
```

| Option | Description | Default |
|---|---|---|
| `FOLDER` | Project folder | `.` |
| `-o`, `--output FILE` | Output file (relative to the folder) | `PROJECT_SUMMARY.md` |
| `--stdout` | Print instead of writing a file | |
| `--max-depth N` | Depth of the directory tree | 4 |
| `--max-entries N` | Max. entries per folder in the tree | 25 |
| `--max-files N` | Max. code files with details | 80 |
| `--max-items N` | Max. functions/classes per file | 12 |
| `--lang de\|en\|auto` | Language of the output | `auto` |
| `--llm` | Add an introduction written by an LLM | off |
| `--base-url URL` | OpenAI-compatible API (env `OPENAI_BASE_URL`) | `http://localhost:11434/v1` |
| `--model NAME` | Model name (env `LLM_MODEL`) | `qwen3.6:35b-a3b` |
| `--api-key KEY` | API key (env `OPENAI_API_KEY`) | `none` |

Special modes:

| Mode | Description |
|---|---|
| *(no arguments)* | Start the GUI (needs a graphical session) |
| `--gui` | Force the GUI |
| `--mcp` | Start the MCP server on stdio |
| `--mcp-config` | Print the MCP configuration as JSON |
| `--init-agent [FOLDER] [--file NAME]` | Write the agent instruction block (default file `AGENTS.md`) |
| `--version` | Show the version |

Examples:

```bash
./Pengu-Brief-x86_64.AppImage . --stdout
./Pengu-Brief-x86_64.AppImage ~/code/my-app -o ~/summaries/my-app.md --max-depth 3
./Pengu-Brief-x86_64.AppImage . --llm --model llama3.1:8b --lang en
```

## Language

- **GUI:** click the flag button in the top right corner. The choice is remembered.
- **Command line / MCP:** `--lang de|en`, otherwise the environment variable `PENGU_LANG`, otherwise your system language (`LANG`), otherwise English.
- The language also applies to the generated headings and to the agent instruction block.

## Requirements and notes

- **Linux x86_64.** The AppImage is built on Ubuntu 24.04 and therefore needs **glibc 2.38 or newer** (e.g. Ubuntu 24.04+, Fedora 39+, Debian 13+). To support older systems, build it on an older distribution.
- **FUSE 2** is needed to run AppImages. Without it, start with `APPIMAGE_EXTRACT_AND_RUN=1 ./Pengu-Brief-x86_64.AppImage` (slightly slower start).
- The GUI needs a graphical session (X11 or Wayland). The command line and the MCP server do not.
- If `zenity` or `kdialog` is installed, the native folder dialog is used; otherwise a simple Tk dialog.
- Code is analyzed statically and **never executed**. For JavaScript, TypeScript, Go, Rust, Java, Kotlin and C# only names are extracted (regex), so unusual declarations can be missed.
- The overview is a snapshot – regenerate it when the project changes.
- **Privacy:** nothing leaves your machine, except the optional LLM request to the URL you configure (default: `localhost`).

## Build from source

```bash
sudo apt install python3-tk python3-venv      # Debian/Ubuntu; other distros: install Tkinter
./build.sh                                    # creates Pengu-Brief-x86_64.AppImage
```

`build.sh` creates a virtual environment, bundles the app with PyInstaller and packs it with [appimagetool](https://github.com/AppImage/appimagetool) (downloaded on first use). Use `PYTHON=/path/to/python ./build.sh` to pick another interpreter.

Run from source without building:

```bash
pip install customtkinter pillow
python src/app_main.py                   # GUI
python src/app_main.py . --stdout        # command line
```

Recreate the icon (needs Pillow): `python tools/make_icon.py assets/pengu-brief.png 512`

```text
pengu-brief/
├── assets/pengu-brief.png      logo / icon
├── docs/                       screenshots
├── src/
│   ├── app_main.py             entry point: GUI, CLI, MCP server
│   ├── pengu_gui.py            GUI (CustomTkinter), German/English
│   ├── summarize_folder.py     the summarizer
│   ├── agent_setup.py          AGENTS.md block and MCP configuration
│   └── i18n.py                 language detection
├── tools/make_icon.py          draws the icon
├── build.sh                    builds the AppImage
└── requirements.txt
```

<p align="right"><a href="#pengu-brief">↑ back to top</a> · <a href="#deutsch">Deutsch ↓</a></p>

---

# Deutsch

Pengu-Brief macht aus einem Projektordner eine **kompakte Markdown-Übersicht** – Struktur, Einstiegspunkte, Klassen und Funktionssignaturen, Abhängigkeiten. Gib diese Datei einem (lokalen) LLM oder einem KI-Coding-Agenten, damit er zu Beginn jeder Sitzung nicht wieder alle Dateien lesen muss.

Das Programm ist ein **einzelnes AppImage** mit drei Gesichtern: eine Oberfläche, ein Kommandozeilenwerkzeug und ein **MCP-Server**. Alles läuft offline.

## Wozu?

Lokale LLMs sind durch Kontextgröße und Geschwindigkeit begrenzt, und lange Prompts sind der langsame Teil. Statt einen Agenten durch Hunderte Dateien zu schicken, bekommt er eine einseitige Karte des Projekts. Eine typische Übersicht umfasst nur einige hundert bis wenige tausend Tokens.

<table>
  <tr>
    <td><img src="docs/screenshot-de-run.png" alt="Reiter Zusammenfassen" width="400"></td>
    <td><img src="docs/screenshot-de-preview.png" alt="Reiter Vorschau" width="400"></td>
  </tr>
  <tr>
    <td><img src="docs/screenshot-de-agent.png" alt="Reiter KI-Agent" width="400"></td>
    <td></td>
  </tr>
</table>

## Funktionen

- **Oberfläche** mit dunklem/hellem Design und **Deutsch/Englisch-Umschalter** (Flaggen-Button oben rechts)
- **Kommandozeile** für Skripte und Agenten mit Shell-Zugriff
- **MCP-Server** (`--mcp`), damit Agenten das Werkzeug `summarize_folder` direkt aufrufen können
- **Agent-Einrichtung mit einem Klick:** schreibt einen kurzen Anleitungsblock in `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` oder `.cursorrules` – vorhandener Text bleibt unberührt
- **Code-Verständnis**
  - Python per AST: Modul-Docstrings, Funktionssignaturen, Klassen mit ihren öffentlichen Methoden
  - JavaScript, TypeScript, Go, Rust, Java, Kotlin, C#: Namen der Deklarationen auf oberster Ebene
  - Weitere Textdateien fließen in die Statistik ein
- **Projekt-Metadaten:** README, `package.json`, `pyproject.toml`, `requirements.txt`, Makefile, Dockerfile / docker-compose, `go.mod`, `Cargo.toml`, `pom.xml`, `composer.json`, `Gemfile`
- **Sinnvolle Ausschlüsse:** `.git`, `node_modules`, virtuelle Umgebungen, Build-Ausgaben, Lock-Dateien, Binärdateien, Dateien über 1 MB und Muster aus deiner `.gitignore` (einfaches Matching, keine `!`-Negation)
- **Optionale LLM-Einleitung** über jede OpenAI-kompatible API (Ollama, LM Studio, llama.cpp-Server). Gesendet wird nur die erzeugte Übersicht, nie dein Quellcode.
- Merkt sich deine Einstellungen in `~/.config/pengu-brief/config.json`

## Schnellstart

1. Lade `Pengu-Brief-x86_64.AppImage` von der Seite [Releases](../../releases) herunter oder [baue es selbst](#selbst-bauen).
2. Ausführbar machen und starten:
   ```bash
   chmod +x Pengu-Brief-x86_64.AppImage
   ./Pengu-Brief-x86_64.AppImage
   ```
3. Wähle einen **Projektordner** und einen **Ausgabeordner** und klicke auf **Zusammenfassen**. Das Ergebnis erscheint im Reiter *Vorschau* und wird als `PROJECT_SUMMARY.md` gespeichert (oder unter dem gewählten Namen, z. B. `CLAUDE.md`).

Beispielausgabe (gekürzt):

```markdown
# Projekt: my-app

_Automatisch erzeugt mit Pengu-Brief: 11 Dateien, ca. 87 Zeilen._

## Sprachen
- Python: 6 Dateien, 50 Zeilen
- Markdown: 2 Dateien, 20 Zeilen

## Projekt-Infos
- **README.md:** Invoice manager for freelancers: create customers, send invoices, track payments.
- **requirements.txt:** flask, sqlalchemy, pydantic, pytest
- **Makefile-Targets:** `make run`, `make test`

## Einstiegspunkte
- `main.py`

## Module und Schnittstellen
- `app/models.py` (13 Zeilen) - Database models.
    - class Customer: full_name, outstanding_amount
    - class Invoice: total_cents, mark_paid, is_overdue
- `app/services/billing.py` (10 Zeilen) - Billing logic: totals, taxes and reminders.
    - class BillingService: create, send_reminder
    - def calculate_tax(net_cents: int, rate: float=0.2) -> int
```

## Mit einem KI-Coding-Agenten nutzen

**Variante 1 – Agentendatei (am einfachsten).** Lege das AppImage in deinen Projektordner und führe aus:

```bash
./Pengu-Brief-x86_64.AppImage --init-agent .            # schreibt AGENTS.md
./Pengu-Brief-x86_64.AppImage --init-agent . --file CLAUDE.md --lang de
```

(oder nutze den Reiter *KI-Agent* in der Oberfläche). Das ergänzt die Datei um einen deutlich markierten Block, der dem Agenten sagt, dass das Werkzeug existiert, wie er es aufruft und dass er die Übersicht lesen soll, bevor er viele Dateien öffnet. Ein erneuter Aufruf aktualisiert den Block, statt ihn zu verdoppeln. Der Agent braucht die Erlaubnis, Shell-Befehle auszuführen.

| Agent | Übliche Datei |
|---|---|
| Claude Code | `CLAUDE.md` |
| Codex, Copilot und viele andere | `AGENTS.md` |
| Gemini CLI | `GEMINI.md` |
| Cursor | `AGENTS.md` oder `.cursorrules` |

Die Dateinamen sind Konventionen der jeweiligen Werkzeuge – prüfe deren Dokumentation.

**Variante 2 – vorab erzeugte Datei.** Erzeuge die Übersicht einmal und lass den Agenten sie als Kontextdatei laden:

```bash
./Pengu-Brief-x86_64.AppImage . -o CLAUDE.md
```

Erzeuge sie neu, wenn sich die Projektstruktur spürbar ändert.

**Variante 3 – MCP-Server.** Gib die Konfiguration aus und füge sie in `.mcp.json` (Claude Code) oder `mcp.json` (Cursor) ein:

```bash
./Pengu-Brief-x86_64.AppImage --mcp-config
```

```json
{
  "mcpServers": {
    "pengu-brief": {
      "command": "/pfad/zu/Pengu-Brief-x86_64.AppImage",
      "args": ["--mcp"],
      "env": { "APPIMAGE_EXTRACT_AND_RUN": "1" }
    }
  }
}
```

Der Server spricht JSON-RPC über stdio und bietet ein Werkzeug, `summarize_folder`:

| Parameter | Typ | Beschreibung |
|---|---|---|
| `path` | string | Projektordner (Standard: aktuelles Verzeichnis) |
| `max_depth` | integer | Tiefe des Verzeichnisbaums (Standard 4) |
| `max_files` | integer | Max. Code-Dateien mit Details (Standard 80) |
| `max_chars` | integer | Antwort nach so vielen Zeichen kürzen (Standard 40000) |
| `lang` | `"de"` \| `"en"` | Sprache der Überschriften (Standard: automatisch) |
| `output_file` | string | Ergebnis zusätzlich in diese `.md`/`.txt`-Datei schreiben |

## Baumtiefe und Code-Dateien

Beide Regler bestimmen, wie lang die Übersicht wird:

- **Baumtiefe** – wie viele Ordnerebenen im Abschnitt *Struktur* aufgeklappt werden. Tiefere Ebenen werden zu einer Zeile wie `... (12 Dateien)` zusammengefasst.
- **Code-Dateien** – für wie viele Code-Dateien der Abschnitt *Module und Schnittstellen* Klassen und Funktionen auflistet. Die Dateien werden in alphabetischer Pfadreihenfolge genommen, der Rest erscheint als `... 35 weitere Code-Dateien ohne Details`.

Die Standardwerte (Tiefe 4, 80 Dateien) passen für kleine und mittlere Projekte. Bei großen Projekten probiere Tiefe 2–3 und 40–80 Dateien. Halte die Übersicht unter etwa 5.000 Tokens – der Reiter Vorschau zeigt die Größe.

## Kommandozeile

```text
./Pengu-Brief-x86_64.AppImage [ORDNER] [Optionen]
```

| Option | Beschreibung | Standard |
|---|---|---|
| `ORDNER` | Projektordner | `.` |
| `-o`, `--output DATEI` | Ausgabedatei (relativ zum Ordner) | `PROJECT_SUMMARY.md` |
| `--stdout` | Ausgeben statt Datei schreiben | |
| `--max-depth N` | Tiefe des Verzeichnisbaums | 4 |
| `--max-entries N` | Max. Einträge pro Ordner im Baum | 25 |
| `--max-files N` | Max. Code-Dateien mit Details | 80 |
| `--max-items N` | Max. Funktionen/Klassen pro Datei | 12 |
| `--lang de\|en\|auto` | Sprache der Ausgabe | `auto` |
| `--llm` | Einleitung von einem LLM schreiben lassen | aus |
| `--base-url URL` | OpenAI-kompatible API (Umgebungsvariable `OPENAI_BASE_URL`) | `http://localhost:11434/v1` |
| `--model NAME` | Modellname (Umgebungsvariable `LLM_MODEL`) | `qwen3.6:35b-a3b` |
| `--api-key KEY` | API-Schlüssel (Umgebungsvariable `OPENAI_API_KEY`) | `none` |

Sondermodi:

| Modus | Beschreibung |
|---|---|
| *(keine Argumente)* | GUI starten (braucht eine grafische Sitzung) |
| `--gui` | GUI erzwingen |
| `--mcp` | MCP-Server über stdio starten |
| `--mcp-config` | MCP-Konfiguration als JSON ausgeben |
| `--init-agent [ORDNER] [--file NAME]` | Anleitungsblock für Agenten schreiben (Standarddatei `AGENTS.md`) |
| `--version` | Version anzeigen |

Beispiele:

```bash
./Pengu-Brief-x86_64.AppImage . --stdout
./Pengu-Brief-x86_64.AppImage ~/code/my-app -o ~/summaries/my-app.md --max-depth 3
./Pengu-Brief-x86_64.AppImage . --llm --model llama3.1:8b --lang de
```

## Sprache

- **Oberfläche:** Klicke auf den Flaggen-Button oben rechts. Die Wahl wird gespeichert.
- **Kommandozeile / MCP:** `--lang de|en`, sonst die Umgebungsvariable `PENGU_LANG`, sonst deine Systemsprache (`LANG`), sonst Englisch.
- Die Sprache gilt auch für die erzeugten Überschriften und den Anleitungsblock für Agenten.

## Voraussetzungen und Hinweise

- **Linux x86_64.** Das AppImage wird unter Ubuntu 24.04 gebaut und braucht deshalb **glibc 2.38 oder neuer** (z. B. Ubuntu 24.04+, Fedora 39+, Debian 13+). Für ältere Systeme baust du es auf einer älteren Distribution.
- Für AppImages wird **FUSE 2** benötigt. Ohne FUSE startest du mit `APPIMAGE_EXTRACT_AND_RUN=1 ./Pengu-Brief-x86_64.AppImage` (etwas langsamerer Start).
- Die Oberfläche braucht eine grafische Sitzung (X11 oder Wayland). Kommandozeile und MCP-Server brauchen sie nicht.
- Ist `zenity` oder `kdialog` installiert, wird der Ordnerdialog deines Desktops benutzt, sonst ein einfacher Tk-Dialog.
- Code wird nur statisch analysiert und **nie ausgeführt**. Bei JavaScript, TypeScript, Go, Rust, Java, Kotlin und C# werden nur Namen gelesen (Regex), ungewöhnliche Deklarationen können daher fehlen.
- Die Übersicht ist eine Momentaufnahme – erzeuge sie neu, wenn sich das Projekt ändert.
- **Datenschutz:** Nichts verlässt deinen Rechner, außer der optionalen LLM-Anfrage an die URL, die du einstellst (Standard: `localhost`).

## Selbst bauen

```bash
sudo apt install python3-tk python3-venv      # Debian/Ubuntu; andere Distributionen: Tkinter installieren
./build.sh                                    # erzeugt Pengu-Brief-x86_64.AppImage
```

`build.sh` legt eine virtuelle Umgebung an, bündelt die App mit PyInstaller und verpackt sie mit [appimagetool](https://github.com/AppImage/appimagetool) (wird beim ersten Mal heruntergeladen). Mit `PYTHON=/pfad/zu/python ./build.sh` wählst du einen anderen Interpreter.

Ohne Bauen direkt aus dem Quellcode starten:

```bash
pip install customtkinter pillow
python src/app_main.py                   # Oberfläche
python src/app_main.py . --stdout        # Kommandozeile
```

Icon neu erzeugen (braucht Pillow): `python tools/make_icon.py assets/pengu-brief.png 512`

<p align="right"><a href="#pengu-brief">↑ nach oben</a> · <a href="#english">English ↑</a></p>
