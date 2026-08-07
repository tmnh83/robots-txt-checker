---
name: robots-txt-checker
description: "Prüft eine robots.txt live gegen relevante Bots via tamethebots MCP. Immer anwenden wenn der Nutzer fragt ob ein Bot geblockt oder erlaubt ist, eine robots.txt validiert werden soll, geprüft werden soll ob Googlebot, KI-Crawler oder SEO-Tools Zugriff haben, oder Begriffe wie \"robots.txt check\", \"Bot geblockt?\", \"crawlbar?\", \"darf Googlebot?\", \"ist XY erlaubt?\" vorkommen. Auch anwenden bei \"prüf die robots.txt\", \"check mal ob GPTBot durch darf\", \"teste die Konfiguration gegen Bots\". Erfordert den tamethebots-robotstxt MCP (get-robots-allowed Tool)."
---

# robots.txt Checker Skill

Prüft live, ob URLs durch die robots.txt einer Domain erlaubt oder geblockt sind – für einen oder mehrere User-Agents. Nutzt den tamethebots MCP (`get-robots-allowed`).

## Voraussetzung

Der `tamethebots-robotstxt` MCP muss verbunden sein. Das Tool heisst `tamethebots-robotstxt:get-robots-allowed` und nimmt:
- `url` (string, URI) - vollständige URL inkl. https://
- `user_agent` (string, optional) - Default: `Googlebot`

## Workflow

### Schritt 1: Input klären

Aus dem Kontext ableiten oder den Nutzer fragen:
- **Domain** - welche Website soll geprüft werden? (z.B. `monobunt.at`)
- **Modus** - Vollcheck (alle Kategorien) oder gezielt einzelne Bots?
- **Pfade** - Standard-Pfade reichen meist; bei WooCommerce/Shop auch Shop-Pfade prüfen

Wenn der Nutzer nur eine Domain nennt ohne weitere Angaben: Vollcheck mit Standard-Pfaden durchführen.

### Schritt 2: Prüfpfade zusammenstellen

Standard-Pfade (immer prüfen):
- `https://[domain]/` (Homepage)
- `https://[domain]/wp-admin/` (sollte geblockt sein)
- `https://[domain]/wp-admin/admin-ajax.php` (sollte erlaubt sein)

Bei WooCommerce/Shop zusätzlich:
- `https://[domain]/shop/` oder `/produkte/`
- `https://[domain]/wp-content/uploads/wc-logs/`
- `https://[domain]/wp-content/uploads/woocommerce_uploads/`

Bei Feeds (wenn SPAM-Backlink-Blocker aktiv):
- `https://[domain]/feed/`

### Schritt 3: Bot-Gruppen definieren

Prüfe in dieser Reihenfolge, jeweils mit repräsentativen Agents:

**Sichtbarkeit (müssen erlaubt sein):**
- `Googlebot`
- `Bingbot`
- `DuckDuckBot`
- `FacebookBot` (bei Shops besonders wichtig)
- `Applebot`
- `PerplexityBot`
- `OAI-SearchBot`
- `Claude-SearchBot`
- `Googlebot-Image`
- `Googlebot-Video`

**AI Training (sollten geblockt sein):**
- `GPTBot`
- `ClaudeBot`
- `anthropic-ai`
- `CCBot`
- `Google-Extended` (Sonderfall: kann AI Overviews beeinflussen, siehe Hinweis unten)

**SEO-Tools (sollten geblockt sein):**
- `AhrefsBot`
- `SemrushBot`
- `DotBot`
- `MJ12bot`

**Scraper/Spam (sollten geblockt sein):**
- `Bytespider`
- `BLEXBot`
- `MauiBot`

### Schritt 4: MCP-Calls ausführen

Für jede Kombination aus Bot + Pfad einen Call machen:

```
tool: tamethebots-robotstxt:get-robots-allowed
url: "https://[domain]/[pfad]"
user_agent: "[BotName]"
```

Effizienz: Zuerst alle Sichtbarkeits-Bots gegen die Homepage checken. Dann geblockte Bots gegen die Homepage. Dann kritische Pfade (wp-admin, ajax) separat.

### Schritt 5: Ergebnis aufbereiten

Ausgabe als strukturierte Tabelle mit:
- Bot-Name
- Kategorie (Sichtbarkeit / AI Training / SEO-Tool / Scraper)
- Status pro geprüftem Pfad (✅ erlaubt / ❌ geblockt)
- Gefundene Regel (wenn vom MCP zurückgegeben)

**Tabellenformat:**

| Bot | Kategorie | / | /wp-admin/ | /wp-admin/admin-ajax.php |
|---|---|---|---|---|
| Googlebot | Sichtbarkeit | ✅ | ❌ | ✅ |
| GPTBot | AI Training | ❌ | ❌ | ❌ |

### Schritt 6: Problemanalyse

Nach der Tabelle: Auffälligkeiten benennen.

**Kritische Probleme (sofort fixen):**
- Sichtbarkeits-Bot ist für `/` geblockt
- `/wp-admin/admin-ajax.php` ist geblockt (bricht Formulare etc.)
- FacebookBot geblockt bei einem Shop (kaputte Link-Previews)

**Hinweise (zur Diskussion):**
- Google-Extended geblockt: Erkläre Doppelfunktion (Training UND AI Overviews)
- Screaming Frog geblockt: Darauf hinweisen, dass eigene Audits "Respect robots.txt" deaktivieren müssen

**Was nicht problematisch ist:**
- AI Training Bots geblockt = gewollt
- SEO Tools geblockt = gewollt
- Yandex/Baidu geblockt bei DACH-fokussierten Sites = gewollt

## Sonderfall: Google-Extended

Google-Extended hat eine Doppelfunktion:
- Blockiert: Kein AI Training-Datensatz
- Blockiert: Möglicherweise schlechtere Sichtbarkeit in AI Overviews / Gemini Search

Bei Shops mit internationalem Fokus: Empfehlung zum Freischalten. Bei reinen DACH-Shops: kann geblockt bleiben, Auswirkung aktuell noch gering.

## Kurzcheck-Modus

Wenn der Nutzer nur einen spezifischen Bot oder eine spezifische URL prüfen will (z.B. "Darf FacebookBot auf spektraled.at?"):
- Nur diesen einen Call machen
- Ergebnis direkt und knapp ausgeben
- Keine volle Tabelle aufbauen

## Fehlerbehandlung

Wenn der MCP keinen Zugriff auf die robots.txt bekommt:
- Domain prüfen (https:// vergessen?)
- Darauf hinweisen, dass die robots.txt möglicherweise nicht erreichbar ist
- Als Alternative: Nutzer kann die robots.txt unter `https://[domain]/robots.txt` manuell aufrufen

Wenn `user_agent` nicht erkannt wird:
- Exakte Schreibweise aus der robots.txt verwenden (case-sensitive bei manchen Parsern)
- Leerzeichen in Agent-Namen können Probleme machen (z.B. "sogou spider")