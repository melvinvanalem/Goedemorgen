# Goedemorgen App

Persoonlijk ochtend-dashboard voor dagelijks gebruik. Gebouwd als single HTML-file, gehost op GitHub Pages.

**Live:** [melvinvanalem.github.io/Goedemorgen/goedemorgen.html](https://melvinvanalem.github.io/Goedemorgen/goedemorgen.html)

---

## Wat het doet

- Live klok en datum in het Nederlands
- Weer vandaag + morgen op basis van huidige locatie
- Reistijd en verkeersverwachting voor vaste routes (Utrecht → Houten, Utrecht → Capelle)
- Google Agenda afspraken van de dag
- Nieuws: NOS algemeen, Tweakers tech, The Decoder AI
- Op maandag, zaterdag en zondag geen reistijd — alleen een melding

---

## Hoe te openen

De app werkt via een URL met token en Gist ID als parameters:

```
https://melvinvanalem.github.io/Goedemorgen/goedemorgen.html?token=JOUWTOKEN&gist=JOUWGISTID
```

Deze URL is opgeslagen als bladwijzer op het beginscherm van mijn iPhone.

---

## Benodigde accounts en sleutels

| Wat | Waar aan te maken | Waarvoor |
|-----|------------------|----------|
| Google OAuth Client ID | console.cloud.google.com | Google Agenda koppeling |
| TomTom API key | developer.tomtom.com | Reistijd en verkeer |
| GitHub Personal Access Token | github.com → Settings → Developer settings | Gist sync |
| GitHub Gist ID | gist.github.com | Opslag instellingen |

---

## Instellingen synchronisatie

Instellingen worden opgeslagen in een privé GitHub Gist. Token en Gist ID staan alleen in de URL — nooit in de code of localStorage.

Bij nieuw device:
1. Open de app via de volledige URL met token en Gist ID
2. Instellingen worden automatisch geladen
3. Google Agenda opnieuw koppelen via OAuth

---

## Op beginscherm zetten (iPhone)

1. Open de volledige URL in Safari
2. Tik op het deel-icoontje (vierkantje met pijl omhoog)
3. Kies **Zet op beginscherm**
4. Naam: Goedemorgen

---

## Bestanden

| Bestand | Omschrijving |
|---------|-------------|
| `goedemorgen.html` | De volledige app — één bestand |
| `CHANGELOG.md` | Versiegeschiedenis en wijzigingen |
| `README.md` | Dit bestand |

---

## Versie

Huidige versie: **1.3** — zie [CHANGELOG.md](CHANGELOG.md) voor alle wijzigingen.
