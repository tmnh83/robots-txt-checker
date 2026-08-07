# robots-txt-checker

Ein Claude Skill, der eine robots.txt live gegen relevante Bots prüft: Suchmaschinen, KI-Crawler, SEO-Tools und bekannte Scraper. Das Ergebnis ist ein strukturierter Report, der zeigt, wer erlaubt und wer geblockt ist, inklusive Einordnung der Auffälligkeiten.

## Was der Skill macht

- Prüft eine Domain gegen eine kuratierte Liste relevanter User-Agents, gegliedert in vier Kategorien: Sichtbarkeit (Googlebot, Bingbot, DuckDuckBot, FacebookBot, Applebot, PerplexityBot, OAI-SearchBot, Claude-SearchBot und weitere), KI-Training (GPTBot, ClaudeBot, anthropic-ai, CCBot, Google-Extended), SEO-Tools (AhrefsBot, SemrushBot, DotBot, MJ12bot) und Scraper/Spam (Bytespider, BLEXBot, MauiBot).
- Prüft neben der Startseite automatisch kritische Standardpfade wie /wp-admin/ und /wp-admin/admin-ajax.php sowie, je nach Shop-Typ, WooCommerce- und Feed-Pfade.
- Gibt das Ergebnis als Tabelle aus: Bot, Kategorie, Status pro geprüftem Pfad und die gefundene Regel.
- Ordnet Auffälligkeiten ein: kritische Probleme, die sofort gefixt werden sollten (z. B. gesperrte Sichtbarkeits-Bots oder gesperrte admin-ajax.php), Punkte zur Diskussion (z. B. Google-Extended) und Dinge, die absichtlich geblockt sein sollen.
- Bietet einen Kurzcheck-Modus für gezielte Einzelfragen wie "Darf FacebookBot auf domain.at?", ohne die volle Tabelle aufzubauen.

## Voraussetzungen

- Ein Claude-Produkt mit Skill-Unterstützung (z. B. Claude Code, Claude Desktop oder Claude.ai je nach Verfügbarkeit).
- Der MCP-Server `tamethebots-robotstxt` muss verbunden sein. Der Skill nutzt dessen Tool `get-robots-allowed` mit den Parametern `url` (vollständige URL inkl. https://) und optional `user_agent` (Default: Googlebot).

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

### Beispielausgabe

| Bot | Kategorie | / | /wp-admin/ | /wp-admin/admin-ajax.php |
|---|---|---|---|---|
| Googlebot | Sichtbarkeit | ✅ | ❌ | ✅ |
| GPTBot | KI-Training | ❌ | ❌ | ❌ |

Im Anschluss an die Tabelle benennt der Skill kritische Probleme und Diskussionspunkte im Klartext.

## Sonderfall Google-Extended

Google-Extended steuert sowohl die Nutzung für KI-Training als auch, teils, die Sichtbarkeit in AI Overviews und Gemini Search. Der Skill weist bei einer Sperre auf diese Doppelfunktion hin, statt sie pauschal als Problem oder als unproblematisch einzustufen.

## Lizenz

MIT (nach eigenem Ermessen anpassen)
