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
