```markdown
# Pulse

> Spokojny kokpit dla Twoich agentów AI.

[![License: MIT](https://img.shields.io/badge/License-MIT-5B8DEF?style=flat-square)](LICENSE)
[![Tauri](https://img.shields.io/badge/Tauri-2-24C8DB?style=flat-square&logo=tauri&logoColor=white)](https://tauri.app)
[![Svelte](https://img.shields.io/badge/Svelte-5-FF3E00?style=flat-square&logo=svelte&logoColor=white)](https://svelte.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-000000?style=flat-square&logo=rust&logoColor=white)](https://www.rust-lang.org)
[![ACP](https://img.shields.io/badge/ACP-compatible-3DD68C?style=flat-square)](https://agentclientprotocol.com)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-6E6E76?style=flat-square)]()

**Pulse** to lekka aplikacja desktopowa do monitorowania i kontrolowania lokalnie uruchomionych agentów AI — Claude Code, Gemini CLI, Codex, OpenCode i innych. Wszyscy agenci w jednym miejscu, bez otwierania ciężkiego IDE.

Zbudowana w Rust (Tauri 2). Działa na Linuksie, macOS i Windows. Na Linuksie wygląda jak dopracowana natywna aplikacja z ekosystemu macOS.

---

## Spis treści

- [Dlaczego Pulse](#dlaczego-pulse)
- [Funkcje](#funkcje)
- [Architektura](#architektura)
- [Szybki start](#szybki-start)
- [Konfiguracja](#konfiguracja)
- [Design](#design)
- [Stack technologiczny](#stack-technologiczny)
- [Roadmap](#roadmap)
- [Współtworzenie](#współtworzenie)
- [Licencja](#licencja)

---

## Dlaczego Pulse

W 2026 roku programiści rutynowo korzystają z wielu agentów kodujących jednocześnie: jeden robi code review, drugi generuje testy, trzeci prowadzi research. Edytory (Zed, VS Code) mają wbudowane panele agentów, ale są one ściśle związane z IDE i przeciążone funkcjami, których nie potrzebujesz, gdy chcesz tylko rzucić okiem, co robią Twoi agenci.

Pulse adresuje tę niszę: **chcę widzieć i kontrolować moich agentów, ale nie chcę do tego otwierać całego edytora.**

| Problem | Rozwiązanie w Pulse |
|---|---|
| Kilka terminali obok siebie | Jeden ekran ze wszystkimi agentami |
| Ciężkie panele w IDE | Lekka aplikacja desktopowa (~10 MB) |
| Niestabilne integracje pod dostawców | Jeden standard: Agent Client Protocol |
| Krzykliwe TUI z neonami | Spokojny, minimalistyczny interfejs |
| Brak historii po restarcie | Persystencja sesji w SQLite |

---

## Funkcje

| Funkcja | Opis | Status |
|---|---|---|
| Lista agentów | Wszyscy agenci w jednym miejscu, kropka statusu: zielona / szara / czerwona | ✅ |
| Widok szczegółów | Klik w panel rozwija strumień zdarzeń w miejscu | ✅ |
| Anulowanie sesji | Jeden skrót, gdy agent zabrnie w ślepy zaułek | ✅ |
| Zatwierdzanie uprawnień | Agent chce wykonać polecenie shellowe? Zatwierdź bez wychodzenia z Pulse | 🚧 |
| Filtrowanie zdarzeń | Tylko błędy? Tylko wywołania narzędzi? We wszystkich panelach naraz | 🚧 |
| Persystencja sesji | SQLite zapisuje historię, po restarcie widzisz stan przed zamknięciem | 🚧 |
| Eksport snapshotu | Stan wszystkich agentów do Markdown lub JSON | 📋 |
| Tryb zen | `⌘.` ukrywa wszystko poza listą agentów i kropkami statusu | 📋 |

Legenda: ✅ gotowe · 🚧 w toku · 📋 zaplanowane

---

## Architektura

Pulse nie jest zbiorem hacków pod konkretnych dostawców. Rozmawia z agentami przez **[Agent Client Protocol (ACP)](https://agentclientprotocol.com)** — otwarty standard (JSON-RPC 2.0 po stdio) stworzony przez Zed, który pozwala dowolnemu klientowi rozmawiać z dowolnym agentem w ustandaryzowany sposób.

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

Kluczowa decyzja: cała logika (ACP, zarządzanie procesami, SQLite) żyje w Ruście. Frontend tylko renderuje. Dzięki temu aplikacja pozostaje responsywna nawet przy dziesięciu równoległych agentach.

---

## Szybki start

### Wymagania

| Narzędzie | Wersja | Link |
|---|---|---|
| Rust | stable | [rust-lang.org/tools/install](https://www.rust-lang.org/tools/install) |
| Node.js | LTS | [nodejs.org](https://nodejs.org) |
| Tauri CLI | 2.x | [tauri.app/start](https://tauri.app/start/) |

Na Linuksie wymagane są zależności systemowe Tauri — pełna lista w [oficjalnej dokumentacji](https://tauri.app/start/prerequisites/).

### Instalacja

```bash
git clone https://github.com/twoj-user/pulse.git
cd pulse
npm install
npm run tauri dev
```

### Budowanie wersji produkcyjnej

```bash
npm run tauri build
```

Gotowe binarki znajdziesz w `src-tauri/target/release/bundle/`.

---

## Konfiguracja

Pulse czyta konfigurację z `~/.config/pulse/agents.toml`. Każdy agent musi wspierać ACP — lista kompatybilnych agentów rośnie, sprawdź [ACP Registry](https://agentclientprotocol.com/get-started/registry).

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
| `Enter` | Rozwiń / zwiń szczegóły agenta |
| `c` | Anuluj sesję aktywnego agenta |
| `a` | Zatwierdź żądanie uprawnień |
| `f` | Otwórz filtr zdarzeń |
| `⌘.` | Tryb zen |
| `⌘,` | Ustawienia |
| `q` | Wyjście |

---

## Design

Pulse wygląda tak, jak powinien wyglądać spokojny kokpit.

### Paleta

| Rola | Kolor | HEX |
|---|---|---|
| Tło aplikacji | Prawie czarny | `#0A0A0B` |
| Tło panelu | Ciemny grafit | `#121214` |
| Obramowanie | Ledwo widoczne | `#1F1F23` |
| Tekst główny | Jasny szary | `#E8E8EA` |
| Tekst wyciszony | Średni szary | `#6E6E76` |
| Akcent | Stonowany błękit | `#5B8DEF` |
| Status: pracuje | Zielony | `#3DD68C` |
| Status: błąd | Czerwony | `#E5484D` |

### Zasady, których nie łamiemy

- Jedna grubość obramowania (1px). Spójne zaokrąglenia (10px panele, 6px przyciski).
- Dużo powietrza. Minimum 16px między sekcjami, 24px padding w panelach.
- Animacje: 180–220ms, jedna krzywa `cubic-bezier(0.32, 0.72, 0, 1)` — zapożyczona z macOS.
- Typografia zamiast ikon. Ikony tylko tam, gdzie naprawdę trzeba (Lucide, stroke 1.5px).
- Zero gradientów. Zero glow. Zero neonów.

---

## Stack technologiczny

| Warstwa | Wybór | Uzasadnienie |
|---|---|---|
| Shell aplikacji | [Tauri 2](https://tauri.app) | ~10 MB zamiast ~150 MB Electrona, start < 300ms |
| Backend | Rust (tokio, `serde_json`) | Wydajność, bezpieczeństwo, zarządzanie procesami |
| Klient ACP | Własna implementacja JSON-RPC 2.0 | Pełna kontrola, minimalne zależności |
| Frontend | [Svelte 5](https://svelte.dev) + Tailwind | Najmniejszy bundle, płynne animacje |
| Typografia | Inter + JetBrains Mono | Czytelność UI + czytelność kodu |
| Persystencja | SQLite (`rusqlite`) | Lokalnie, bez zależności od chmury |
| Ikony | [Lucide](https://lucide.dev) | Cienki stroke, spójny zestaw |

---

## Roadmap

- [x] Klient ACP z presetami (Claude, Gemini, Codex)
- [x] Lista agentów + widok szczegółów
- [x] Anulowanie sesji
- [ ] Zatwierdzanie uprawnień z poziomu Pulse
- [ ] Filtrowanie zdarzeń we wszystkich panelach
- [ ] Persystencja sesji w SQLite
- [ ] Eksport snapshotu do Markdown / JSON
- [ ] Tryb zen (`⌘.`)
- [ ] Pasek boczny dla wielu agentów
- [ ] Obsługa transportu HTTP ACP (monitorowanie zdalnych agentów)

Pełny plan i dyskusja w [Issues](https://github.com/twoj-user/pulse/issues).

---

## Współtworzenie

Pulse jest otwarty na wkład. Jeśli chcesz pomóc:

1. Zrób fork repozytorium
2. Stwórz branch (`git checkout -b feature/nazwa`)
3. Commituj zmiany (`git commit -m 'Dodaj feature'`)
4. Push (`git push origin feature/nazwa`)
5. Otwórz Pull Request

Zanim zaczniesz, przeczytaj [CONTRIBUTING.md](CONTRIBUTING.md) i [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md). Jeśli nie jesteś pewien, od czego zacząć — zajrzyj do issues oznaczonych [`good first issue`](https://github.com/twoj-user/pulse/labels/good%20first%20issue).

### Zgłaszanie błędów

Użyj [szablonu issue](https://github.com/twoj-user/pulse/issues/new/choose). Dołącz wersję Pulse, system operacyjny i logi z `~/.local/share/pulse/logs/`.

---

## Licencja

Dystrybuowane na licencji MIT. Szczegóły w pliku [LICENSE](LICENSE).

---

<div align="center">

**[⬆ Wróć na górę](#pulse)**

Pulse — bo kontrola nad agentami nie musi krzyczeć.

</div>
```
