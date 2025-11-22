# SPEC TURBO - Unified Email Extraction & Company Scoring Pipeline

**Version:** 1.0
**Date:** 2025-11-15
**Purpose:** Complete specification for finding business contacts and scoring companies for AWS cloud consulting partnerships

---

## 🎯 BUSINESS OBJECTIVE

**Goal:** Find high-value business contacts (email addresses) at companies that are ideal targets for AWS cloud consulting services.

**Our Firm:**
- 3-person boutique AWS cloud consulting
- After-hours projects (everyone has full-time jobs)
- Two business models: (1) Project consulting, (2) Staff augmentation/outsourcing

**Target Companies:**
- 50-500 employees
- Financial stability (Series A-C or profitable)
- Traditional industries needing cloud migration (FinTech, HealthTech, E-commerce, Manufacturing)
- NOT: Competitors, tech giants, cloud-native startups

---

## 📊 COMPLETE PROCESSING PIPELINE

```
┌────────────────────────────────────────────────────────┐
│ PHASE 0: BATCH SOURCING & PRE-VALIDATION              │
│ - Source from: Job boards, Crunchbase, LinkedIn       │
│ - Pre-validate: Check HTTP status, filter 403/404     │
│ - Output: Validated list of accessible domains        │
└──────────────────┬─────────────────────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────────────────────┐
│ PHASE 1: COMPANY PRE-FILTER (Quick Score)             │
│ - Size check (10-5,000 employees)                     │
│ - AWS Partner check (reject Advanced/Premier)         │
│ - Industry check (reject consulting/tech giants)      │
│ - Cloud signal scan (job postings, news)              │
│ - Entity verification (domain matches company)        │
│ Time: 5-10 min per company                            │
│ Output: PASS/REJECT/SUSPICIOUS + priority flag        │
└──────────────────┬─────────────────────────────────────┘
                   │
                   ▼ (Only PASS companies)
┌────────────────────────────────────────────────────────┐
│ PHASE 2: EMAIL RECONNAISSANCE & EXTRACTION            │
│ - Sitemap analysis (find all contact pages)           │
│ - Multi-page scraping (/contact, /sales, /partners)   │
│ - Deep extraction (HTML, JavaScript, JSON)            │
│ - Pattern generation (info@, sales@, partners@)       │
│ - MX record verification                              │
│ Time: 10-15 min per company                           │
│ Output: List of emails with sources                   │
└──────────────────┬─────────────────────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────────────────────┐
│ PHASE 3: EMAIL SCORING & FILTERING                    │
│ - Score emails by type (sales=9, info=7, support=5)   │
│ - Filter out HR/recruitment emails                    │
│ - Remove duplicates & invalid                         │
│ - Rank by business value                              │
│ Time: 2-5 min per company                             │
│ Output: Prioritized email list with scores            │
└──────────────────┬─────────────────────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────────────────────┐
│ PHASE 4: COMPANY DEEP SCORING                         │
│ - Financial stability (0-30 pts)                      │
│ - Cloud need (0-25 pts)                               │
│ - Company fit (0-20 pts)                              │
│ - Reputation (0-15 pts)                               │
│ - Accessibility (0-10 pts)                            │
│ Time: 15-20 min per company                           │
│ Output: Company score (0-100) + recommendation        │
└──────────────────┬─────────────────────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────────────────────┐
│ PHASE 5: ENRICHMENT & OUTPUT                          │
│ - Add company context (industry, size, tech stack)    │
│ - Identify opportunities (migration, staff aug)       │
│ - Generate outreach recommendations                   │
│ - Create final CSV + JSON reports                     │
│ Output: Prioritized contact list ready for outreach   │
└────────────────────────────────────────────────────────┘
```

---

## PHASE 0: BATCH SOURCING & PRE-VALIDATION

### A) BATCH SOURCING STRATEGY

**TIER 1 - BEST SOURCES (70-80% success rate):**

**1. Job Board Scraping (FREE, HIGHEST ROI)**
```
Platforms: Indeed, LinkedIn Jobs, Greenhouse
Search terms: "AWS Cloud Engineer", "DevOps Engineer (AWS)", "Cloud Migration Specialist"
Filter: Posted last 30 days
Rationale: If hiring = clear budget + need
Action: Extract company names from job postings
```

**2. Crunchbase Advanced Search ($30/month)**
```
Filters:
- Funding: Series A, B, C
- Employees: 51-500
- Industries: FinTech, HealthTech, E-commerce, SaaS
- Founded: 2010-2020
- Geography: [Your target market]
```

**3. LinkedIn Sales Navigator ($99/month)**
```
Filters:
- Company size: 51-500
- Industry: Financial Services, Healthcare, E-commerce
- Job postings: "cloud", "AWS", "DevOps", "migration"
- Growth: Hiring in last 6 months
```

**TIER 2 - GOOD SOURCES (40-60% success rate):**
- G2/Capterra review mining (companies reviewing on-premise software)
- Industry conference attendee lists
- AWS Marketplace buyers (if accessible)
- "Best Places to Work" lists (filter 50-500 employees, non-tech)

**AVOID:**
- ❌ Random company lists (5-20% success rate)
- ❌ Web scraping without filters
- ❌ Purchased email lists (spam traps, outdated)

### B) PRE-VALIDATION TOOL

**Purpose:** Check which domains are accessible before scraping

```python
#!/usr/bin/env python3
"""
Pre-validation: Check HTTP status for batch of domains
"""
import asyncio
import aiohttp

async def validate_url(session, domain):
    try:
        async with session.get(f"https://{domain}",
                              timeout=10,
                              allow_redirects=True) as resp:
            return {
                'domain': domain,
                'status': resp.status,
                'accessible': resp.status < 400,
                'redirect': str(resp.url) if resp.url != f"https://{domain}" else None
            }
    except Exception as e:
        return {'domain': domain, 'status': 'ERROR', 'accessible': False, 'error': str(e)}

async def validate_batch(domains):
    async with aiohttp.ClientSession() as session:
        tasks = [validate_url(session, d) for d in domains]
        return await asyncio.gather(*tasks)

# USAGE: Only process accessible domains in next phases
# Eliminates 403/404 errors upfront
```

**Output:** `batch_XXX_validated.csv` (only accessible domains)

---

## PHASE 1: COMPANY PRE-FILTER (Quick Score)

**Purpose:** Eliminate obvious bad fits BEFORE deep analysis (saves 60% time)

**Time per company:** 5-10 minutes

### STEP-BY-STEP PROCESS:

**STEP 1: Size Check (2 min)**
```
Check: LinkedIn company page OR website About section
Action: Count employees
Decision:
- AUTO-REJECT if <10 (too small, no budget)
- AUTO-REJECT if >5,000 (enterprise procurement nightmare)
- PASS if 10-5,000 → Continue to Step 2
```

**STEP 2: AWS Partner Check (1 min)**
```
Check: search "site:aws.amazon.com/partners [company name]"
       OR https://partners.amazonaws.com/
Decision:
- AUTO-REJECT if Advanced/Premier Partner (they're the experts!)
- AUTO-REJECT if Select Partner in cloud consulting
- PASS if not a partner OR Bronze → Continue to Step 3
```

**STEP 3: Industry/Competitor Check (2 min)**
```
Check: Visit homepage, read "What we do" / "Services"
Decision:
- AUTO-REJECT if:
  * Cloud consulting firm
  * DevOps consultancy
  * IT staff augmentation
  * Tech giant (Microsoft, Google, Amazon, Meta, Apple, IBM, Oracle)
  * In exclusion list (smabbler.com, etc.)
- PASS if product company, traditional industry → Continue to Step 4
```

**STEP 4: Cloud Need Signal Scan (3 min)**
```
Check:
- Careers page: Look for "cloud", "AWS", "DevOps", "migration"
- Google: "[company] cloud migration" OR "[company] digital transformation"

Flag:
- POSITIVE: Job postings mention AWS/cloud, recent cloud news
- NEUTRAL: No signals (traditional company, no tech presence)
- NEGATIVE: Cloud-native SaaS (already experts, wrong target)
```

**STEP 5: Entity Verification (2 min)**
```
Check:
- Does website content match domain/company name?
- Google "[domain] company" - do results align?

Flag SUSPICIOUS if:
- Search results show different company
- Domain recently changed hands
- Website content outdated/placeholder
```

### OUTPUT FORMAT:

```json
{
  "company_domain": "example.com",
  "pre_filter_result": "PASS | REJECT | SUSPICIOUS",
  "reason": "Traditional company (120 employees), no AWS partnership, hiring cloud engineers",
  "priority_flag": "HIGH | MEDIUM | LOW",
  "cloud_signals": ["Job posting: AWS Solutions Architect", "Blog mentions cloud migration"],
  "estimated_time_saved_if_reject": "25 min"
}
```

**Batch Pre-Filter Summary:**
```
Total companies: 20
Passed: 8 (40%)
Rejected: 10 (50%)
Suspicious: 2 (10%)
Time saved: 250 minutes
Proceed to Phase 2 with: 8 companies
```

---

## PHASE 2: EMAIL RECONNAISSANCE & EXTRACTION

**Purpose:** Find all business contact emails for companies that passed pre-filter

**Time per company:** 10-15 minutes

### A) MULTI-PAGE CRAWLING STRATEGY

**Pages to check (in priority order):**

1. **Primary targets (check first):**
   - `/contact`, `/contact-us`, `/contactus`
   - `/sales`, `/get-quote`, `/request-demo`
   - `/partners`, `/partnership`, `/partnerships`

2. **Secondary targets:**
   - `/about`, `/about-us`, `/company`
   - `/enterprise`, `/business`

3. **Discovery methods:**
   - `sitemap.xml`, `sitemap_index.xml` → Parse for contact-related URLs
   - `robots.txt` → Sometimes lists contact pages
   - Footer links → Extract all hrefs containing "contact", "sales", "partner"

### B) EMAIL EXTRACTION TECHNIQUES

**1. Visible email addresses:**
```python
import re

email_pattern = r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'
emails = re.findall(email_pattern, page_html)
```

**2. JavaScript/JSON extraction:**
```python
# Common patterns:
# window.contact_email = "sales@example.com"
# {"email": "info@example.com"}
# data-email="partners@example.com"

js_patterns = [
    r'["\']?email["\']?\s*[:=]\s*["\']([^"\']+@[^"\']+)["\']',
    r'mailto:([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})',
    r'data-email=["\']([^"\']+@[^"\']+)["\']'
]
```

**3. Obfuscated emails:**
```python
# "sales [at] company [dot] com"
# "sales@company·com" (non-standard chars)
# HTML entities: "sales&#64;company&#46;com"

def deobfuscate_email(text):
    text = text.replace('[at]', '@').replace('(at)', '@')
    text = text.replace('[dot]', '.').replace('(dot)', '.')
    text = text.replace('·', '.')
    # HTML entity decode
    import html
    return html.unescape(text)
```

**4. Contact form hidden fields:**
```python
# <input type="hidden" name="to" value="contact@example.com">
# <form action="mailto:sales@example.com">

soup = BeautifulSoup(html, 'html.parser')
hidden_emails = soup.find_all('input', {'type': 'hidden', 'name': 'to'})
form_actions = soup.find_all('form', {'action': re.compile(r'mailto:')})
```

### C) COMMON PATTERN GENERATION

**Generate & verify standard email patterns:**

```python
COMMON_PATTERNS = [
    'sales', 'partners', 'partnerships', 'business', 'bizdev',
    'info', 'contact', 'hello', 'inquiries',
    'support', 'help'  # Lower priority but include
]

async def generate_patterns(domain):
    emails = []
    for pattern in COMMON_PATTERNS:
        email = f"{pattern}@{domain}"
        if await verify_mx_record(domain):  # Only if domain has mail server
            emails.append({
                'email': email,
                'source': 'generated',
                'confidence': 'medium'
            })
    return emails

async def verify_mx_record(domain):
    """Check if domain has MX record (mail server)"""
    try:
        import dns.resolver
        dns.resolver.resolve(domain, 'MX')
        return True
    except:
        return False
```

### D) SITEMAP CRAWLING

```python
def find_contact_pages_from_sitemap(domain):
    """Parse sitemap.xml to find contact-related pages"""
    sitemap_urls = [
        f"https://{domain}/sitemap.xml",
        f"https://{domain}/sitemap_index.xml",
        f"https://{domain}/sitemap-index.xml"
    ]

    contact_keywords = ['contact', 'sales', 'partner', 'about', 'inquiry', 'business']

    # Fetch & parse XML
    # Filter URLs containing keywords
    # Return prioritized list

    return contact_page_urls
```

### E) SMART RETRY WITH USER-AGENTS

**Handle 403 blocks:**

```python
USER_AGENTS = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X)',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)'
]

async def fetch_with_retry(url, max_retries=3):
    for i, ua in enumerate(USER_AGENTS[:max_retries]):
        try:
            headers = {'User-Agent': ua}
            async with session.get(url, headers=headers, timeout=10) as resp:
                if resp.status == 200:
                    return await resp.text()
                await asyncio.sleep(2 * (i + 1))  # Respectful crawling
        except:
            continue
    return None
```

### OUTPUT FORMAT:

```json
{
  "company_domain": "example.com",
  "emails_found": [
    {
      "email": "partners@example.com",
      "source": "scraped",
      "location": "/partners page",
      "confidence": "high",
      "extraction_method": "visible_text"
    },
    {
      "email": "sales@example.com",
      "source": "scraped",
      "location": "/contact page - JavaScript",
      "confidence": "high",
      "extraction_method": "javascript_extraction"
    },
    {
      "email": "info@example.com",
      "source": "generated",
      "location": "common_pattern",
      "confidence": "medium",
      "extraction_method": "pattern_generation_mx_verified"
    }
  ],
  "pages_checked": ["/contact", "/sales", "/partners", "/about", "sitemap.xml"],
  "total_emails": 3
}
```

---

## PHASE 3: EMAIL SCORING & FILTERING

**Purpose:** Rank emails by business value, filter out useless contacts

**Time per company:** 2-5 minutes

### EMAIL SCORING RUBRIC

```python
EMAIL_SCORES = {
    # TIER 1 - Highest Value (9-10)
    'sales': 9,
    'partners': 9,
    'partnerships': 9,
    'business': 9,
    'bizdev': 9,

    # TIER 2 - High Value (7-8)
    'info': 7,
    'contact': 7,
    'hello': 7,
    'inquiries': 7,

    # TIER 3 - Medium Value (5-6)
    'support': 5,
    'help': 5,
    'customerservice': 5,

    # TIER 4 - Low Value (2-4) - Usually EXCLUDE
    'hr': 2,
    'jobs': 2,
    'careers': 2,
    'press': 3,
    'media': 3,

    # TIER 5 - No Value (0-1) - ALWAYS EXCLUDE
    'noreply': 0,
    'abuse': 0,
    'postmaster': 0,
    'webmaster': 1
}

def score_email(email):
    """Score email address by prefix"""
    prefix = email.split('@')[0].lower()

    for keyword, score in EMAIL_SCORES.items():
        if keyword in prefix:
            return score

    return 5  # Default neutral score
```

### FILTERING RULES

**Exclude these emails entirely:**

```python
def should_exclude(email):
    """Return True if email should be excluded"""
    exclude_patterns = [
        'noreply', 'no-reply', 'donotreply',
        'abuse', 'postmaster', 'webmaster',
        'hr@', 'jobs@', 'careers@', 'recruiting@',
        'press@', 'media@', 'marketing@'
    ]

    prefix = email.split('@')[0].lower()
    return any(pattern in prefix for pattern in exclude_patterns)
```

### DEDUPLICATION & VALIDATION

```python
def deduplicate_and_validate(emails):
    """Remove duplicates and validate format"""
    seen = set()
    valid_emails = []

    for email_data in emails:
        email = email_data['email'].lower().strip()

        # Skip if seen
        if email in seen:
            continue

        # Validate format
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email):
            continue

        # Skip if should exclude
        if should_exclude(email):
            continue

        seen.add(email)
        email_data['email'] = email
        email_data['score'] = score_email(email)
        valid_emails.append(email_data)

    # Sort by score (highest first)
    return sorted(valid_emails, key=lambda x: x['score'], reverse=True)
```

### OUTPUT FORMAT:

```csv
company,email,email_type,score,source,confidence,location
example.com,partners@example.com,partners,9,scraped,high,"/partners page"
example.com,sales@example.com,sales,9,scraped,high,"/contact - JavaScript"
example.com,info@example.com,info,7,generated,medium,"common pattern (MX verified)"
example.com,support@example.com,support,5,scraped,high,"/contact page footer"
```

---

## PHASE 4: COMPANY DEEP SCORING

**Purpose:** Evaluate if company is good target for AWS consulting

**Time per company:** 15-20 minutes

### SCORING METHODOLOGY (0-100 points)

**Total Score = Financial (30) + Cloud Need (25) + Fit (20) + Reputation (15) + Accessibility (10)**

#### 1. FINANCIAL STABILITY (0-30 points)

**25-30 pts:** Public company, profitable, or Series B+ with strong revenue
**18-24 pts:** Series A funded, steady revenue growth, established customers
**10-17 pts:** Seed or bootstrapped with revenue, sustainable
**5-9 pts:** Pre-revenue but funded, unclear sustainability
**0-4 pts:** No funding, no revenue, financial distress

**Data sources:**
- Crunchbase (funding, investors, revenue range)
- Company website (About, Press releases)
- LinkedIn (employee growth trend)
- News (recent funding announcements)

#### 2. CLOUD ADOPTION NEED (0-25 points)

**20-25 pts:** Pre-cloud or early migration, clear AWS need
- On-premise infrastructure visible
- Job posts for "cloud migration"
- No AWS certifications on team

**12-19 pts:** Partial cloud adoption, optimization opportunities
- Some AWS usage but basic
- Looking for serverless/containers
- AWS Well-Architected candidates

**5-11 pts:** Cloud-native but may need specialists
- Already on AWS
- Advanced use cases
- Temporary capacity needs

**0-4 pts:** No cloud need or non-AWS

**Data sources:**
- Job postings (AWS mentioned? Migration projects?)
- Engineering blog (tech stack?)
- BuiltWith/Wappalyzer (cloud provider detection)
- AWS Partner listings

#### 3. COMPANY FIT (0-20 points)

**16-20 pts:** Perfect fit
- Product company, 50-500 employees
- Tech-adjacent industry (FinTech, HealthTech)
- Boutique-friendly culture

**10-15 pts:** Good fit
- 10-50 or 500-1,000 employees
- Product or hybrid
- Works with small vendors

**5-9 pts:** Moderate fit
- Service company
- Very small (<10) or large (>1,000)
- Enterprise procurement

**0-4 pts:** Poor fit
- Wrong industry/model
- Enterprise-only vendors

**Criteria:**
- Employee count (LinkedIn, About)
- Business model (Product vs Service)
- Industry category
- Geographic location (time zones)

#### 4. REPUTATION (0-15 points)

**12-15 pts:** Excellent
- Glassdoor 4.0+
- Positive contractor reviews
- Industry awards
- Strong employer brand

**8-11 pts:** Good
- Glassdoor 3.5-3.9
- No major red flags
- Some positive signals

**4-7 pts:** Neutral/Unknown
- Limited reviews
- Mixed feedback
- New company

**0-3 pts:** Poor
- Glassdoor <3.5
- Payment issues reported
- High turnover

**Data sources:**
- Glassdoor (ratings, contractor reviews)
- Clutch.co (client reviews)
- BBB (complaints)
- Google "[company] contractor payment issues"

#### 5. ACCESSIBILITY (0-10 points)

**8-10 pts:** Highly accessible
- Clear contact for partnerships/sales
- Responsive
- Active LinkedIn
- Small decision chain

**5-7 pts:** Moderately accessible
- Generic contact forms
- Medium company
- May need warm intro

**2-4 pts:** Low accessibility
- Enterprise procurement
- Hard to reach decision makers
- Long sales cycles

**0-1 pts:** Inaccessible
- No contact info
- Auto-reject small vendors

**Indicators:**
- Contact page structure
- Sales/partnerships email available
- LinkedIn presence of key personnel

### SCORING DECISION TREE

```
IF score >= 70:
  RECOMMENDATION: HIGH_PRIORITY
  ACTION: Immediate outreach

ELIF score >= 40:
  RECOMMENDATION: MEDIUM_PRIORITY
  ACTION: Targeted outreach, low time investment

ELSE:
  RECOMMENDATION: AVOID or LOW_PRIORITY
  ACTION: Skip or very low-effort ping
```

### OUTPUT FORMAT:

```json
{
  "company_domain": "example.com",
  "company_name": "Example Corp",
  "overall_score": 76,
  "recommendation": "HIGH_PRIORITY",

  "score_breakdown": {
    "financial_stability": {"score": 24, "max": 30, "evidence": "Series B ($15M), 200% YoY growth"},
    "cloud_need": {"score": 22, "max": 25, "evidence": "Job posting: AWS Cloud Engineer, blog mentions migration"},
    "company_fit": {"score": 18, "max": 20, "evidence": "B2B SaaS, 120 employees, tech-forward"},
    "reputation": {"score": 11, "max": 15, "evidence": "Glassdoor 3.8, no payment issues"},
    "accessibility": {"score": 7, "max": 10, "evidence": "partners@example.com visible, mid-market sales"}
  },

  "company_profile": {
    "industry": "FinTech",
    "employee_count": 120,
    "estimated_revenue": "$10M-$50M",
    "funding_stage": "Series B",
    "headquarters": "Austin, TX",
    "business_model": "B2B SaaS"
  },

  "cloud_maturity": {
    "current_state": "Pre-Cloud / On-Premise",
    "aws_usage": "None detected",
    "migration_indicators": [
      "Job: AWS Solutions Architect",
      "Blog: 'Cloud Migration Roadmap Q3 2024'",
      "CTO interview mentions AWS move"
    ],
    "opportunity_type": "Migration Project + Support"
  },

  "opportunities": [
    {
      "type": "Migration Project",
      "description": "Migrate core platform to AWS",
      "confidence": "High",
      "timeline": "6-12 months",
      "engagement_model": "Project consulting"
    }
  ],

  "next_steps": [
    "Reach out via partners@example.com",
    "Reference their Q3 blog on cloud migration",
    "Offer free AWS Well-Architected review",
    "Position as boutique AWS experts"
  ]
}
```

---

## PHASE 5: ENRICHMENT & OUTPUT

**Purpose:** Add context, generate final prioritized contact list

### FINAL CSV OUTPUT FORMAT

```csv
company,company_score,recommendation,email,email_score,industry,size,tech_stack,opportunity,source,confidence,next_action
example.com,76,HIGH_PRIORITY,partners@example.com,9,FinTech,120,"on-prem",Migration Project,scraped,high,"Reach out with AWS Well-Architected offer"
example.com,76,HIGH_PRIORITY,sales@example.com,9,FinTech,120,"on-prem",Migration Project,scraped,high,"Alternative contact"
example.com,76,HIGH_PRIORITY,info@example.com,7,FinTech,120,"on-prem",Migration Project,generated,medium,"Backup contact"
another.com,58,MEDIUM_PRIORITY,contact@another.com,7,E-commerce,85,"AWS basic",Optimization,scraped,high,"Targeted outreach - serverless pitch"
```

### BATCH SUMMARY REPORT

```json
{
  "batch_id": "batch_001",
  "total_companies_input": 20,
  "pre_filter_passed": 8,
  "emails_found_total": 24,
  "high_priority_companies": 3,
  "medium_priority_companies": 5,
  "avoided_companies": 12,

  "top_targets": [
    {"company": "example.com", "score": 76, "emails": 3, "opportunity": "Migration"},
    {"company": "healthtech.com", "score": 72, "emails": 2, "opportunity": "Multi-cloud"},
    {"company": "finapp.com", "score": 68, "emails": 1, "opportunity": "Staff Aug"}
  ],

  "avoided_reasons": {
    "AWS partner": 2,
    "Too large (>5k employees)": 3,
    "Cloud-native (no need)": 4,
    "Competitor": 1,
    "No emails found": 2
  },

  "efficiency_metrics": {
    "time_invested_hours": 12,
    "companies_analyzed_deep": 8,
    "high_value_targets_found": 3,
    "time_per_target": "4 hours",
    "success_rate": "37.5%"
  }
}
```

---

## 🚀 IMPLEMENTATION PRIORITIES

### PRIORITY 1 - Quick Wins (1-2 days implementation)

**Must have:**
1. ✅ Pre-validation tool (Python script to check HTTP status)
2. ✅ Email scoring system (TIER 1-5 ranking)
3. ✅ Multi-page checking (/contact, /sales, /partners)
4. ✅ Basic company pre-filter (size, AWS partner, industry)

**Expected impact:** 50% better email quality, 40% time saved

### PRIORITY 2 - Enhanced Extraction (3-5 days)

**Should have:**
5. ✅ Sitemap crawling
6. ✅ Common pattern generation + MX verification
7. ✅ JavaScript/JSON email extraction
8. ✅ Company scoring (0-100 points)

**Expected impact:** 70% email discovery rate, full company context

### PRIORITY 3 - Intelligence Layer (1 week)

**Nice to have:**
9. ⚠️ Industry classification (FinTech, HealthTech detection)
10. ⚠️ Tech stack detection (BuiltWith/Wappalyzer integration)
11. ⚠️ Alternative sources (LinkedIn, Hunter.io APIs)
12. ⚠️ Email verification (SMTP check - careful with rate limits)

**Expected impact:** Personalized outreach, higher conversion

---

## 📈 SUCCESS METRICS

### BEFORE (Baseline - Current Approach)

```
Input: 20 companies (random list)
Time: 10 hours (30 min each full analysis)
Output:
- 1 email found (support@procore.com - low value)
- 0 HIGH-priority targets
- 2 MEDIUM-priority targets
Success rate: 10%
Time per valuable target: 5 hours
```

### AFTER (With Turbo Spec Implementation)

```
Input: 20 companies (job board sourced)
Pre-filter: 2 hours (6 min each)
  → 12 rejected (AWS partners, wrong size, competitors)
  → 8 passed to deep analysis

Deep analysis: 8 companies × 30 min = 4 hours
Output:
- 24 emails found (avg 3 per company)
- 3 HIGH-priority targets (score 70+)
- 5 MEDIUM-priority targets (score 40-69)
Success rate: 62.5%
Time per valuable target: 45 minutes

Efficiency gain: 6.7× faster per valuable target
Quality improvement: 6× more high-priority targets
```

---

## ⚠️ IMPORTANT CONSIDERATIONS

### Compliance & Ethics

1. **GDPR/Privacy:** Scraping publicly available business contacts is generally OK, but:
   - Don't scrape personal emails (firstname.lastname@)
   - Only target role-based emails (sales@, info@)
   - Honor robots.txt where possible
   - Respect rate limits (don't DDoS)

2. **Terms of Service:** Some sites prohibit scraping
   - Check robots.txt
   - Add delays between requests (2-5 seconds)
   - Use respectful crawling practices
   - Consider official APIs where available

3. **Email Verification:** SMTP verification can trigger spam filters
   - Use sparingly
   - Consider paid services (NeverBounce, ZeroBounce)
   - MX record check is safer alternative

### Rate Limiting & Performance

```python
# Respectful crawling
DELAY_BETWEEN_REQUESTS = 3  # seconds
MAX_CONCURRENT_REQUESTS = 5
TIMEOUT = 10  # seconds

# Retry logic
MAX_RETRIES = 3
BACKOFF_FACTOR = 2  # exponential backoff
```

### Error Handling

```python
# Always handle gracefully
try:
    result = await fetch_page(url)
except aiohttp.ClientError:
    log_error("Network error", url)
    return None
except asyncio.TimeoutError:
    log_error("Timeout", url)
    return None
except Exception as e:
    log_error(f"Unexpected: {e}", url)
    return None
```

---

## 🎯 FINAL CHECKLIST

When processing a batch, ensure:

- [ ] Batch sourced from quality source (job boards, Crunchbase, LinkedIn)
- [ ] Pre-validation completed (HTTP status checked)
- [ ] Pre-filter run (size, AWS partner, industry checks)
- [ ] Only PASS companies proceed to deep analysis
- [ ] Multiple pages checked per company (/contact, /sales, /partners, sitemap)
- [ ] Emails extracted from HTML, JavaScript, and common patterns
- [ ] MX records verified for generated patterns
- [ ] Emails scored and filtered (exclude HR/noreply)
- [ ] Company scored on 5 dimensions (0-100 scale)
- [ ] Context added (industry, size, tech stack, opportunities)
- [ ] Output generated in both CSV and JSON
- [ ] Batch summary created with metrics

**Result:** Prioritized list of high-value contacts at companies that are excellent fits for AWS cloud consulting.

---

## 📚 APPENDIX: KEY LEARNINGS FROM TESTING

**Tested companies:** procore.com, davey.com, customfit.me, bgsu.edu

**Key lessons:**
1. Financial stability ≠ good target (Procore: $1.2B but AWS partner = score 12/100)
2. Entity verification critical (CustomFit domain confusion)
3. Size extremes problematic (reject <10 or >5,000 employees)
4. Cloud-native = auto-reject (they're experts, not learners)
5. Job postings = strongest signal (hiring cloud engineers = need + budget)
6. Pre-filtering saves 60% time (caught Procore AWS partnership in 2 min vs 30 min full analysis)

**Patterns that predict HIGH scores:**
- ✅ 50-500 employees
- ✅ Series A-C or bootstrapped profitable
- ✅ Traditional industry (finance, healthcare, manufacturing)
- ✅ Job postings: AWS/cloud/migration
- ✅ Recent "digital transformation" news
- ✅ Glassdoor 3.5+
- ✅ Accessible contact (sales@, partners@)

**Auto-reject patterns:**
- ❌ AWS Advanced/Premier Partner
- ❌ Cloud-native SaaS
- ❌ >5,000 or <10 employees
- ❌ Consulting firms in tech
- ❌ Glassdoor <3.0 or payment issues

---

**END OF SPEC TURBO v1.0**
