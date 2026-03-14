# Track 1: LinkedIn Practitioner Search — Summary

## Methodology

Used authenticated Playwright session with stored credentials at `/home/mepurgamentum/playwright-tools/linkedin-state.json`. Ran three searches:

1. `https://www.linkedin.com/search/results/people/?keywords=Head%20of%20Corporate%20Affairs`
2. `https://www.linkedin.com/search/results/people/?keywords=VP%20Corporate%20Affairs%20stakeholder`
3. `https://www.linkedin.com/search/results/people/?keywords=Chief%20Communications%20Officer%20stakeholder%20governance`

Session confirmed logged-in (no CAPTCHA or auth wall encountered). Results loaded dynamically and were extracted via `page.innerText('body')`.

---

## Results Found

### Search 1: "Head of Corporate Affairs"

| Name | Title | Organization | Location |
|---|---|---|---|
| Susie Tappouni | Head of Corporate Affairs | Amgen | Los Angeles, CA |
| Beth Richek | SVP, Head of Corporate Affairs Communications | Wells Fargo | Charlotte, NC |
| Katie Beirne Fallon | EVP and Head of Corporate Affairs | Fidelity Investments | Washington DC-Baltimore Area |

**Additional result noted:**
- One result referenced "United Kingdom National Nuclear Laboratory" (Head of Corporate Affairs) but name was not captured before session ended.

### Search 2: "VP Corporate Affairs stakeholder"

| Name | Title | Organization | Location |
|---|---|---|---|
| Nezihe Ceyla Arsel | VP, Corporate Communications, Event & Sponsorship, Sustainability | Akbank | Istanbul, Türkiye |

*Note: This search also showed a "Filter by seniority level" premium gate after the first few results, limiting page 2 scraping.*

### Search 3: "Chief Communications Officer stakeholder governance"

*Results were not fully captured before the session timeout. The search returned but pagination was limited. No additional named contacts captured.*

---

## Candidates Generated From This Track

- **Amgen** → Susie Tappouni (Head of Corporate Affairs) — promoted to HIGH priority
- **Wells Fargo** → Beth Richek (SVP, Head of Corporate Affairs Communications) — MEDIUM priority
- **Fidelity Investments** → Katie Beirne Fallon (EVP Corporate Affairs) — HIGH priority

---

## Methodology Notes

- LinkedIn anti-scraping measures limit the depth of results visible without Sales Navigator
- The first search (Head of Corporate Affairs) was most productive — generic enough to capture diverse results, specific enough to surface senior practitioners
- Page 2 and beyond require premium (Sales Navigator) access for reliable scraping
- The searches for specific platforms (Quorum, Borealis, RepTrak) in LinkedIn profiles would require Sales Navigator's skills/keyword filters
- 2nd-degree connection filter is visible and could be leveraged to prioritize warm outreach paths

---

## Gaps and Recommendations

- **Searches not completed:** "Head of Government & Corporate Affairs", "Director of Stakeholder Relations", "RepTrak corporate affairs" — recommend running in a follow-up session
- **Platform-specific searches** (people who list Quorum, Borealis, or Signal AI in their profiles/skills) would require Sales Navigator
- **Pagination:** Only first page captured for searches 1 and 2. LinkedIn's anti-automation measures prevent reliable 2-3 page scraping without additional delays and more human-like interaction patterns
- **International candidates:** The Akbank result (Istanbul) indicates valuable international corporate affairs practitioners are findable, but a dedicated international search was not run
