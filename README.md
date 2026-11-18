# Pulse

> Spokojny kokpit dla Twoich agentów AI.

[![License: MIT](https://img.shields.io/badge/License-MIT-5B8DEF?style=flat-square)](LICENSE)
[![Tauri](https://img.shields.io/badge/Tauri-2-24C8DB?style=flat-square&logo=tauri&logoColor=white)](https://tauri.app)
[![Svelte](https://img.shields.io/badge/Svelte-5-FF3E00?style=flat-square&logo=svelte&logoColor=white)](https://svelte.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-000000?style=flat-square&logo=rust&logoColor=white)](https://www.rust-lang.org)
[![ACP](https://img.shields.io/badge/ACP-compatible-3DD68C?style=flat-square)](https://agentclientprotocol.com)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-6E6E76?style=flat-square)]()

**Pulse** to lekka aplikacja desktopowa do monitorowania i kontrolowania lokalnie uruchomionych agentów AI, takich jak Claude Code, Gemini CLI, Codex czy OpenCode. Wszyscy agenci są widoczni w jednym miejscu, bez potrzeby otwierania ciężkiego IDE.

Aplikacja jest zbudowana w Rust (Tauri 2) i działa na Linuksie, macOS oraz Windows. Na Linuksie wygląda jak dopracowana, natywna aplikacja z ekosystemu macOS.

---

## Spis treści

- [Dlaczego Pulse](#dlaczego-pulse)
- [Funkcje](#funkcje)
- [Architektura](#architektura)
- [Szybki start](#szybki-start)
- [Konfiguracja](#konfiguracja)
- [Design](#design)
- [Stack technologiczny](#stack-technologiczny)
- [Roadmapa](#roadmapa)
- [Współtworzenie](#współtworzenie)
- [Licencja](#licencja)

---

## Dlaczego Pulse

W 2026 roku programiści często pracują z wieloma agentami kodującymi jednocześnie: jeden wykonuje code review, drugi generuje testy, a trzeci prowadzi research. Edytory takie jak Zed czy VS Code mają wbudowane panele agentów, ale są ściśle związane z IDE i przeciążone funkcjami, których nie potrzebujesz, gdy chcesz tylko sprawdzić, co robią Twoi agenci.

Pulse odpowiada na tę potrzebę: **chcesz widzieć i kontrolować swoich agentów, ale nie chcesz za każdym razem otwierać całego edytora.**

| Problem | Rozwiązanie w Pulse |
|---|---|
| Kilka terminali obok siebie | Jeden ekran ze wszystkimi agentami |
| Ciężkie panele w IDE | Lekka aplikacja desktopowa (ok. 10 MB) |
| Osobne integracje dla każdego dostawcy | Jeden standard: Agent Client Protocol |
| Przeładowane, krzykliwe interfejsy | Spokojny, minimalistyczny interfejs |
| Brak historii po restarcie | Zapis sesji w SQLite |

---

## Funkcje

| Funkcja | Opis | Status |
|---|---|---|
| Lista agentów | Wszyscy agenci w jednym miejscu, z kropką statusu: zielona, szara lub czerwona | ✅ |
| Widok szczegółów | Kliknięcie panelu rozwija strumień zdarzeń w tym samym miejscu | ✅ |
| Anulowanie sesji | Jeden skrót, gdy agent utknie w ślepym zaułku | ✅ |
| Zatwierdzanie uprawnień | Agent chce wykonać polecenie shellowe? Zatwierdź je bez wychodzenia z Pulse | 🚧 |
| Filtrowanie zdarzeń | Tylko błędy? Tylko wywołania narzędzi? Filtr działa we wszystkich panelach naraz | 🚧 |
| Persystencja sesji | SQLite zapisuje historię, dzięki czemu po restarcie widzisz stan sprzed zamknięcia | 🚧 |
| Eksport snapshotu | Stan wszystkich agentów do Markdown lub JSON | 📋 |
| Tryb zen | `⌘.` ukrywa wszystko poza listą agentów i kropkami statusu | 📋 |

Legenda: ✅ gotowe · 🚧 w toku · 📋 zaplanowane

---

## Architektura

Pulse nie opiera się na rozwiązaniach dedykowanych konkretnym dostawcom. Komunikuje się z agentami przez **[Agent Client Protocol (ACP)](https://agentclientprotocol.com)**, otwarty standard (JSON-RPC 2.0 po stdio) stworzony przez Zed. Dzięki niemu dowolny klient może w ustandaryzowany sposób rozmawiać z dowolnym agentem.

```
┌─────────────────────────────────────────────────────┐
│                      Pulse (Tauri)                  │
│                                                     │
│   ┌─────────────┐          ┌──────────────────┐    │
│   │  Frontend   │◄────────►│  Backend (Rust)  │    │
│   │  Svelte 5   │  invoke  │  tokio + ACP     │    │
│   └─────────────┘          └────────┬─────────┘    │
│                                     │              │
└─────────────────────────────────────┼──────────────┘
                                      │ JSON-RPC / stdio
                        ┌─────────────┼─────────────┐
                        ▼             ▼             ▼
                   ┌────────┐   ┌────────┐   ┌────────┐
                   │ Claude │   │ Gemini │   │ Codex  │
                   └────────┘   └────────┘   └────────┘
```

Kluczowa decyzja projektowa: cała logika (ACP, zarządzanie procesami, SQLite) znajduje się w Ruście, a frontend jedynie renderuje dane. Dzięki temu aplikacja pozostaje responsywna nawet przy dziesięciu równoległych agentach.

---

## Szybki start

### Wymagania

| Narzędzie | Wersja | Link |
|---|---|---|
| Rust | stable | [rust-lang.org/tools/install](https://www.rust-lang.org/tools/install) |
| Node.js | LTS | [nodejs.org](https://nodejs.org) |
| Tauri CLI | 2.x | [tauri.app/start](https://tauri.app/start/) |

Na Linuksie wymagane są zależności systemowe Tauri. Pełną listę znajdziesz w [oficjalnej dokumentacji](https://tauri.app/start/prerequisites/).

### Instalacja

```bash
git clone https://github.com/vpui/pulse.git
cd pulse
npm install
npm run tauri dev
```

### Budowanie wersji produkcyjnej

```bash
npm run tauri build
```

Gotowe pliki binarne znajdziesz w `src-tauri/target/release/bundle/`.

---

## Konfiguracja

Pulse wczytuje konfigurację z pliku `~/.config/pulse/agents.toml`. Każdy agent musi obsługiwać ACP. Listę kompatybilnych agentów znajdziesz w [ACP Registry](https://agentclientprotocol.com/get-started/registry).

```toml
[[agent]]
name = "Claude"
command = "npx"
args = ["@agentclientprotocol/claude-agent-acp"]
accent = "#5B8DEF"

[[agent]]
name = "Gemini"
command = "gemini"
args = ["--acp"]

[[agent]]
name = "Codex"
command = "codex"
args = ["acp"]
```

### Skróty klawiszowe

| Skrót | Akcja |
|---|---|
| `Tab` | Przełącz panel między agentami |
| `Enter` | Rozwiń lub zwiń szczegóły agenta |
| `c` | Anuluj sesję aktywnego agenta |
| `a` | Zatwierdź żądanie uprawnień |
| `f` | Otwórz filtr zdarzeń |
| `⌘.` | Tryb zen |
| `⌘,` | Ustawienia |
| `q` | Wyjście |

---

## Design

Pulse ma wyglądać jak spokojny kokpit: informacje są czytelne, a interfejs nie rozprasza.

### Paleta

| Rola | Kolor | HEX |
|---|---|---|
| Tło aplikacji | Prawie czarny | `#0A0A0B` |
| Tło panelu | Ciemny grafit | `#121214` |
| Obramowanie | Ledwo widoczne | `#1F1F23` |
| Tekst główny | Jasnoszary | `#E8E8EA` |
| Tekst wyciszony | Średnioszary | `#6E6E76` |
| Akcent | Stonowany błękit | `#5B8DEF` |
| Status: pracuje | Zielony | `#3DD68C` |
| Status: błąd | Czerwony | `#E5484D` |

### Zasady projektowe

- Obramowania mają grubość 1 px. Zaokrąglenia są spójne: 10 px dla paneli i 6 px dla przycisków.
- Zachowujemy dużo przestrzeni: minimum 16 px między sekcjami i 24 px marginesu wewnątrz paneli.
- Animacje trwają 180–220 ms i używają jednej krzywej `cubic-bezier(0.32, 0.72, 0, 1)`, zapożyczonej z macOS.
- Stawiamy na typografię zamiast ikon. Ikony pojawiają się tylko tam, gdzie są naprawdę potrzebne (Lucide, grubość linii 1.5 px).
- Nie używamy gradientów, poświaty ani neonów.

---

## Stack technologiczny

| Warstwa | Wybór | Uzasadnienie |
|---|---|---|
| Powłoka aplikacji | [Tauri 2](https://tauri.app) | ok. 10 MB zamiast ok. 150 MB w Electronie, start poniżej 300 ms |
| Backend | Rust (tokio, `serde_json`) | Wydajność, bezpieczeństwo, zarządzanie procesami |
| Klient ACP | Własna implementacja JSON-RPC 2.0 | Pełna kontrola i minimalne zależności |
| Frontend | [Svelte 5](https://svelte.dev) + Tailwind | Najmniejszy bundle i płynne animacje |
| Typografia | Inter + JetBrains Mono | Czytelny interfejs i czytelny kod |
| Persystencja | SQLite (`rusqlite`) | Działa lokalnie, bez zależności od chmury |
| Ikony | [Lucide](https://lucide.dev) | Cienka linia, spójny zestaw |

---

## Roadmapa

- [x] Klient ACP z presetami (Claude, Gemini, Codex)
- [x] Lista agentów i widok szczegółów
- [x] Anulowanie sesji
- [ ] Zatwierdzanie uprawnień z poziomu Pulse
- [ ] Filtrowanie zdarzeń we wszystkich panelach
- [ ] Persystencja sesji w SQLite
- [ ] Eksport snapshotu do Markdown / JSON
- [ ] Tryb zen (`⌘.`)
- [ ] Pasek boczny dla wielu agentów
- [ ] Obsługa transportu HTTP ACP (monitorowanie zdalnych agentów)

Pełny plan i dyskusje znajdziesz w [Issues](https://github.com/vpui/pulse/issues).

---

## Współtworzenie

Pulse jest otwarty na wkład. Jeśli chcesz pomóc:

1. Zrób fork repozytorium.
2. Utwórz branch (`git checkout -b feature/nazwa`).
3. Zatwierdź zmiany (`git commit -m 'Dodaj feature'`).
4. Wypchnij branch (`git push origin feature/nazwa`).
5. Otwórz Pull Request.

Przed rozpoczęciem przeczytaj [CONTRIBUTING.md](CONTRIBUTING.md) i [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md). Jeśli nie wiesz, od czego zacząć, zajrzyj do issues oznaczonych etykietą [`good first issue`](https://github.com/vpui/pulse/labels/good%20first%20issue).

### Zgłaszanie błędów

Skorzystaj z [szablonu issue](https://github.com/vpui/pulse/issues/new/choose). Dołącz wersję Pulse, system operacyjny oraz logi z `~/.local/share/pulse/logs/`.

---

## Licencja

Projekt jest udostępniany na licencji MIT. Szczegóły znajdziesz w pliku [LICENSE](LICENSE).

---

<div align="center">

**[⬆ Wróć na górę](#pulse)**

Pulse, bo kontrola nad agentami nie musi krzyczeć.

</div>
