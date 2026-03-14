# Research Notes — SCI Design Partner Identification

**Date:** March 2026
**Researcher:** Aristotle (subagent session)
**Scope:** Three research tracks: LinkedIn practitioner search, Arthur Page Society member research, platform customer intelligence

---

## What Worked Well

### Web Research Tools
- **Jina Reader** (`r.jina.ai`) was the most reliable tool for extracting structured content from websites. Clean markdown output, handled dynamic JavaScript-rendered pages better than raw curl.
- **Tavily search** was effective for finding named contacts within platform case studies (e.g., finding "Dana Bolden, CCO, Colgate-Palmolive" from Signal AI content, "Jane Lawrie, KPMG" from Signal AI CEO interview).
- **Direct case study URLs** (Quorum, Borealis) yielded the best named contacts when combined with grep for role-indicating keywords (Director, Head, VP, Chief, Senior).

### Platform Case Study Intelligence (Track 3)
- **Quorum** was the most transparent with named contacts: both S&P Global (Darlene Rosenkoetter) and DTE Energy (Alissa Sevrioukova) were named and quoted extensively in public case study content.
- **Borealis** named two contacts: Trans Mountain (Alain Parisé, Land Director) and Agnico Eagle Mines (Brad Ryder found via LinkedIn search).
- **Signal AI** yielded the highest-value named contact (Jane Lawrie, KPMG — whose cross-domain remit maps directly to SCI) through video/podcast content, not just text.
- **RepTrak** case studies are almost entirely anonymized — only Lenovo was named, and the named contact (Charlotte West) was identified via a LinkedIn post by RepTrak, not the case study itself.

### Page Society Conference (Track 2)
- The 2025 Annual Conference agenda was fully retrievable and contained the most senior named practitioners in the research — CCO-level contacts at American Airlines, Southwest Airlines, Philip Morris International, and Mubadala are highly valuable.
- Philip Morris International (Travis Parman) stands out as the single most SCI-aligned organization: their "smoke-free company" transformation narrative requires four-domain stakeholder governance simultaneously.

### LinkedIn (Track 1)
- Authentication worked cleanly — no CAPTCHA or session expiry encountered.
- First-page results yielded Fidelity Investments (Katie Beirne Fallon), Amgen (Susie Tappouni), and Wells Fargo (Beth Richek) — all 2nd-degree connections with public profiles.
- The cross-domain profile of Amgen's Susie Tappouni (strategic communications + patient advocacy) was the most differentiated LinkedIn find.

---

## What Didn't Work / Limitations

### LinkedIn Depth
- LinkedIn's premium gate prevents reliable 2nd+ page scraping without Sales Navigator
- Platform-specific searches (people who list "Quorum" or "Borealis" in skills) require LinkedIn's advanced search filters (Sales Navigator)
- Only 3 of 6 planned searches were completed before session ended

### RepTrak Anonymization
- RepTrak's case studies are deliberately anonymized for most customers. The Global RepTrak 100 list is a useful proxy (these companies have invested in reputation measurement) but doesn't confirm active platform usage.
- Recommend: use RepTrak's annual Global RepTrak 100 rankings as a secondary target list — top-ranked companies in each sector are likely clients.

### Page Society Full Member List
- The `page.org/member-organizations/` page did not render the full organization list (JavaScript required). Only ~8 organizations visible. 
- The full list is estimated at 150+ organizations — this is a significant gap.
- Recommendation: Use Playwright to execute JavaScript and render the full list, or search for the list on third-party sites.

### Borealis Platform Rebranding
- Borealis has been acquired by and rebranded under Irth Solutions in some markets — some case studies are now at `irthsolutions.com` rather than `boreal-is.com`. The research covered both, but the full customer list may be distributed across both domains.

---

## Gaps to Fill in Follow-Up Research

1. **Full Page Society member list** — critical for cross-referencing 150+ organizations. Requires JavaScript execution in Playwright.

2. **Platform-specific LinkedIn searches** — people listing Quorum/Borealis/RepTrak/Signal AI in skills. Requires Sales Navigator.

3. **Named contacts at Uber, PanAust, Ecolab** — these are medium-priority candidates where the organization is identified but no named practitioner was found. LinkedIn search would close this gap.

4. **RepTrak Global 100 cross-reference** — check which Global RepTrak 100 companies also face multi-domain stakeholder challenges (all of them) and identify which are in SCI's sweet spot (mid-to-large, not Fortune 10).

5. **HPE (Hewlett Packard Enterprise)** — mentioned in PageViews Monthly as running a "Reputation Council" cross-functional group (comms, legal, HR, investor relations). This is a direct SCI signal. Named contact needed.

6. **Toyota Motor Corporation** — Julie Hamp (Senior Media Advisor) was a Page Society speaker. Her background includes being Toyota's first female board member and CCO. The current Toyota CCO/CCAO should be identified.

7. **Quorum "Wonk Week 2025" speakers** — this event had 12+ sessions with public affairs professionals. Named contacts likely available from session recordings.

8. **Signal AI "Exec Connect" breakfast participants** — Signal AI runs senior CCO roundtables ("Exec Connect"). One was described in a LinkedIn post; participants include "senior Communications and Corporate Affairs leaders." Identifying who attended could yield 5-10 additional candidates.

9. **Q4/Nasdaq IR clients** — the secondary design partner profile (IR clients navigating activist campaigns/ESG controversies) was not researched. Recommend searching for Q4 Inc. or Nasdaq IR case studies with named contacts.

10. **Edelman alumni** — the secondary profile (Edelman alumni now in-house) was not researched. LinkedIn search for "Edelman" in past experience + current "Head of Corporate Affairs" or "CCO" in-house role would be productive.

---

## Methodological Recommendations for Next Session

1. Run LinkedIn searches with Playwright using longer waits (5+ seconds) and more realistic scroll behavior to avoid anti-bot detection
2. Retrieve the full Page Society member list via Playwright JavaScript execution
3. Search Quorum's "Policy Wins" podcast for named corporate affairs practitioners (each episode features a practitioner — likely 50+ named contacts)
4. Search Signal AI's "Exec Connect" blog posts for attendee names
5. Cross-reference RepTrak's Global 100 list (by industry) against known multi-domain governance challenges to identify 10-15 additional target organizations
