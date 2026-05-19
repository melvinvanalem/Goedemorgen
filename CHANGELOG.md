# Changelog — Goedemorgen App

All notable changes to this project will be documented in this file.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## Release note voor GitHub (1.6.5)

---

## [1.6.5] — 2026-05-19

### Added
- **Refresh-knop in de header** — een `↻`-knop links naast de instellingen
  vernieuwt alle dashboard-secties (weer, reistijd, agenda, nieuws) parallel.
  Pull-to-refresh werkt niet betrouwbaar in iOS standalone webapps; deze knop
  vervangt die handeling. De knop spint tijdens het laden (`@keyframes spin`)
  en is gedisabled zolang de refresh loopt, met een minimum van 600ms zichtbare
  feedback voor snelle responses.
- **`refreshAll()`** — herlaadt `loadWeather()`, `loadTraffic()`, `loadCalendar()`
  en `loadNews()` via `Promise.allSettled` zodat één traag onderdeel de rest
  niet blokkeert.
- **`.header-actions`** flex-wrapper rondom de twee knoppen voor consistente
  uitlijning op smalle schermen.

### Changed
- **TomTom routing-calls gehalveerd** — `calcRoute()` deed eerst twee aparte
  routing-requests (één met `traffic=true`, één met `traffic=false`) om de
  vertraging te berekenen. Nu gebruiken we de parameter
  `computeTravelTimeFor=all` waardoor de respons in één request zowel
  `travelTimeInSeconds` (live verkeer) als `noTrafficTravelTimeInSeconds`
  (vrije doorgang) bevat. Combineert met de geocode-cache uit 1.6.4: per route
  nog maar één TomTom-call in plaats van vier — dat is een **~75% reductie**
  van het API-verkeer en lost de resterende "soms niet beschikbaar"-meldingen op.
- **`fetchJsonWithRetry()`** uitgebreid van één retry (400ms) naar twee retries
  met exponentiële backoff (400ms → 1200ms). Een tijdelijke 429 of 503 wordt nu
  netjes opgevangen in plaats van direct als "Niet beschikbaar" getoond.
- **`calcRoute()`** valideert nu expliciet de aanwezigheid van `routes[0].summary`
  en gooit een duidelijke error bij ontbrekende data, in plaats van te crashen
  op `undefined.travelTimeInSeconds`.

### Fixed
- **Verkeer-badge toonde bijna altijd "Vlot rijden"** — de oude `traffic=true` /
  `traffic=false` aanpak gaf vaak twee bijna identieke reistijden terug, waardoor
  het verschil (`diff`) tegen nul aanlag en de drempels voor "Druk op de weg"
  (>20%) en "Veel drukte" (>40%) nooit gehaald werden. Door over te stappen op
  `computeTravelTimeFor=all` levert TomTom nu de échte vrije-doorgang baseline
  (`noTrafficTravelTimeInSeconds`) — vertraging wordt zichtbaar en de badges
  kloppen weer.

### Notes
- **iOS pull-to-refresh in standalone modus** is op iOS 17+ enigszins
  beschikbaar maar gedraagt zich grillig — vooral als de eerste pixel van
  de pagina niet zichtbaar is. De expliciete refresh-knop is daarom de
  betrouwbare route.

  ---

## [1.6.4] — 2026-05-19

### Fixed
- **Route-titel klopte niet bij terugrit** — na 12:00 toont de header "🏠 Naar huis"
  en worden `from`/`to` correct omgedraaid, maar de rij-titel bleef "Naar werk
  (Houten)" met daaronder "Vertrek: Pakketboot 57, Houten". Inhoudelijk klopte het
  niet: je vertrekt vanuit werk, niet ernaartoe. Nu wordt elke route-naam die met
  "Naar " begint dynamisch omgezet naar "Vanuit " (regex `^Naar\s+` →
  `Vanuit `). Routes zonder dat prefix krijgen een `↩ ` voorzetsel.
- **Wisselende "Niet beschikbaar" per refresh** — TomTom geocode- en routing-calls
  gingen elke refresh opnieuw parallel naar de API. Op een gratis TomTom-account
  veroorzaakt dat sporadische rate-limit afwijzingen (429), waardoor telkens een
  ándere route faalde. Twee maatregelen:
  - **Geocode-cache** — lat/lon per adres wordt nu een week gecached in
    `gm_geocode_cache` (localStorage, niet in `SYNC_KEYS`). Dezelfde routes
    worden bij elke refresh dus zonder geocode-calls berekend.
  - **`fetchJsonWithRetry()`** — één retry met 400ms backoff op 429 en 5xx
    responses voor de routing-calls.

### Added
- **Bestemming zichtbaar in elke route-rij** — onder de route-naam staat nu
  `<vertrekadres> → <bestemmingsadres>` in plaats van alleen "Vertrek: …".
  Geldt voor alle routes inclusief kind-routes (wegbrengen én ophalen).
- **Route-rij is klikbaar → opent Google Maps app met route geladen** — elke
  rij is nu een `<a href="comgooglemaps://?saddr=…&daddr=…&directionsmode=driving">`
  die direct de Google Maps-app opent met vertrekadres en bestemming
  ingevuld en rij-modus geselecteerd. Consistent met de bestaande
  "Open Google Maps →" knop. Werkt ook op rijen die als "Niet beschikbaar"
  worden getoond — dan kun je alsnog handmatig de route in Maps openen.
- **`mapsRouteUrl(from, to)`** — helper die het comgooglemaps:// URL bouwt.
- **`escapeHtml()`** — kleine helper zodat adressen veilig in zowel attribute
  als tekstpositie worden gerenderd; voorkomt rendering-glitches bij rare tekens
  (`&`, `<`, `"`) in adresnamen.

### Changed
- **`.route-row` CSS** uitgebreid met `color: inherit`, `text-decoration: none`
  en een lichte tap-feedback (`a.route-row:active`) zodat het element zowel als
  `<div>` (skeleton) als `<a>` (echte rij) goed rendert.
- **`.route-sub`** krijgt `line-height: 1.35` en `word-break: break-word` voor
  betere weergave van twee adressen op smalle schermen.
- **`.route-left` / `.route-right`** met `flex: 1 / flex-shrink: 0` zodat de
  reistijd-badge altijd rechts past, ook bij lange adressen.

---

## [1.6.3] — 2026-05-19

### Fixed
- **"Agenda niet beschikbaar" in iOS standalone webapp** — the OAuth token expires
  hourly, but `silentTokenRefresh()` is unreliable in iOS PWA mode because iOS opens
  `accounts.google.com` in Safari instead of the standalone webview. The webapp held
  a stale token whose expiry timestamp was still in the future, so `validToken()`
  returned it, the Google API rejected it, and the generic catch branch hid the
  problem behind a dead "Agenda niet beschikbaar" message — without offering any
  way to recover.
- **Per-calendar event fetches did not check HTTP status** — only the initial
  `calendarList` request threw on `!r.ok`. A 401 on individual calendar requests
  passed through silently as an empty result, masking auth failures.
- **Cancelled or malformed events crashed the render** — Google Calendar API can
  return items with `status: "cancelled"` and no `start`/`end` fields. One such
  event made `new Date(ev.start.dateTime || ev.start.date)` throw a TypeError,
  which propagated to the outer catch and triggered the dead fallback message.

### Changed
- **OAuth token & expiry now sync via Gist** — `gm_gcal_token` and `gm_gcal_exp`
  added to `SYNC_KEYS`. A token refresh in Safari (where the OAuth redirect flow
  works) now propagates within ~2 seconds to the iPhone webapp on the home screen.
  Eenmalig koppelen blijft genoeg; daarna geen handmatig werk meer.
- **`buildSettingsPayload()` & `applySettings()`** uitgebreid met `gcalToken` en
  `gcalExp` velden. Bestaande export-bestanden zonder die velden blijven werken
  (de `if (data.x !== undefined)`-guards laten ontbrekende velden gewoon staan).
- **`authedFetch()` helper geïntroduceerd in `loadCalendar()`** — elke 401/403
  van Google gooit nu `'auth'`, ongeacht welke API-call faalt. Een totale rejection
  op alle per-agenda fetches wordt óók als auth-fout behandeld.
- **Catch-fallback toont een actieknop** — "🔄 Opnieuw verbinden" in plaats van
  de doodlopende tekst "Agenda niet beschikbaar". Bij een 'auth'-fout wordt nu
  ook `tokenExp` op 0 gezet en `gm_silent_failed` uit `sessionStorage` verwijderd
  zodat een volgende stille vernieuwingspoging weer mogelijk is.
- **Cancelled-event filter** — `ev.status !== 'cancelled' && ev.start && ev.end`
  toegevoegd in zowel `loadCalendar()` als `loadTomorrowAgenda()`.

### Security notes
- De OAuth-token (read-only Calendar scope, 1 uur geldig) staat nu in de privé
  Gist naast adressen, routes en agenda-IDs. Het dreigingsbeeld is praktisch
  ongewijzigd: wie toegang krijgt tot de GitHub Personal Access Token heeft
  sowieso al toegang tot alle synchroniseerde instellingen. De token verloopt
  bovendien automatisch binnen een uur.

### Notes
- **Eenmalig herkoppelen vereist na update**: open de app in Safari (niet vanaf
  beginscherm), ⚙️ → Google Agenda → Ontkoppelen → Koppel opnieuw. Daarna werkt
  zowel Safari als de homescreen-webapp automatisch.

  ---

---

## [1.6.2] — 2026-05-19

### Fixed
- **NOS button did not open** — the "meer nieuws →" link used the URL scheme
  `nosapp://`, but the NOS app does not register that scheme. Safari showed
  "Cannot open page because the address is invalid". Replaced with `https://nos.nl/`;
  iOS opens the NOS app via Universal Links if installed, and falls back to the
  website in Safari otherwise.
- **Outlook button opened the inbox instead of the calendar** — the "Open Outlook
  agenda →" link used `ms-outlook://` without a path, so Outlook defaulted to the
  mail tab. Replaced with `ms-outlook://events` so the app opens directly in the
  calendar tab.

### Notes
- Both changes are single-line edits in `goedemorgen.html`; no changes to APIs,
  storage, or the OAuth flow. No re-authentication required.

---

## [1.6.1] — 2026-05-18

### Changed
- **External links now use iOS URL schemes** — all app links have been updated from
  `https://` URLs to native URL schemes so iOS opens the installed app directly instead
  of Safari. No re-login required.

| Link | Before | After |
|---|---|---|
| Open Weeronline → | `https://www.weeronline.nl` | `weeronline://` |
| Open Google Maps → | `https://maps.google.com` | `comgooglemaps://` |
| Open Outlook agenda → | `https://outlook.office.com/calendar/view/day` | `ms-outlook://` |
| meer nieuws → | `https://nos.nl` | `nosapp://` |

- **`target="_blank"` removed** from app-scheme links — URL scheme links navigate
  directly to the native app and do not require a new browser tab.

### Notes
- **meer tech →** (`https://tweakers.net`) and **meer ai →**
  (`https://the-decoder.com`) remain as `https://` links — these are web-only
  publications with no native iOS app.

  ---

## [1.6] — 2026-05-18

### Added
- **"Open Outlook agenda" knop** — subtiele knop onderaan de agenda-kaart, onder het
  vandaag- en morgenblok. Opent `outlook.office.com/calendar/view/day` direct in de
  dagweergave van vandaag in een nieuw tabblad.
- **Directe toegang tot werkagenda** — lost het ontbreken van Outlook-synchronisatie
  praktisch op; oude en terugkerende afspraken zijn direct zichtbaar via één tik.
- **"Meer nieuws" knop** — onderaan de Nieuws-sectie; opent `https://nos.nl` in een
  nieuw tabblad.
- **"Meer tech" knop** — onderaan de Tech-sectie; opent Tweakers in een nieuw tabblad.
- **"Meer AI" knop** — onderaan de AI-sectie; opent The Decoder in een nieuw tabblad.
- **Google Maps link in reistijd-kaart** — tikt op de kaart of knop opent Google Maps
  met de betreffende route direct ingeladen.
- **Weeronline link in weer-kaart** — subtiele link onderaan de weer-kaart opent
  `https://weeronline.nl` voor een uitgebreid weersoverzicht.

### Changed
- Agenda-kaart uitgebreid met een vaste voettekst met de Outlook-knop.
- Nieuws-, tech- en AI-secties uitgebreid met een voettekst met doorklinklink.
- Reistijd- en weerkaart uitgebreid met externe links.


---

## [1.5] — 2026-05-18

### Added

#### Hourly weather strip
- **Horizontal weather strip** — displays hourly forecast from the current hour until
  midnight, up to 12 hours ahead. Horizontally scrollable with one finger on iPhone.
- **Current hour highlighted** — subtle amber background (`rgba(245,158,11,0.12)`)
  on the current time slot for instant orientation.
- **Conditional precipitation row** — the rain row is only rendered when at least one
  hour in the strip has > 0mm precipitation. Hidden entirely on dry days.
- **No new API key required** — extends the existing Open-Meteo request with
  `hourly=temperature_2m,precipitation,weathercode`, `timezone=auto` and
  `forecast_days=2`. Reuses the existing `wmo()` function for weather icons.

#### Tomorrow's agenda
- **Tomorrow block in the calendar card** — shows tomorrow's events below a dashed
  divider, visible only when relevant: after 17:00, when all of today's events have
  ended, or when there are no events today.
- **Collapsed (default)** — compact single line `morgen › 09:00 Title  13:00 Title`
  showing up to 3 events. When more than 3: `en X meer…` appended.
- **Expands on tap** — full event list for tomorrow, all-day events first, then
  sorted by start time. Same styling as today's events at 75% opacity.
  Tap again to collapse.
- **No events tomorrow** — block and divider are hidden entirely.
- **Separate Google Calendar fetch** for tomorrow's date range (00:00–23:59),
  using the same selected calendars and authorization as today's fetch.

### Changed
- Open-Meteo API request extended with `hourly` fields, `timezone=auto` and
  `forecast_days=2` (was `forecast_days=1`).

  ---

## [1.4.1] — 2026-05-18

### Fixed
- **App blijft laden na Google Agenda koppelen** — `GIST_TOKEN` en `GIST_ID` werden
  gedeclareerd vóór de `ls` helper (regel 823 vs. regel 841). JavaScript's `const` heeft
  een temporal dead zone: aanroepen vóór de declaratieregel gooien een `ReferenceError`
  die het volledige script afbreekt. Gevolg: `catchToken()` en `init()` werden nooit
  uitgevoerd, waardoor de laadskeletons oneindig bleven staan.
- **Waarom het alleen na OAuth-redirect crashte** — bij een URL met `?token=` is
  `_urlToken` truthy; JavaScript evalueert `_urlToken || ls.get(...)` dan via
  short-circuit en roept `ls.get()` nooit aan. Zonder query-parameter (zoals bij
  terugkeer van de OAuth-redirect) werd `ls.get()` wél geëvalueerd → crash.

### Changed
- Gist-credential initialisatie (`_urlParams`, `GIST_TOKEN`, `GIST_ID`) verplaatst
  naar ná de `ls`-declaratie zodat `ls.get()` veilig aangeroepen kan worden.

  ---

## [1.4] — 2026-05-18

### Added
- **Stille OAuth-tokenvernieuwing** — wanneer het Google-token verlopen is, wordt automatisch
  een stille redirect uitgevoerd naar Google met `prompt=none`. Als de gebruiker al ingelogd
  is bij Google (standaard op een persoonlijk apparaat), keert de app direct terug met een
  vers token zonder enige zichtbare interactie.
- **`silentTokenRefresh()`** — voert de `prompt=none`-redirect uit via `location.replace()`
  zodat de redirect niet in de browsergeschiedenis terechtkomt.
- **Loop-beveiliging via `sessionStorage`** — twee vlaggen voorkomen oneindige redirect-loops:
  - `gm_silent_pending` — gezet vóór de redirect, verwijderd bij terugkeer van Google
  - `gm_silent_failed` — gezet als Google een fout teruggeeft (bijv. gebruiker niet
    ingelogd); voorkomt herhaalde pogingen binnen dezelfde sessie. Wordt automatisch
    gewist bij de volgende app-start (sessionStorage leeft per sessie).

### Changed
- **`catchToken()` uitgebreid** — verwerkt nu ook de `error`-parameter in de URL-hash
  die Google teruggeeft bij een mislukte `prompt=none`-aanvraag. Eerder werd alleen
  `access_token` afgehandeld.
- **`init()` uitgebreid** — controleert na de Gist-sync of het token verlopen is.
  Zo ja, en als aan de voorwaarden is voldaan, wordt `silentTokenRefresh()` aangeroepen
  en stopt `init()` (de redirect neemt het over).

### Behavior
- **Eerste keer op een nieuw apparaat of homescreen-context** — stille vernieuwing wordt
  geprobeerd; Google heeft daar nog geen sessie, stuurt een fout terug, de app toont de
  "Verbind Google Agenda"-knop. Eenmalig koppelen volstaat.
- **Daarna** — bij elke volgende start vernieuwt de app het token automatisch en onzichtbaar
  zolang de gebruiker ingelogd is bij Google in dat browser-context (doorgaans maanden).
- **Geen wijzigingen nodig in Google Cloud Console** — werkt met het bestaande OAuth
  client ID en redirect URI.

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
