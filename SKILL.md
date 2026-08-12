---
name: robots-txt-checker
description: >-
  Prüft live, ob URLs durch die robots.txt einer Domain erlaubt oder geblockt
  sind – für einen oder mehrere User-Agents. Nutzt den tamethebots MCP
  (`get-robots-allowed`). Auch anwenden beim Erstellen/Generieren einer neuen
  robots.txt (z.B. für einen WooCommerce-Shop) – inkl. Gruppierungs-,
  Zeichensatz- und Regional-Regeln.
---

# robots.txt Checker & Generator Skill

Prüft live, ob URLs durch die robots.txt einer Domain erlaubt oder geblockt sind – für einen oder mehrere User-Agents. Nutzt den tamethebots MCP (`get-robots-allowed`). Wird auch verwendet, um neue robots.txt-Dateien zu erstellen bzw. bestehende zu überarbeiten.

## Voraussetzung

Der `tamethebots-robotstxt` MCP muss verbunden sein. Das Tool heisst `tamethebots-robotstxt:get-robots-allowed` und nimmt:
- `url` (string, URI) - vollständige URL inkl. https://
- `user_agent` (string, optional) - Default: `Googlebot`

Wichtig: Das Tool prüft nur die tatsächlich live veröffentlichte robots.txt einer Domain. Ein selbst entworfener Entwurf (Draft) kann damit NICHT vorab getestet werden – erst nach dem Hochladen auf den Server.

---

## TEIL A: Prüfen (Check-Modus)

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
- `https://[domain]/shop/` oder `/produkte/` (bzw. deutsche Pfade: `/produkt-kategorie/...`)
- Cart/Checkout/Account-Pfade ermitteln (Sprache beachten! DE-Shops nutzen oft `/warenkorb/`, `/kasse/`, `/mein-konto/` statt `/cart/`, `/checkout/`, `/my-account/` – vor dem Check die tatsächliche Struktur per `web_fetch` der Startseite verifizieren, nicht blind die englischen Standardpfade annehmen)
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

**AI Training (sollten geblockt sein, außer explizit gewünscht):**
- `GPTBot`, `ClaudeBot`, `anthropic-ai`, `CCBot`, `Google-Extended`, `Applebot-Extended`, `Meta-ExternalAgent`, `Bytespider`, `Diffbot`, `Omgilibot`, `Omgili`, `ImagesiftBot`, `Timpibot`, `Webzio-Extended`

**SEO-Tools (sollten geblockt sein):**
- `AhrefsBot`, `SemrushBot`, `SemrushBot-SA`, `DotBot`, `MJ12bot`

**Regional irrelevante Suchmaschinen (bei Sites mit klar abgegrenztem Zielraum, z.B. AT-DE):**
- `YandexBot` (Russland/CIS), `Baiduspider` (China), `Sogou` (China), `360Spider` (China), `PetalBot` (Huawei, primär Asien), `Yeti` (Naver, Korea), `coccocbot-web` (Vietnam), `SeznamBot` (Tschechien), `Exabot` (Frankreich)
- Nur blocken, wenn der Zielmarkt das rechtfertigt (immer beim Nutzer nachfragen bzw. explizit im Kontext prüfen, welche Region relevant ist – nicht automatisch annehmen)

> ⚠️ MONOBUNT-Regel: `Screaming Frog SEO Spider` darf NIE in einer robots.txt für MONOBUNT-Kunden geblockt werden. MONOBUNT nutzt das Tool intern für eigene Audits. Falls Screaming Frog in einer bestehenden robots.txt geblockt ist, als Fehler markieren und Entfernung empfehlen.

**Scraper/Spam (sollten geblockt sein):**
- `Bytespider`, `BLEXBot`, `MauiBot`

### Schritt 4: MCP-Calls ausführen

Für jede Kombination aus Bot + Pfad einen Call machen:

```
tool: tamethebots-robotstxt:get-robots-allowed
url: "https://[domain]/[pfad]"
user_agent: "[BotName]"
```

Effizienz: Zuerst alle Sichtbarkeits-Bots gegen die Homepage checken. Dann geblockte Bots gegen die Homepage. Dann kritische Pfade (wp-admin, ajax, cart/checkout) separat.

### Schritt 5: Ergebnis aufbereiten

Ausgabe als strukturierte Tabelle mit:
- Bot-Name
- Kategorie (Sichtbarkeit / AI Training / SEO-Tool / Regional irrelevant / Scraper)
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
- Google-Extended geblockt: Erkläre Doppelfunktion (Training UND ggf. AI Overviews je nach aktuellem Stand von Google)

**MONOBUNT-spezifischer Fehler (kritisch melden):**
- Screaming Frog SEO Spider geblockt: Sofort als Fehler markieren, Entfernung empfehlen. MONOBUNT nutzt das Tool intern.

**Was nicht problematisch ist:**
- AI Training Bots geblockt = gewollt
- SEO Tools geblockt = gewollt
- Regional irrelevante Suchmaschinen geblockt bei klar abgegrenztem Zielmarkt = gewollt

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

---

## TEIL B: Erstellen/Generieren einer neuen robots.txt

### Vor dem Entwurf: bestehende Situation prüfen

1. Bestehende robots.txt per `web_fetch` unter `https://[domain]/robots.txt` holen und analysieren (nicht blind überschreiben – vorhandene, funktionale Regeln wie Plugin-Allow-Listen für Elementor/WP Rocket erhalten).
2. Tatsächliche URL-Struktur der Seite prüfen (Sprache! Deutsche WooCommerce-Shops nutzen oft `/warenkorb/`, `/kasse/`, `/mein-konto/` statt der englischen Standardpfade).
3. Kritische Business-Entscheidungen NICHT selbst treffen, sondern beim Nutzer erfragen (z.B. per AskUserQuestion):
   - Sollen AI-Trainingsbots geblockt werden, oder differenziert (reine Trainingsbots blocken, sichtbarkeits-/GEO-relevante wie OAI-SearchBot, ChatGPT-User, Claude-SearchBot, Claude-User, PerplexityBot erlauben)?
   - Sollen SEO-Tool-Bots (Ahrefs, Semrush etc.) geblockt werden?
   - Welcher Zielraum/Zielmarkt gilt – daraus ergibt sich, welche regionalen Suchmaschinen-Bots irrelevant sind und geblockt werden können/sollen.
   - Sollen Shop-Funktionsseiten (Cart/Checkout/Account) gesperrt werden?

### Zeichensatz-Regel (WICHTIG – immer anwenden)

In der robots.txt-Datei selbst (Direktiven UND Kommentare) ausschließlich reine ASCII-Zeichen verwenden. Keine Sonderzeichen, die je nach Server-Encoding, Editor oder Crawler-Parser falsch dargestellt oder falsch codiert werden können, insbesondere:
- Gedankenstriche/Halbgeviertstriche `–` `—` -> stattdessen normaler Bindestrich `-`
- Pfeile `→` `←` -> stattdessen ausschreiben ("und", "damit", "-\>") oder weglassen
- Typografische/"smarte" Anführungszeichen `„` `"` `'` `'` -> stattdessen einfache gerade Anführungszeichen `"` bzw. `'`
- Sonstige Unicode-Sonderzeichen (Aufzählungspunkte wie `•`, Emojis etc.) -> vermeiden, stattdessen `-` oder `#` für Kommentar-Gliederung nutzen

Grund: robots.txt ist eine reine Textdatei ohne garantiertes Encoding-Handling durch alle Crawler/Server-Setups; ASCII ist die sicherste Wahl. Diese Regel gilt für JEDE erstellte oder bearbeitete robots.txt, nicht nur für Vorschau-/Chat-Ausgaben.

### Gruppierungs-Regel (WICHTIG – immer anwenden)

robots.txt erlaubt mehrere `User-agent:`-Zeilen direkt hintereinander, gefolgt von gemeinsamen Direktiven – das bildet EINE Gruppe (RFC 9309, von allen relevanten Crawlern unterstützt). Bots mit identischen Regeln werden IMMER zusammengefasst, NICHT einzeln mit wiederholtem `Disallow: /` aufgelistet.

Richtig:
```
User-agent: GPTBot
User-agent: CCBot
User-agent: Bytespider
Disallow: /
```

Falsch (nicht mehr verwenden):
```
User-agent: GPTBot
Disallow: /

User-agent: CCBot
Disallow: /

User-agent: Bytespider
Disallow: /
```

Diese Regel gilt für alle Bot-Kategorien mit gleicher Regel: AI-Trainingsbots, SEO-Tools, regional irrelevante Suchmaschinen jeweils in einem gemeinsamen Block.

### Kritischer Mechanik-Hinweis: Gruppen sind exklusiv, nicht additiv

Ein Crawler matched IMMER nur die spezifischste für ihn zutreffende `User-agent`-Gruppe – Regeln aus der `*`-Gruppe werden für einen Bot mit eigener Gruppe NICHT zusätzlich angewendet.

Konsequenz: Für Bots, die bewusst ERLAUBT werden sollen (z.B. OAI-SearchBot, ChatGPT-User, Claude-SearchBot, PerplexityBot, Applebot, FacebookBot), KEINEN eigenen Block mit `Allow: /` anlegen – sonst verlieren genau diese Bots die Sperren aus der `*`-Gruppe (z.B. `/wp-admin/`, `/warenkorb/`, `/kasse/`). Stattdessen diese Bots einfach NICHT auflisten, damit sie automatisch unter die `*`-Gruppe fallen und deren Standard-Regeln (inkl. Sperren) erben.

Nur Bots, die VOLLSTÄNDIG anders behandelt werden sollen als die `*`-Gruppe (i.d.R. komplett gesperrt), bekommen eine eigene Gruppe.

### Aufbau einer generierten Datei (Reihenfolge)

1. `User-agent: *` Gruppe mit Standard-Disallow (wp-admin, wp-includes, Plugins/Cache, Login, Shop-Funktionsseiten je nach Nutzerentscheidung) + Allow-Ausnahmen (admin-ajax.php, Uploads, ggf. spezifische Plugin-Assets für Elementor/WP Rocket)
2. Gruppierter Block: AI-Trainingsdaten-Scraper (nur die wirklich zu blockenden, siehe Nutzerentscheidung)
3. Gruppierter Block: SEO-Tool-Bots (falls vom Nutzer gewünscht)
4. Gruppierter Block: regional irrelevante Suchmaschinen-Bots (falls Zielraum das rechtfertigt)
5. `Sitemap:`-Zeile am Ende

### Nach der Erstellung

- Datei liefern und explizit darauf hinweisen, dass sie erst nach Veröffentlichung mit dem Check-Modus (Teil A) gegen die Live-Version verifiziert werden kann.
- Anbieten, nach dem Hochladen einen Vollcheck durchzuführen.
- Vor der Auslieferung selbst nochmal auf reine ASCII-Zeichen (siehe Zeichensatz-Regel) und korrekte Gruppierung (siehe Gruppierungs-Regel) prüfen.
