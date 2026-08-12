# robots-txt-checker

Ein Claude Skill für zwei Aufgaben rund um robots.txt: Live-Check einer bestehenden robots.txt gegen relevante Bots, und Erstellung/Generierung einer neuen robots.txt (z. B. für einen WooCommerce-Shop). Das Ergebnis beim Check ist ein strukturierter Report, der zeigt, wer erlaubt und wer geblockt ist, inklusive Einordnung der Auffälligkeiten. Bei der Erstellung liefert der Skill eine fertige, korrekt gruppierte robots.txt-Datei.

## Was der Skill macht

### Live-Check
- Prüft eine Domain gegen eine kuratierte Liste relevanter User-Agents, gegliedert in fünf Kategorien: Sichtbarkeit (Googlebot, Bingbot, DuckDuckBot, FacebookBot, Applebot, PerplexityBot, OAI-SearchBot, Claude-SearchBot und weitere), KI-Training (GPTBot, ClaudeBot, anthropic-ai, CCBot, Google-Extended und weitere), SEO-Tools (AhrefsBot, SemrushBot, DotBot, MJ12bot), Regional irrelevante Suchmaschinen (YandexBot, Baiduspider, Sogou, PetalBot und weitere, bei Sites mit klar abgegrenztem Zielmarkt) und Scraper/Spam (Bytespider, BLEXBot, MauiBot).
- Prüft neben der Startseite automatisch kritische Standardpfade wie /wp-admin/ und /wp-admin/admin-ajax.php sowie, je nach Shop-Typ, WooCommerce- und Feed-Pfade. Bei deutschsprachigen Shops werden dafür zuerst die tatsächlichen Pfade (z. B. /warenkorb/, /kasse/, /mein-konto/) anhand der Live-Seite ermittelt statt englische Standardpfade anzunehmen.
- Gibt das Ergebnis als Tabelle aus: Bot, Kategorie, Status pro geprüftem Pfad und die gefundene Regel.
- Ordnet Auffälligkeiten ein: kritische Probleme, die sofort gefixt werden sollten (z. B. gesperrte Sichtbarkeits-Bots oder gesperrte admin-ajax.php), Punkte zur Diskussion (z. B. Google-Extended) und Dinge, die absichtlich geblockt sein sollen.
- Bietet einen Kurzcheck-Modus für gezielte Einzelfragen wie "Darf FacebookBot auf domain.at?", ohne die volle Tabelle aufzubauen.

### robots.txt erstellen/generieren
- Analysiert vor dem Entwurf die bestehende, live veröffentlichte robots.txt sowie die tatsächliche URL-Struktur der Seite, statt blind zu überschreiben.
- Fragt vor der Erstellung gezielt die Business-Entscheidungen ab, die nicht automatisch angenommen werden sollten: Sollen KI-Trainingsbots geblockt oder differenziert behandelt werden (reine Trainingsbots blocken, sichtbarkeits-/GEO-relevante Bots wie OAI-SearchBot oder PerplexityBot erlauben)? Sollen SEO-Tool-Bots geblockt werden? Welcher Zielmarkt gilt, und welche regionalen Suchmaschinen sind damit irrelevant? Sollen Shop-Funktionsseiten wie Warenkorb/Kasse/Konto gesperrt werden?
- Fasst Bots mit identischen Regeln in einer gemeinsamen `User-agent`-Gruppe zusammen, statt für jeden Bot einen eigenen Block zu wiederholen.
- Legt für explizit erwünschte Bots keine eigenen `Allow`-Blöcke an, da robots.txt-Gruppen exklusiv matchen und ein eigener Block sonst die Sperren aus der `*`-Gruppe (z. B. /wp-admin/) für genau diesen Bot aufheben würde.
- Verwendet in der generierten Datei ausschließlich reine ASCII-Zeichen (keine Gedankenstriche, Pfeile oder typografischen Anführungszeichen), da robots.txt ohne garantiertes Encoding-Handling über alle Crawler/Server-Setups hinweg funktionieren muss.
- Weist darauf hin, dass eine neu erstellte Datei erst nach Veröffentlichung mit dem Live-Check verifiziert werden kann, und bietet diesen Check nach dem Hochladen an.

## Voraussetzungen

- Ein Claude-Produkt mit Skill-Unterstützung (z. B. Claude Code, Claude Desktop oder Claude.ai je nach Verfügbarkeit).
- Der MCP-Server `tamethebots-robotstxt` muss verbunden sein. Der Skill nutzt dessen Tool `get-robots-allowed` mit den Parametern `url` (vollständige URL inkl. https://) und optional `user_agent` (Default: Googlebot). Dieses Tool prüft ausschließlich die live veröffentlichte robots.txt – ein Entwurf kann damit nicht vorab getestet werden.
- Für die Erstellung/Generierung zusätzlich Web-Fetch-Zugriff, um die bestehende robots.txt und URL-Struktur der Zieldomain zu analysieren.

Details zur Skill- und MCP-Installation können sich je nach Claude-Version unterscheiden. Die aktuelle Dokumentation dazu findet sich auf [docs.claude.com](https://docs.claude.com).

## Installation

1. Repository klonen oder die Datei `SKILL.md` herunterladen.
2. Die Datei in das für dein Claude-Produkt vorgesehene Skills-Verzeichnis legen.
3. Sicherstellen, dass der `tamethebots-robotstxt` MCP verbunden ist.

## Verwendung

Der Skill greift automatisch, wenn im Chat Formulierungen wie diese vorkommen:
- "Prüf die robots.txt von domain.at"
- "Darf Googlebot auf domain.at?"
- "Check mal ob GPTBot durch darf"
- "Ist die Seite crawlbar?"
- "Teste die Konfiguration gegen Bots"
- "Erstell mir eine passende robots.txt für domain.at"
- "Bau mir eine robots.txt für meinen WooCommerce-Shop"

### Beispielausgabe (Check)

| Bot | Kategorie | / | /wp-admin/ | /wp-admin/admin-ajax.php |
|---|---|---|---|---|
| Googlebot | Sichtbarkeit | ✅ | ❌ | ✅ |
| GPTBot | KI-Training | ❌ | ❌ | ❌ |

Im Anschluss an die Tabelle benennt der Skill kritische Probleme und Diskussionspunkte im Klartext.

## Sonderfall Google-Extended

Google-Extended steuert sowohl die Nutzung für KI-Training als auch, teils, die Sichtbarkeit in AI Overviews und Gemini Search. Der Skill weist bei einer Sperre auf diese Doppelfunktion hin, statt sie pauschal als Problem oder als unproblematisch einzustufen.

## Lizenz

MIT
