# Changelog — Goedemorgen App

All notable changes to this project will be documented in this file.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.3.2] — 2026-05-18

### Changed
- **Gist-credentials opgeslagen in localStorage** — token en Gist ID worden bij de eerste
  keer laden via `?token=` en `?gist=` direct opgeslagen in `gm_gist_token` en `gm_gist_id`.
  Bij volgende laadmomenten (refresh, homescreen-shortcut) worden de waarden uit localStorage
  gelezen zodat de volledige URL niet meer vereist is.
- **Prioriteitsvolgorde credentials** — URL-parameters hebben altijd voorrang op localStorage;
  een nieuwe URL overschrijft bestaande opgeslagen waarden.

### Added
- **Gist ID-weergave in instellingen** — toont het actieve Gist ID gemaskeerd
  (`••••••<laatste 6 tekens>`) wanneer sync actief is.
- **"Ontkoppelen" knop** — verwijdert `gm_gist_token` en `gm_gist_id` uit localStorage
  en herlaadt de pagina; zichtbaar alleen wanneer sync actief is.
- `K.gist` (`gm_gist_token`) en `K.gistId` (`gm_gist_id`) teruggezet in het `K`-object
  nu deze sleutels weer in localStorage worden opgeslagen.
- `disconnectGist()` toegevoegd als functie voor de ontkoppelactie.

---

## [1.3.1] — 2026-05-18

### Fixed
- **Agenda niet beschikbaar na Gist-sync** — `renderSettingsCals` crashte wanneer
  `gm_gcal_cals` als `"null"` in localStorage stond (gezet door Gist-sync). `ls.get()`
  retourneert in dat geval `null` in plaats van de fallback, waarna `null.includes()`
  een TypeError gooide. De catch-block ving dit op en overschreef de agenda-content
  met "Agenda niet beschikbaar" — ook als er wél afspraken waren. Opgelost met een
  expliciete `|| _allCalIds` fallback.
- **Verkeerd toggle-gedrag bij onaangeroerde agendakeuze** — `toggleCal` startte
  altijd met een lege array als `K.cals` `null` was. Omdat alle agenda's dan visueel
  aangevinkt leken maar de interne array leeg was, voegde de eerste klik een agenda toe
  i.p.v. te verwijderen; de tweede klik maakte de array leeg (`[]`). Resultaat: na twee
  kliks werden er geen agenda-IDs meer meegestuurd naar de API. Opgelost door te starten
  vanuit `_allCalIds` (module-scope cache) als `K.cals` leeg of null is.
- **Lege agenda-selectie toonde geen afspraken** — `selIds = []` resulteerde in nul
  API-calls, waardoor "Geen afspraken vandaag" verscheen ook al waren er afspraken.
  Opgelost door een lege array gelijk te behandelen aan `null` (gebruik alle agenda's).

### Changed
- `_allCalIds` toegevoegd als module-scope variabele; wordt gevuld door
  `renderSettingsCals` zodat `toggleCal` altijd de volledige lijst kent.

---

## [1.3] — 2026-05-18

### Added
- **GitHub Gist sync** — instellingen worden opgeslagen in en geladen vanuit een GitHub Gist,
  waardoor synchronisatie tussen apparaten (laptop, iPhone, iPad) mogelijk is zonder backend.
- **Auto-save via debounce** — elke schrijfactie naar een `SYNC_KEYS`-sleutel triggert automatisch
  een `saveToGist()` na 2 seconden stilte, zodat handmatig opslaan niet nodig is.
- **Auto-load bij opstarten** — als URL-parameters aanwezig zijn, worden instellingen stil geladen
  vanuit de Gist vóór het dashboard wordt opgebouwd. Maximale wachttijd: 3 seconden
  (`Promise.race` met timeout) om het dashboard nooit te blokkeren.
- **URL-parameter authenticatie** (`?token=` + `?gist=`) — token en Gist ID worden uitsluitend
  uit de URL gelezen en nooit opgeslagen in `localStorage` of elders.
- **`GIST_TOKEN` / `GIST_ID` constanten** — worden eenmalig bepaald via `URLSearchParams`
  bij het laden van de pagina; in-memory only.
- **`SYNC_KEYS` Set** — definieert welke `localStorage`-sleutels een Gist-sync triggeren.
  OAuth-tokens en API-credentials zijn expliciet uitgesloten.
- **`skipGistSync` vlag** — voorkomt een recursieve sync-loop wanneer `applySettings()`
  meerdere `ls.set()`-aanroepen doet.
- **`applySettings(data)`** — centrale functie voor het toepassen van instellingen,
  hergebruikt door zowel import als Gist-load.
- **`buildSettingsPayload()`** — gedeelde functie voor export en Gist-save; bevat geen tokens.
- **`debouncedGistSync()`** — 2-seconden debounce wrapper rondom `saveToGist(true)`.
- **Gist-statusindicator in instellingen** — toont groen of oranje melding afhankelijk van
  of `?token=` en `?gist=` aanwezig zijn in de URL.
- **Nederlandse foutmeldingen** bij Gist API-fouten: ongeldige token (401), Gist niet gevonden
  (404), bestand ontbreekt, netwerk onbereikbaar.

### Changed
- `ls.set()` uitgebreid: triggert `debouncedGistSync()` als de sleutel in `SYNC_KEYS` staat.
- `exportSettings()` hergebruikt nu `buildSettingsPayload()` in plaats van inline constructie.
- `importSettings()` delegeert nu naar `applySettings()` in plaats van 9 losse `if`-blokken.
- Gist-sectie in het instellingenscherm toont alleen nog de sync-knoppen en statusregel;
  alle invoervelden zijn verwijderd.

### Removed
- `K.gist` en `K.gistId` uit het `K`-object — worden niet langer in `localStorage` opgeslagen.
- `saveGistCredentials()` — functie volledig verwijderd.
- Token- en Gist ID-invoervelden (`<input>`) uit het instellingenscherm.
- `openSettings()`-regels die token/ID-velden populeerden vanuit `localStorage`.

---

## [1.2] — 2026-05-16

### Added
- **Exporteer instellingen** — alle gebruikersinstellingen kunnen als `.json`-bestand worden
  gedownload via een knop in het instellingenscherm.
- **Importeer instellingen** — een eerder geëxporteerd `.json`-bestand kan worden ingeladen;
  het instellingenscherm wordt direct bijgewerkt na import.
- Versieveld `_version: '1.2'` en tijdstempel `_exported` in het export-payload.

### Changed
- Instellingenscherm uitgebreid met sectie "Export / Import".

---

## [1.1] — 2026-05-15

### Added
- **Tijdgebaseerde routerichting** — van 04:00 tot 11:59 wordt de route van thuis naar werk
  berekend; van 12:00 tot 03:59 de terugroute van werk naar huis.
- **Kindroutes (generiek)** — ouders kunnen een of meerdere kinderen toevoegen met naam,
  ophaaladres, brengadres, en dag/tijdconfiguratie. Bijbehorende verkeersinformatie
  verschijnt automatisch in het dashboard op de ingestelde dagen en tijden.
- **Weerlocatie zichtbaar** — de naam van de weerlocatie (stad, land) wordt getoond in
  de weersectie zodat duidelijk is voor welke locatie het weer wordt opgehaald.
- **Bevestigingsfeedback bij opslaan** — knoppen tonen een korte groene bevestiging
  ("Opgeslagen ✓") na het opslaan van instellingen via `showSaved()`.
- **Handmatige weerlocatie** — naast automatische geolocatie kan een vaste locatie
  (coördinaten of plaatsnaam) worden ingevoerd in de instellingen.
- **Privacymodus adressen** — thuisadres en werkadres worden niet langer hardcoded in
  de broncode; ze worden ingevoerd via de instellingen en opgeslagen in `localStorage`.
- Nieuwsproxy gemigreerd naar **corsproxy.io + native `DOMParser`** voor XML-parsing,
  met `allorigins.win` als fallback.

### Fixed
- **TomTom reistijd 159 minuten** — geocodeerverzoeken misten de parameter `countrySet=NL`,
  waardoor een verkeerd adres in het buitenland werd opgepakt. Opgelost door
  `&countrySet=NL` toe te voegen aan alle TomTom Search API-aanroepen.
- **Nieuws niet beschikbaar** — rss2json.com vereist inmiddels een API-sleutel voor het
  `count`-parameter; volledig vervangen door eigen XML-parsing pipeline.
- **Weerlocatienaam leeg bij laden** — het weerkaart-element miste `id="weather-card"`,
  waardoor de locatienaam na een Nominatim-lookup nergens kon worden weggeschreven.

---

## [1.0] — 2026-05-14

### Added
- Initiële release als single-file dashboard (`goedemorgen.html`).
- **Live klok en datum** in het Nederlands (`NL_DAYS`, `NL_MONTHS`).
- **Weer** via Open-Meteo API (geen sleutel vereist); toont temperatuur, gevoelstemperatuur,
  neerslag en windsnelheid. Locatie via browser-geolocatie of Nominatim reverse geocoding.
- **Verkeersinformatie** via TomTom Routing API; toont reistijd en vertraging per route.
- **Google Agenda** via OAuth 2.0 implicit flow; toont de agenda-afspraken van de dag.
- **Nieuws** via RSS-feeds (algemeen nieuws, technologie/AI) met tijdstempel.
- **Instellingenscherm** (⚙️) voor thuisadres, werkadres, TomTom API-sleutel en
  Google OAuth Client ID.
- Dark theme met CSS custom properties (`--bg: #0f1923`, `--accent: #f59e0b`).
- Mobile-first layout geoptimaliseerd voor iPhone Safari met `safe-area-inset` padding.
- `localStorage` als persistentielaag via `ls.get()` / `ls.set()` helper met
  JSON-serialisatie en stille foutafhandeling.
