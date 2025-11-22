# Analiza i Propozycje Usprawnienia Procesu Pozyskiwania Kontaktów

## Kontekst Biznesowy
**Cel:** Znalezienie adresów email do wysyłki propozycji współpracy w zakresie konsultingu IT

---

## 1. ANALIZA OBECNEGO STANU

### Co działa dobrze:
✓ Ekstrakcja emaili z JavaScript/JSON (jak w przypadku wkbw.com)
✓ Podstawowa filtracja (wykluczenie HR/rekrutacji)
✓ Próba sprawdzenia zarówno strony głównej jak i /contact

### Zidentyfikowane problemy:
❌ **Niska skuteczność** - w batch_000 (20 firm) znaleziono tylko 1 adres
❌ **Błędy dostępu** - 403/forbidden na niektórych stronach
❌ **Brak priorytetyzacji** - support@procore.com to wsparcie techniczne, nie bizdev
❌ **Brak kontekstu** - nie wiemy czy firma w ogóle jest dobrym targete'm
❌ **Jednowymiarowe podejście** - tylko scraping, brak alternatywnych źródeł

---

## 2. PROPOZYCJE USPRAWNIEŃ

### A) PRIORYTETYZACJA I KATEGORYZACJA EMAILI

**Problem:** Nie wszystkie emaile są równie wartościowe dla bizdev.

**Rozwiązanie:** System scoringu emaili

```
SCORING EMAILI (1-10):

TIER 1 - Najwyższa wartość (9-10):
- sales@domain.com
- partners@domain.com
- business@domain.com
- bizdev@domain.com
- partnerships@domain.com

TIER 2 - Wysoka wartość (7-8):
- info@domain.com
- contact@domain.com
- hello@domain.com
- inquiries@domain.com

TIER 3 - Średnia wartość (5-6):
- support@domain.com (wsparcie może przekierować)
- customerservice@domain.com

TIER 4 - Niska wartość (2-4):
- hr@domain.com (chyba że oferujemy rekrutację IT)
- jobs@domain.com
- press@domain.com
- media@domain.com

TIER 5 - Bez wartości (0-1):
- noreply@domain.com
- abuse@domain.com
- postmaster@domain.com
```

### B) STRATEGIA MULTI-PAGE CRAWLING

**Problem:** Sprawdzamy tylko stronę główną i /contact

**Rozwiązanie:** Szersze przeszukiwanie w określonej kolejności

```
KOLEJNOŚĆ SPRAWDZANIA STRON:

1. /contact, /contact-us → Najczęstsze miejsce kontaktów biznesowych
2. /sales, /get-quote → Dedykowane dla leadów
3. /partners, /partnership → Świetne dla współpracy B2B
4. /about, /about-us → Często zawiera "media inquiries" lub ogólny kontakt
5. /enterprise, /business → Dla rozwiązań biznesowych
6. Sitemap.xml → Może ujawnić ukryte strony kontaktowe
7. robots.txt → Czasem zawiera linki do stron kontaktowych
```

### C) GENERATOR COMMON EMAIL PATTERNS

**Problem:** Wiele firm nie publikuje emaili ale używa standardowych wzorców

**Rozwiązanie:** Generowanie i weryfikacja typowych adresów

```python
# Dla domeny: example.com

WZORCE DO SPRAWDZENIA:
1. info@example.com
2. sales@example.com
3. contact@example.com
4. hello@example.com
5. business@example.com
6. partners@example.com
7. partnerships@example.com
8. inquiries@example.com

METODY WERYFIKACJI:
- Sprawdzenie MX record (czy domena ma serwer mail)
- SMTP verification (czy skrzynka istnieje) - OSTROŻNIE z rate limiting
- Email hunter API (zewnętrzne narzędzie)
```

### D) PRE-VALIDATION TOOL (fetch-help-tool-spec.md)

**Implementacja sugerowanego narzędzia:**

```python
#!/usr/bin/env python3
"""
Pre-validation tool - sprawdza dostępność stron przed właściwym scrapingiem
"""

import asyncio
import aiohttp
from typing import List, Dict
import csv

async def check_url(session, url: str) -> Dict:
    """Sprawdza status HTTP dla danego URL"""
    try:
        async with session.get(f"https://{url}",
                              timeout=10,
                              allow_redirects=True) as response:
            return {
                'url': url,
                'status': response.status,
                'accessible': response.status < 400,
                'redirect': str(response.url) if response.url != f"https://{url}" else None
            }
    except Exception as e:
        return {
            'url': url,
            'status': 'ERROR',
            'accessible': False,
            'error': str(e)
        }

async def validate_batch(batch_file: str):
    """Przetwarza cały batch i zwraca tylko dostępne strony"""
    with open(batch_file, 'r') as f:
        urls = [line.strip() for line in f if line.strip()]

    async with aiohttp.ClientSession() as session:
        tasks = [check_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)

    # Zapisz tylko dostępne strony
    accessible = [r for r in results if r['accessible']]
    inaccessible = [r for r in results if not r['accessible']]

    return accessible, inaccessible

# UŻYCIE:
# python validate_batch.py batch_000.csv
# OUTPUT: batch_000_validated.csv (tylko dostępne strony)
```

### E) WZBOGACENIE DANYCH O KONTEKST

**Problem:** Nie wiemy czy firma jest dobrym targetem dla konsultingu IT

**Rozwiązanie:** Ekstrakcja dodatkowych informacji

```
INFORMACJE DO ZEBRANIA:

1. BRANŻA/SEKTOR
   - Retail, Finance, Healthcare, Tech, Manufacturing...
   - Pomaga targetować ofertę

2. WIELKOŚĆ FIRMY (jeśli dostępne)
   - Small business vs Enterprise
   - LinkedIn company size
   - Liczba pracowników ze strony "About"

3. STACK TECHNOLOGICZNY
   - Wykrycie używanych technologii (Wappalyzer-like)
   - Angular, React, .NET, Java...
   - Pokazuje gdzie możemy pomóc

4. SYGNAŁY POTRZEBY IT:
   - Czy mają sekcję "Careers" z ofertami IT? → mogą potrzebować wsparcia
   - Czy wspominają o "digital transformation"?
   - Czy mają dział "Innovation"?

FORMAT WYNIKU:
company,email,score,industry,size,tech_stack,signals
procore.com,support@procore.com,5,Construction Tech,Enterprise,"React,AWS",careers-it
```

### F) ALTERNATYWNE ŹRÓDŁA KONTAKTÓW

**Problem:** Wiele firm nie publikuje emaili na stronie

**Rozwiązanie:** Inne źródła danych

```
1. LINKEDIN:
   - LinkedIn Sales Navigator API
   - Strona firmowa → "See all employees" → Filtr "Business Development"
   - Często pokazuje kluczowych ludzi z kontaktami

2. CRUNCHBASE:
   - Informacje o startupach
   - Często zawiera kontakty do founders/executives

3. HUNTER.IO / APOLLO.IO:
   - Bazy danych emaili firmowych
   - Pokazują wzorce emaili w firmie

4. WHOIS DATA:
   - Dane administratora domeny
   - Często zawiera email kontaktowy

5. GITHUB (dla firm tech):
   - Organization page
   - Często zawiera kontakty do maintainerów
```

### G) STRATEGIA "SITEMAP CRAWLING"

**Implementacja sugestii z spec_d.md:**

```python
def find_contact_pages_from_sitemap(domain: str) -> List[str]:
    """
    Pobiera sitemap.xml i szuka stron zawierających kontakt
    """
    sitemap_urls = [
        f"https://{domain}/sitemap.xml",
        f"https://{domain}/sitemap_index.xml",
        f"https://{domain}/sitemap-index.xml",
    ]

    contact_keywords = [
        'contact', 'sales', 'partner', 'about',
        'inquiry', 'business', 'enterprise'
    ]

    # Parse sitemap XML
    # Filter URLs containing contact keywords
    # Return prioritized list

    return contact_page_urls

# To daje nam listę WSZYSTKICH potencjalnie interesujących stron
# zamiast zgadywania czy /contact istnieje
```

### H) SMART RETRY Z RÓŻNYMI USER-AGENTS

**Problem:** Niektóre strony blokują (403) standardowe crawlery

**Rozwiązanie:**

```python
USER_AGENTS = [
    # Standard browser
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...',
    # Mobile (czasem dają więcej danych)
    'Mozilla/5.0 (iPhone; CPU iPhone OS 14_0...)...',
    # Różne przeglądarki
    'Mozilla/5.0 (Macintosh; Intel Mac OS X...)...'
]

# Retry z różnymi UA jeśli dostaniemy 403
# Dodaj opóźnienia między requestami (respectful crawling)
```

---

## 3. PROPONOWANA NOWA ARCHITEKTURA

```
PIPELINE PRZETWARZANIA:

┌─────────────────────────────────────────┐
│ FAZA 1: PRE-VALIDATION                  │
│ - Check HTTP status                     │
│ - Filter out inaccessible (403, 404)   │
│ - Detect redirects                      │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│ FAZA 2: RECONNAISSANCE                  │
│ - Sitemap analysis                      │
│ - Identify contact pages               │
│ - Tech stack detection                 │
│ - Industry classification              │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│ FAZA 3: EMAIL EXTRACTION                │
│ - Scrape identified pages              │
│ - Extract from HTML/JS/JSON            │
│ - Generate common patterns             │
│ - Verify MX records                    │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│ FAZA 4: SCORING & ENRICHMENT            │
│ - Score emails by type                 │
│ - Add company context                  │
│ - Check alternative sources            │
│ - Remove duplicates/invalid            │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│ OUTPUT: PRIORITIZED CONTACT LIST        │
│ company | email | score | context      │
└─────────────────────────────────────────┘
```

---

## 4. FORMAT WYJŚCIOWY - PROPOZYCJA

**Obecny format:** `website,email`

**Proponowany format:**

```csv
company,email,email_type,score,industry,size,tech_stack,source,confidence,notes
procore.com,sales@procore.com,sales,9,Construction Tech,Enterprise,"React,AWS",scraped,high,"Found on /contact page"
procore.com,support@procore.com,support,5,Construction Tech,Enterprise,"React,AWS",scraped,high,"Alternative contact"
teradata.com,info@teradata.com,general,7,Data Analytics,Enterprise,"Java,Cloud",generated,medium,"Common pattern, MX verified"
```

**Korzyści:**
- Wiemy które emaile najpierw targetować (score)
- Mamy kontekst do personalizacji wiadomości (industry, tech_stack)
- Wiemy jak pewny jest dany kontakt (confidence)

---

## 5. QUICK WINS - CO ZAIMPLEMENTOWAĆ NAJPIERW?

### PRIORYTET 1 (Największy impact, mały effort):
1. ✅ **Email scoring system** - dodaj kolumnę 'score' do outputu
2. ✅ **Multi-page checking** - sprawdzaj /contact, /sales, /partners
3. ✅ **Pre-validation tool** - Python script z fetch-help-tool-spec.md

### PRIORYTET 2 (Duży impact, średni effort):
4. ✅ **Common pattern generator** - info@, sales@, contact@
5. ✅ **Sitemap crawling** - implementacja z spec_d.md
6. ✅ **Industry classification** - prosta kategoryzacja firm

### PRIORYTET 3 (Nice to have):
7. ⚠️ **Alternative sources** - LinkedIn, Hunter.io (wymaga API keys)
8. ⚠️ **Tech stack detection** - dodatkowa wartość dla targetowania
9. ⚠️ **Email verification** - SMTP check (ostrożnie z rate limiting)

---

## 6. PRZYKŁADOWA IMPLEMENTACJA - IMPROVED BATCH PROCESSOR

```python
#!/usr/bin/env python3
"""
Improved Email Finder - implementacja usprawnionego procesu
"""

import asyncio
import aiohttp
from bs4 import BeautifulSoup
import re
from typing import List, Dict, Optional
import dns.resolver
from urllib.parse import urljoin

class EmailFinder:

    EMAIL_SCORES = {
        'sales': 9, 'partners': 9, 'business': 9,
        'info': 7, 'contact': 7, 'hello': 7,
        'support': 5, 'help': 5,
        'hr': 2, 'jobs': 2, 'careers': 2,
        'noreply': 0, 'abuse': 0
    }

    CONTACT_PAGES = [
        '/contact', '/contact-us', '/contactus',
        '/sales', '/get-quote', '/request-demo',
        '/partners', '/partnership',
        '/about', '/about-us',
        '/enterprise', '/business'
    ]

    COMMON_PATTERNS = [
        'sales', 'info', 'contact', 'hello',
        'business', 'partners', 'inquiries'
    ]

    def __init__(self, domain: str):
        self.domain = domain
        self.found_emails = []

    async def find_emails(self) -> List[Dict]:
        """Main method - znajduje wszystkie emaile dla domeny"""

        # 1. Check sitemap
        sitemap_pages = await self.check_sitemap()

        # 2. Multi-page scraping
        pages_to_check = self.CONTACT_PAGES + sitemap_pages
        emails_found = await self.scrape_multiple_pages(pages_to_check)

        # 3. Generate common patterns
        generated = await self.generate_common_patterns()

        # 4. Score and deduplicate
        all_emails = emails_found + generated
        scored = self.score_emails(all_emails)

        return scored

    async def check_sitemap(self) -> List[str]:
        """Sprawdza sitemap.xml w poszukiwaniu stron kontaktowych"""
        # Implementation
        pass

    async def scrape_multiple_pages(self, pages: List[str]) -> List[Dict]:
        """Scrapuje wiele stron równolegle"""
        # Implementation
        pass

    async def generate_common_patterns(self) -> List[Dict]:
        """Generuje typowe wzorce emaili i weryfikuje MX"""
        emails = []
        for pattern in self.COMMON_PATTERNS:
            email = f"{pattern}@{self.domain}"
            if await self.verify_mx_record():
                emails.append({
                    'email': email,
                    'source': 'generated',
                    'confidence': 'medium'
                })
        return emails

    async def verify_mx_record(self) -> bool:
        """Sprawdza czy domena ma serwer mail"""
        try:
            dns.resolver.resolve(self.domain, 'MX')
            return True
        except:
            return False

    def score_emails(self, emails: List[Dict]) -> List[Dict]:
        """Przypisuje score emailom"""
        for email_data in emails:
            email = email_data['email']
            prefix = email.split('@')[0].lower()

            score = 5  # default
            for keyword, keyword_score in self.EMAIL_SCORES.items():
                if keyword in prefix:
                    score = keyword_score
                    break

            email_data['score'] = score

        return sorted(emails, key=lambda x: x['score'], reverse=True)

# USAGE:
# finder = EmailFinder('procore.com')
# results = await finder.find_emails()
```

---

## 7. PODSUMOWANIE I REKOMENDACJE

### Co robić dalej?

**KROK 1: Quick Win Implementation (1-2 dni)**
- Stwórz pre-validation tool (Python)
- Dodaj multi-page checking (/contact, /sales, /partners)
- Zaimplementuj email scoring

**KROK 2: Enhanced Extraction (3-5 dni)**
- Sitemap crawling
- Common pattern generation z MX verification
- Lepszy format outputu (CSV z dodatkowymi kolumnami)

**KROK 3: Context & Intelligence (1 tydzień)**
- Industry classification
- Tech stack detection
- Alternative sources (LinkedIn, Hunter.io)

### Oczekiwane rezultaty:

**Przed:**
- 1/20 użytecznych emaili (5% skuteczność)
- Brak kontekstu firmowego
- Wiele błędów dostępu

**Po usprawnieniach:**
- 10-15/20 emaili (50-75% skuteczność)
- Priorytetyzacja według wartości biznesowej
- Kontekst do personalizacji outreach
- Pre-filtrowane accessible domains

---

## 8. PYTANIA DO ROZWAŻENIA

1. **Skala operacji**: Ile batch'y planujesz przetwarzać? (wpływa na potrzebę rate limiting)

2. **Budget**: Czy możesz użyć płatnych API (Hunter.io, Apollo)? Znacznie zwiększy skuteczność

3. **Compliance**: Czy masz zgodę na scraping? (niektóre firmy mają terms against it)

4. **Target audience**: Czy wszystkie firmy to potencjalni klienci? Może lepiej pre-filtrować branże?

5. **Follow-up**: Jak planujesz weryfikować czy emaile są aktywne przed wysyłką?

---

**Gotowy do implementacji któregoś z tych usprawnień? Powiedz które cię najbardziej interesują!**
