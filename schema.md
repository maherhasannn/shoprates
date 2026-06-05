---
name: schema.md
description: Generate a complete enterprise-level JSON-LD schema @graph for Credible Law pages (bankruptcy, business debt, MCA defense, asset protection, Chapter 11, Subchapter V, UCC liens, bank levies, judgment defense, creditor lawsuits, commercial litigation). The user provides a page slug plus the full page content; the skill outputs only the final JSON-LD script tag — Google-indexable, ready to paste into WordPress. Use whenever the user pastes Credible Law page content and asks for schema, JSON-LD, structured data, or anything along those lines.
---

# Credible Law Schema Generator

This skill produces one thing: a complete, valid, Google-indexable `<script type="application/ld+json">` block for a Credible Law page. Nothing else. No preamble, no "here you go", no closing notes. Just the script tag.

## Input contract

The user will always provide:
1. A slug or full URL (e.g., `mca-bankruptcy-options` or `https://crediblelaw.com/mca-bankruptcy-options/`)
2. The full page content (HTML or pasted text — title, meta description, H1, body copy, FAQ section, internal links)

If either is missing, ask once. Do not fetch the page. Do not invent content.

## Output contract

Return only:

```
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@graph": [ ... ]
}
</script>
```

No text before. No text after. No code-fence labels like ```json. Just the script tag and its valid JSON-LD contents.

---

## Build process

Work through these steps in order.

### Step 1: Extract page facts

From the pasted page content, pull:
- **Full page URL**: `https://crediblelaw.com/<slug>/` (always trailing slash)
- **SEO title**: from the `<title>` tag or page title
- **Meta description**: from the meta description tag
- **H1**: the main article headline
- **Word count**: count words in the article body (exclude nav/footer; include FAQ text if part of article flow)
- **Primary keyword**: main topic phrase from H1/title
- **Secondary keyword**: closely related phrase from subheads or repeated body language
- **FAQs**: pull every Q&A pair verbatim. Do not paraphrase. Do not invent. If no FAQ section exists, stop and tell the user.
- **Internal/external links**: collect every URL for use in Step 3.

### Step 2: Classify the page topic

Assign the page to one bucket. The bucket drives breadcrumb parent, Service entity, and DefinedTerms/Things.

- **Bankruptcy** — Chapter 11, Chapter 7, Subchapter V, reorganization, emergency filing, debtor-in-possession
  - Breadcrumb parent: `Bankruptcy and Debt Solutions` → `https://crediblelaw.com/bankruptcy-and-debt-solutions/`
- **MCA Defense** — merchant cash advance, MCA lawsuit, MCA collections, MCA settlement, MCA bank levy, stop ACH, confession of judgment, MCA default judgment, MCA UCC lien
  - Breadcrumb parent: `Merchant Cash Advance Defense` → `https://crediblelaw.com/mca-defense-attorney/`
- **Creditor / Commercial Litigation** — creditor lawsuit, judgment defense, breach of contract, partnership disputes, shareholder disputes, commercial fraud
  - Breadcrumb parent: `Business Litigation` → `https://crediblelaw.com/business-litigation/`
- **Asset Protection / Bank Levy / UCC** — business bank levy, frozen account, UCC lien removal, asset protection, receivables lien
  - Breadcrumb parent: UCC/bank levy tied to MCA → MCA Defense; tied to general creditors → Business Litigation; pure asset protection → `Bankruptcy and Debt Solutions`

If the page is genuinely mixed, pick the bucket that best matches the H1's primary intent.

### Step 3: Verify every link before using it

Only use links from the **Verified Valid Links** section at the bottom of this skill. If a link in the page content isn't on that list and you can't match it to a confirmed page, leave it out. Never link to a page that may not exist.

For DefinedTerm `url` fields: only attach a URL if the term maps cleanly to a confirmed page. When in doubt, omit the URL field.

For external `sameAs` on Thing entities: use only the authoritative external sources listed in Verified Valid Links.

### Step 4: Assemble the @graph

Build every required entity — all six site-wide entities plus the page-specific ones for the assigned bucket. Follow the schema specs in the **Schema Spec** section exactly.

Key rules:
- **NAP must be exact**: Credible Law / 160 Thorn St, San Diego, CA 92103 / +1-888-201-0441
- **Stable @id values**: use exactly the IDs documented below
- **Dates**: use today's date for both `datePublished` and `dateModified` in YYYY-MM-DD format
- **FAQ entities**: one `Question` per Q&A, verbatim from the page
- **Word count**: include as a string in the Article entity
- **Keywords array**: lead with primary keyword, then secondary, then the standard topical keywords
- **DefinedTerms and Things**: include only the ones for the page's bucket (see bucket map in Schema Spec)

### Step 5: Compliance check

Before outputting, verify:
- [ ] All URLs are absolute (start with `https://`)
- [ ] All `@id` values are stable and follow the documented pattern
- [ ] No fake reviews, no `aggregateRating`, no fake attorney names
- [ ] No "best", "top rated", "guaranteed", or outcome promises
- [ ] No language suggesting the page creates an attorney-client relationship
- [ ] Every link points to a confirmed-real page
- [ ] FAQ answers are legally cautious — describe how things may work, not guarantees
- [ ] JSON is valid (no trailing commas, all strings quoted, all brackets matched)
- [ ] Output is ONLY the `<script>` block — no preamble or commentary

### Step 6: Return the script block

Output the `<script type="application/ld+json">` block. Nothing else.

---

## What this skill never does

- Never fetches pages from the web
- Never invents FAQs
- Never links to unverified pages
- Never adds preamble, explanations, or closing remarks
- Never includes `aggregateRating`, fake reviews, fake attorney names, or guaranteed-outcome language
- Never uses superlatives like "best" or "top rated"

---

## Schema Spec

### NAP and stable IDs

**Name / Address / Phone — exact on every page:**
- Credible Law / 160 Thorn St, San Diego, CA 92103 / +1-888-201-0441

**Site-wide stable @id values:**
- `https://crediblelaw.com/#organization`
- `https://crediblelaw.com/#legalservice`
- `https://crediblelaw.com/#localbusiness`
- `https://crediblelaw.com/#website`
- `https://crediblelaw.com/#place`
- `https://crediblelaw.com/#contact`

**Page-specific @id pattern** (`[URL]` = full page URL with trailing slash):
- `[URL]#webpage`, `[URL]#article`, `[URL]#faq`, `[URL]#service`, `[URL]#breadcrumb`, `[URL]#terms`, `[URL]#primary-topic`, `[URL]#<term-slug>`

---

### Site-wide entities (include all six on every page)

**Organization**
```json
{
  "@type": "Organization",
  "@id": "https://crediblelaw.com/#organization",
  "name": "Credible Law",
  "url": "https://crediblelaw.com/",
  "telephone": "+1-888-201-0441",
  "logo": { "@type": "ImageObject", "url": "https://crediblelaw.com/wp-content/uploads/credible-law-logo.png" },
  "address": { "@type": "PostalAddress", "streetAddress": "160 Thorn St", "addressLocality": "San Diego", "addressRegion": "CA", "postalCode": "92103", "addressCountry": "US" },
  "contactPoint": { "@id": "https://crediblelaw.com/#contact" },
  "sameAs": [
    "https://www.youtube.com/@GetCredibleLaw",
    "https://www.instagram.com/crediblelaw",
    "https://www.linkedin.com/company/credible-law",
    "https://www.facebook.com/profile.php?id=61575616241309",
    "https://medium.com/@crediblelawsite"
  ]
}
```

**ContactPoint**
```json
{
  "@type": "ContactPoint",
  "@id": "https://crediblelaw.com/#contact",
  "telephone": "+1-888-201-0441",
  "contactType": "customer service",
  "areaServed": "US",
  "availableLanguage": "English"
}
```

**Place**
```json
{
  "@type": "Place",
  "@id": "https://crediblelaw.com/#place",
  "name": "Credible Law San Diego Office",
  "address": { "@type": "PostalAddress", "streetAddress": "160 Thorn St", "addressLocality": "San Diego", "addressRegion": "CA", "postalCode": "92103", "addressCountry": "US" }
}
```

**LegalService**
```json
{
  "@type": "LegalService",
  "@id": "https://crediblelaw.com/#legalservice",
  "name": "Credible Law Bankruptcy, Business Debt, MCA Defense, and Asset Protection Legal Help",
  "url": "https://crediblelaw.com/",
  "telephone": "+1-888-201-0441",
  "priceRange": "Consultation-based",
  "provider": { "@id": "https://crediblelaw.com/#organization" },
  "address": { "@type": "PostalAddress", "streetAddress": "160 Thorn St", "addressLocality": "San Diego", "addressRegion": "CA", "postalCode": "92103", "addressCountry": "US" },
  "areaServed": [
    { "@type": "City", "name": "San Diego" },
    { "@type": "State", "name": "California" },
    { "@type": "Country", "name": "United States" }
  ],
  "serviceType": [
    "Business bankruptcy","Chapter 11 bankruptcy","Subchapter V bankruptcy","Business debt restructuring",
    "Business asset protection","Creditor lawsuit defense","Judgment defense","Default judgment defense",
    "Business bank levy defense","Frozen business bank account legal help","UCC lien defense","UCC lien removal",
    "Merchant cash advance defense","MCA lawsuit defense","MCA bankruptcy options",
    "Commercial litigation defense","Emergency business debt legal help"
  ],
  "knowsAbout": [
    "Chapter 11 bankruptcy","Subchapter V bankruptcy","Chapter 7 bankruptcy","Automatic stay",
    "Business debt restructuring","Business asset protection","Judgment enforcement","Bank levies",
    "Frozen business bank accounts","UCC liens","UCC-1 financing statements","Merchant cash advances",
    "MCA lawsuits","MCA collections","Default judgments","Confession of judgment",
    "Accounts receivable liens","Business litigation","Commercial litigation","Business debt settlement",
    "Asset liquidation","Reorganization plans","Debtor-in-possession operations"
  ]
}
```

**LocalBusiness**
```json
{
  "@type": "LocalBusiness",
  "@id": "https://crediblelaw.com/#localbusiness",
  "name": "Credible Law",
  "url": "https://crediblelaw.com/",
  "telephone": "+1-888-201-0441",
  "priceRange": "Consultation-based",
  "image": "https://crediblelaw.com/wp-content/uploads/credible-law-logo.png",
  "address": { "@type": "PostalAddress", "streetAddress": "160 Thorn St", "addressLocality": "San Diego", "addressRegion": "CA", "postalCode": "92103", "addressCountry": "US" },
  "parentOrganization": { "@id": "https://crediblelaw.com/#organization" },
  "areaServed": ["San Diego", "California", "United States"]
}
```

**WebSite**
```json
{
  "@type": "WebSite",
  "@id": "https://crediblelaw.com/#website",
  "name": "Credible Law",
  "url": "https://crediblelaw.com/",
  "publisher": { "@id": "https://crediblelaw.com/#organization" },
  "potentialAction": {
    "@type": "SearchAction",
    "target": "https://crediblelaw.com/?s={search_term_string}",
    "query-input": "required name=search_term_string"
  }
}
```

---

### Page-specific entities

**WebPage**
```json
{
  "@type": "WebPage",
  "@id": "[URL]#webpage",
  "url": "[URL]",
  "name": "[SEO TITLE]",
  "description": "[META DESCRIPTION]",
  "isPartOf": { "@id": "https://crediblelaw.com/#website" },
  "publisher": { "@id": "https://crediblelaw.com/#organization" },
  "provider": { "@id": "https://crediblelaw.com/#legalservice" },
  "breadcrumb": { "@id": "[URL]#breadcrumb" },
  "mainEntity": [ { "@id": "[URL]#article" }, { "@id": "[URL]#faq" }, { "@id": "[URL]#service" } ],
  "about": [ { "@id": "[URL]#primary-topic" } ],
  "mentions": [ /* refs to relevant Thing @ids from bucket map */ ],
  "significantLink": [ /* verified internal links relevant to this page */ ],
  "inLanguage": "en-US"
}
```

**Article**
```json
{
  "@type": "Article",
  "@id": "[URL]#article",
  "headline": "[H1]",
  "description": "[META DESCRIPTION]",
  "author": { "@id": "https://crediblelaw.com/#organization" },
  "publisher": { "@id": "https://crediblelaw.com/#organization" },
  "mainEntityOfPage": { "@id": "[URL]#webpage" },
  "datePublished": "[TODAY YYYY-MM-DD]",
  "dateModified": "[TODAY YYYY-MM-DD]",
  "articleSection": ["Bankruptcy Law","Business Bankruptcy","Business Debt","Business Asset Protection","Commercial Litigation","Merchant Cash Advance Defense","Creditor Defense"],
  "keywords": [
    "[PRIMARY KEYWORD]","[SECONDARY KEYWORD]",
    "business bankruptcy","Chapter 11 bankruptcy","Subchapter V bankruptcy","business asset protection",
    "merchant cash advance defense","business debt restructuring","creditor lawsuit defense",
    "UCC lien defense","business bank levy defense"
  ],
  "about": [ { "@id": "[URL]#primary-topic" } ],
  "mentions": [ /* same Thing refs as WebPage */ ],
  "wordCount": "[INTEGER AS STRING]",
  "inLanguage": "en-US"
}
```

**FAQPage** — one `Question` per Q&A verbatim from the page; if no FAQ section exists, stop and tell the user
```json
{
  "@type": "FAQPage",
  "@id": "[URL]#faq",
  "mainEntity": [
    { "@type": "Question", "name": "[QUESTION VERBATIM]", "acceptedAnswer": { "@type": "Answer", "text": "[ANSWER VERBATIM]" } }
  ]
}
```

**DefinedTermSet**
```json
{
  "@type": "DefinedTermSet",
  "@id": "[URL]#terms",
  "name": "Bankruptcy, Business Debt, MCA Defense, and Asset Protection Legal Terms",
  "hasDefinedTerm": [ /* refs to DefinedTerm @ids for this page */ ]
}
```

---

### Service entity

Pick a name from the list below that matches the bucket. Customize `serviceType[0]` for the specific page subject.

```json
{
  "@type": "Service",
  "@id": "[URL]#service",
  "name": "[See name suggestions by bucket below]",
  "serviceType": ["[Primary service for this page]","Business debt legal help","Commercial litigation strategy","Creditor defense","Asset protection risk review"],
  "provider": { "@id": "https://crediblelaw.com/#legalservice" },
  "areaServed": [ { "@type": "City", "name": "San Diego" }, { "@type": "State", "name": "California" }, { "@type": "Country", "name": "United States" } ],
  "audience": { "@type": "BusinessAudience", "audienceType": "Business owners, executives, entrepreneurs, and companies facing business debt, creditor lawsuits, MCA collections, bankruptcy risk, or asset protection concerns" },
  "offers": {
    "@type": "Offer",
    "name": "[Page-specific offer name]",
    "price": "0",
    "priceCurrency": "USD",
    "description": "Consultation-oriented legal review for business owners seeking information about bankruptcy, creditor defense, business debt, MCA-related legal issues, or asset protection options. No outcome is guaranteed."
  },
  "termsOfService": "https://crediblelaw.com/terms-and-conditions/",
  "serviceOutput": ["bankruptcy options review","business debt restructuring assessment","creditor lawsuit defense evaluation","asset protection risk review","automatic stay strategy review","judgment enforcement risk assessment","bank levy response strategy","UCC lien review","MCA debt and lawsuit defense options","business survival legal strategy"]
}
```

**Name suggestions by bucket:**
- Bankruptcy: "Chapter 11 Business Bankruptcy and Restructuring Help" / "Subchapter V Bankruptcy Help for Small Businesses"
- MCA Defense: "Merchant Cash Advance Bankruptcy and Business Debt Defense Help" / "MCA Lawsuit Defense and Settlement Help" / "MCA Bank Levy and ACH Withdrawal Defense Help" / "MCA UCC Lien Removal and Receivables Defense Help"
- Creditor / Commercial Litigation: "Business Litigation and Creditor Defense Help" / "Commercial Fraud Lawsuit Defense Help" / "Breach of Contract Lawsuit Defense Help" / "Partnership and Shareholder Dispute Help"
- Asset Protection / Bank Levy / UCC: "Business Asset Protection and Creditor Defense Help" / "Business Bank Levy Defense and Emergency Creditor Collection Help" / "UCC Lien Defense and Removal Help"

---

### Breadcrumb variants

```json
{
  "@type": "BreadcrumbList",
  "@id": "[URL]#breadcrumb",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://crediblelaw.com/" },
    { "@type": "ListItem", "position": 2, "name": "[PARENT NAME]", "item": "[PARENT URL]" },
    { "@type": "ListItem", "position": 3, "name": "[PAGE TITLE]", "item": "[URL]" }
  ]
}
```

| Bucket | Parent name | Parent URL |
|---|---|---|
| Bankruptcy | Bankruptcy and Debt Solutions | `https://crediblelaw.com/bankruptcy-and-debt-solutions/` |
| MCA Defense | Merchant Cash Advance Defense | `https://crediblelaw.com/mca-defense-attorney/` |
| Creditor / Commercial Litigation | Business Litigation | `https://crediblelaw.com/business-litigation/` |
| Asset Protection / Bank Levy / UCC | Pick closest fit per Step 2 | — |

---

### DefinedTerm catalog

Shape for each term:
```json
{ "@type": "DefinedTerm", "@id": "[URL]#<term-slug>", "name": "[Term name]", "description": "[Legally cautious description]", "inDefinedTermSet": { "@id": "[URL]#terms" }, "url": "[optional — confirmed Credible Law page only]" }
```

| Slug | Name | Description | URL |
|---|---|---|---|
| `business-bankruptcy` | Business bankruptcy | A legal process that may help businesses address overwhelming debt through reorganization or liquidation, subject to court approval and applicable legal requirements. | `https://crediblelaw.com/business-bankruptcy/` |
| `chapter-11-bankruptcy` | Chapter 11 bankruptcy | A bankruptcy process that may allow a business to reorganize debts while continuing operations, subject to court approval and applicable legal requirements. | `https://crediblelaw.com/business-bankruptcy/` |
| `subchapter-v-bankruptcy` | Subchapter V bankruptcy | A streamlined form of Chapter 11 designed for eligible small businesses, which may offer a faster and lower-cost reorganization path. | `https://crediblelaw.com/subchapter-v-bankruptcy-san-diego/` |
| `chapter-7-bankruptcy` | Chapter 7 bankruptcy | A bankruptcy process generally involving liquidation of non-exempt assets to pay creditors, subject to court approval and applicable legal requirements. | — |
| `automatic-stay` | Automatic stay | A court-ordered pause on most collection activities that may take effect when a bankruptcy case is filed, subject to exceptions under the Bankruptcy Code. | — |
| `debtor-in-possession` | Debtor in possession | A business that continues operating its assets and affairs during a Chapter 11 case, subject to court oversight and applicable legal requirements. | — |
| `reorganization-plan` | Reorganization plan | A proposed plan in a Chapter 11 or Subchapter V case that outlines how a business intends to address its debts, subject to creditor and court approval. | — |
| `business-debt-restructuring` | Business debt restructuring | A process that may involve renegotiating the terms of business debts to make them more manageable, which can occur inside or outside of bankruptcy. | — |
| `creditor-lawsuit` | Creditor lawsuit | A civil action filed by a creditor seeking payment, judgment, or other relief related to an alleged business debt. | — |
| `judgment-lien` | Judgment lien | A lien that may attach to property after a court enters a money judgment against a debtor, subject to applicable state and federal law. | — |
| `default-judgment` | Default judgment | A judgment that a court may enter against a defendant who fails to respond to a lawsuit within the required time, subject to applicable procedural rules. | — |
| `bank-levy` | Bank levy | A legal collection tool that may allow a creditor with a judgment to seize funds from a debtor's bank account, subject to applicable law and exemptions. | `https://crediblelaw.com/business-bank-levy-defense/` |
| `frozen-business-bank-account` | Frozen business bank account | A bank account on which transactions are restricted, often as a result of a levy, lien, or other legal process by a creditor. | `https://crediblelaw.com/business-bank-levy-defense/` |
| `ucc-lien` | UCC lien | A security interest in personal property of a business filed under the Uniform Commercial Code, often used by lenders and merchant cash advance companies. | `https://crediblelaw.com/mca-ucc-lien-removal/` |
| `ucc-1-financing-statement` | UCC-1 financing statement | A public filing that gives notice of a creditor's security interest in a debtor's personal property under the Uniform Commercial Code. | — |
| `merchant-cash-advance` | Merchant cash advance | A financing arrangement in which a business receives funds in exchange for a portion of future receivables, often structured outside traditional lending laws. | `https://crediblelaw.com/mca-defense-attorney/` |
| `mca-lawsuit` | MCA lawsuit | A civil action brought by a merchant cash advance company seeking payment, judgment, or enforcement against a business or its principals. | `https://crediblelaw.com/merchant-cash-advance-lawsuit-defense/` |
| `mca-collections` | MCA collections | Collection activities by a merchant cash advance company, which may include ACH withdrawals, lawsuits, UCC liens, bank levies, and judgment enforcement. | — |
| `confession-of-judgment` | Confession of judgment | A clause or document in which a party agrees in advance to entry of a judgment against them, the enforceability of which varies by state. | — |
| `business-asset-protection` | Business asset protection | Legal strategies that may help shield business assets from creditors, subject to applicable law and the specific facts of each case. | — |
| `accounts-receivable` | Accounts receivable | Amounts owed to a business by its customers for goods or services delivered, which may be subject to liens or other creditor claims. | `https://crediblelaw.com/ucc-lien-on-receivables/` |
| `commercial-litigation` | Commercial litigation | Civil litigation involving business disputes, contracts, partnerships, fraud claims, and related commercial matters. | `https://crediblelaw.com/business-litigation/` |
| `business-debt-settlement` | Business debt settlement | A negotiated resolution between a business and one or more creditors, which may reduce the amount owed or restructure payment terms. | — |

---

### Thing entity catalog

Shape for each thing:
```json
{ "@type": "Thing", "@id": "[URL]#<term-slug>", "name": "[Concept name]", "description": "[Short description]", "sameAs": [ /* authoritative sources only */ ] }
```

Always include `#primary-topic`:
```json
{ "@type": "Thing", "@id": "[URL]#primary-topic", "name": "[PRIMARY TOPIC — usually the H1 subject]", "description": "[1–2 sentence description of the main page topic]" }
```

| Slug | Name | Suggested sameAs |
|---|---|---|
| `chapter-11-bankruptcy` | Chapter 11 bankruptcy | `https://www.uscourts.gov/services-forms/bankruptcy`, `https://www.law.cornell.edu/uscode/text/11` |
| `subchapter-v-bankruptcy` | Subchapter V bankruptcy | `https://www.uscourts.gov/services-forms/bankruptcy`, `https://www.sba.gov/` |
| `automatic-stay` | Automatic stay | `https://www.law.cornell.edu/uscode/text/11` |
| `business-asset-protection` | Business asset protection | — |
| `business-debt-restructuring` | Business debt restructuring | — |
| `creditor-lawsuit` | Creditor lawsuit | — |
| `judgment-enforcement` | Judgment enforcement | — |
| `bank-levy` | Bank levy | — |
| `frozen-business-account` | Frozen business account | — |
| `ucc-lien` | UCC lien | `https://www.law.cornell.edu/ucc` |
| `merchant-cash-advance` | Merchant cash advance | `https://www.ftc.gov/business-guidance` |
| `mca-lawsuit` | MCA lawsuit | — |
| `mca-debt` | MCA debt | — |
| `commercial-litigation` | Commercial litigation | — |
| `business-debt-settlement` | Business debt settlement | — |

---

### Topic-bucket → entity-inclusion map

Use only the entities for the page's bucket.

**Bankruptcy bucket**
- DefinedTerms: business-bankruptcy, chapter-11-bankruptcy, subchapter-v-bankruptcy, chapter-7-bankruptcy, automatic-stay, debtor-in-possession, reorganization-plan, business-debt-restructuring, business-asset-protection, business-debt-settlement
- Things: primary-topic, chapter-11-bankruptcy, subchapter-v-bankruptcy, automatic-stay, business-asset-protection, business-debt-restructuring

**MCA Defense bucket**
- DefinedTerms: merchant-cash-advance, mca-lawsuit, mca-collections, ucc-lien, ucc-1-financing-statement, confession-of-judgment, default-judgment, bank-levy, frozen-business-bank-account, accounts-receivable, business-debt-settlement, business-asset-protection, + automatic-stay/chapter-11/subchapter-v if discussed
- Things: primary-topic, merchant-cash-advance, mca-lawsuit, mca-debt, ucc-lien, bank-levy, frozen-business-account, business-asset-protection

**Creditor / Commercial Litigation bucket**
- DefinedTerms: creditor-lawsuit, judgment-lien, default-judgment, bank-levy, business-debt-restructuring, business-debt-settlement, business-asset-protection, commercial-litigation
- Things: primary-topic, creditor-lawsuit, judgment-enforcement, bank-levy, commercial-litigation, business-asset-protection

**Asset Protection / Bank Levy / UCC bucket**
- DefinedTerms: business-asset-protection, bank-levy, frozen-business-bank-account, ucc-lien, ucc-1-financing-statement, judgment-lien, creditor-lawsuit, accounts-receivable, business-debt-restructuring
- Things: primary-topic, business-asset-protection, bank-levy, frozen-business-account, ucc-lien, creditor-lawsuit

---

## Verified Valid Links

Only use links from this list. Never invent or guess URLs.

### Credible Law internal pages

**Bankruptcy and debt**
- `https://crediblelaw.com/`
- `https://crediblelaw.com/business-bankruptcy/`
- `https://crediblelaw.com/bankruptcy-and-debt-solutions/`
- `https://crediblelaw.com/business-bankruptcy-lawyer-san-diego/`
- `https://crediblelaw.com/emergency-chapter-11-filing-san-diego/`
- `https://crediblelaw.com/subchapter-v-bankruptcy-san-diego/`

**MCA and collections**
- `https://crediblelaw.com/mca-bankruptcy-options/`
- `https://crediblelaw.com/mca-defense-attorney/`
- `https://crediblelaw.com/merchant-cash-advance-lawsuit-defense/`
- `https://crediblelaw.com/mca-default-judgment-defense/`
- `https://crediblelaw.com/vacate-mca-default-judgment/`
- `https://crediblelaw.com/merchant-cash-advance-settlement/`
- `https://crediblelaw.com/merchant-cash-advance-debt-relief/`
- `https://crediblelaw.com/mca-froze-my-bank-account/`
- `https://crediblelaw.com/stop-mca-ach-withdrawals-immediately/`

**UCC and asset protection**
- `https://crediblelaw.com/mca-ucc-lien-removal/`
- `https://crediblelaw.com/ucc-lien-on-business-assets/`
- `https://crediblelaw.com/ucc-lien-on-receivables/`
- `https://crediblelaw.com/challenge-ucc-lien-legally/`
- `https://crediblelaw.com/merchant-cash-advance-bank-levy/`
- `https://crediblelaw.com/business-bank-levy-defense/`

**Business litigation**
- `https://crediblelaw.com/business-litigation/`
- `https://crediblelaw.com/commercial-fraud-lawsuits/`
- `https://crediblelaw.com/breach-of-contract-lawsuits/`
- `https://crediblelaw.com/partnership-disputes/`
- `https://crediblelaw.com/shareholder-disputes/`

**Compliance**
- `https://crediblelaw.com/terms-and-conditions/`
- `https://crediblelaw.com/privacy-policy/`
- `https://crediblelaw.com/disclaimer/`
- `https://crediblelaw.com/attorney-advertising/`
- `https://crediblelaw.com/referral-disclosure/`

### Social profiles (Organization sameAs only)
- `https://www.youtube.com/@GetCredibleLaw`
- `https://www.instagram.com/crediblelaw`
- `https://www.linkedin.com/company/credible-law`
- `https://www.facebook.com/profile.php?id=61575616241309`
- `https://medium.com/@crediblelawsite`

### Authoritative external sources (Thing sameAs only)
- `https://www.uscourts.gov/services-forms/bankruptcy` — Chapter 11, Subchapter V, Chapter 7, automatic stay
- `https://www.sba.gov/` — Subchapter V / small business context
- `https://www.ftc.gov/business-guidance` — merchant cash advance guidance
- `https://www.law.cornell.edu/ucc` — UCC liens, UCC-1 financing statements
- `https://www.law.cornell.edu/uscode/text/11` — Bankruptcy Code (Title 11)

### Logo
- `https://crediblelaw.com/wp-content/uploads/credible-law-logo.png`

### Link usage rules
- If the user's pasted content includes a link not on this list, leave it out unless it's an obvious match for a confirmed page (e.g., same URL with/without trailing slash).
- Never link to `https://crediblelaw.com/blog/...` paths, category archives, or tag pages.
- Compliance links belong in WebPage `significantLink` when relevant, not in every entity.
- When `sameAs` is optional and you have no confirmed authoritative source, omit the field.
