# Salesforce → Power BI Integration
### Live Sales Analytics Dashboard with OAuth Connectivity, Semantic Modeling & Scheduled Refresh

---

## Problem Statement

The executive team relied on a siloed, manual data export routine to understand sales performance. Pipeline tracking lived in Salesforce, but the people who needed to act on it — finance, operations, leadership — worked in Excel and Power BI, because that's where cross-departmental planning already happened. Every week, someone exported a CSV by hand and re-uploaded it, which meant the numbers in the exec deck were only ever as fresh as the last person who remembered to refresh them.

The deeper cost wasn't just staleness — it was trust. Because sales, finance, and ops each maintained their own spreadsheet logic for a metric as basic as "win rate," leadership meetings routinely turned into debates over whose number was correct, rather than decisions based on the data itself. And because native Salesforce reports are rigid, slicing pipeline by a new dimension — region, rep, product — meant filing a request with IT and waiting, rather than just clicking a filter.

This gap isn't solvable inside Salesforce alone. Salesforce reporting is genuinely strong *for Salesforce data*, but it has no visibility outside itself — and real business questions (like pipeline-to-revenue conversion adjusted for marketing spend and headcount cost) span systems Salesforce was never built to unify. It's also a licensing problem: giving every finance or ops employee a paid Salesforce seat just to view a number is expensive when Power BI is something most employees already have through Microsoft 365.

This dynamic has intensified rather than faded. As organizations adopt AI copilots (Copilot in Power BI, Salesforce Agentforce), those tools depend on clean, governed semantic models to answer cross-functional questions — something impossible while data stays siloed in separate platforms. Rising scrutiny on SaaS spend is pushing the same direction: centralize reporting in tools already paid for, rather than issuing broad CRM licenses purely for visibility.

---

## What I Built

A live-connected Power BI dashboard, ingesting Salesforce objects into a governed semantic model via OAuth, designed to answer the three questions a sales leader actually asks:

1. How much revenue is in the pipeline right now, by stage?
2. What's our win rate, and is it improving?
3. Which reps or regions are converting best?

---

## Technical Approach

**1. Data Layer — Salesforce Developer Org**
Configured Opportunity, Account, and User objects with the fields required for pipeline analysis (Stage, Amount, Close Date, Owner), and added a custom Region picklist on Account to enable geographic slicing.

**2. Live Connection — OAuth Integration**
Ingested live Salesforce objects into a Power BI workspace via authenticated OAuth login, rather than a static export. This replaces the manual CSV export/import cycle with a persistent, refreshable connection, directly eliminating the weekly manual-export workflow.

**3. Semantic Modeling**
Built and verified relationships in Model view: Opportunity → Account (via AccountId) and Opportunity → User (via OwnerId), forming the semantic model that underlies every visual in the report. This relational layer is what makes slicing by `Account.Region` or `User.Name` possible — without it, the report is one flat, unfilterable table rather than a real analytical model.

**4. DAX Measures**
Defined business logic once, centrally, so every consumer of the report sees the same numbers calculated the same way — using Salesforce's native boolean fields rather than string comparisons, for cleaner and more efficient filter logic:

```DAX
Total Pipeline = SUM(Opportunity[Amount])

Win Rate =
DIVIDE(
    CALCULATE(COUNTROWS(Opportunity), Opportunity[IsWon] = TRUE()),
    CALCULATE(COUNTROWS(Opportunity), Opportunity[IsClosed] = TRUE())
)

Avg Deal Size = AVERAGE(Opportunity[Amount])

Closed Won Amount =
CALCULATE(SUM(Opportunity[Amount]), Opportunity[IsWon] = TRUE())
```

This eliminates the "everyone has their own version of win rate" problem — the measure is defined once, against the same underlying fields Salesforce itself uses to mark a deal closed and won, and reused everywhere downstream.

**5. Visualization**
- Funnel chart — pipeline Amount by Stage, to spot where deals stall
- KPI cards — Total Pipeline, Win Rate, Avg Deal Size
- Line chart — Closed Won revenue trended by month
- Table with conditional formatting — Opportunities by Owner, with a red-to-green gradient on Win Rate to surface rep performance at a glance

**6. Interactivity**
Added Region and Owner slicers so any user can filter the entire report live — turning a static snapshot into a self-service tool that removes the need to file a new report request for every new view of the data.

**7. Scheduled Refresh**
Because both Salesforce and the Power BI workspace are cloud-hosted, scheduled refresh (hourly/daily) requires no on-premises gateway. This is the direct fix for the "exec deck number is always stale" problem — the report stays current with zero manual intervention, typically the single highest-value line item of the entire project when explaining it to a non-technical stakeholder.

---

## Outcome

> Replaced a manual weekly CSV export process with a live, OAuth-connected Power BI semantic model that refreshes automatically, centralizes win-rate and pipeline logic into single reusable DAX measures built on Salesforce's native status fields, and lets any stakeholder self-serve filtered views by region or rep — without filing a new report request or requiring a Salesforce license.

---

## Skills Demonstrated

| Skill | Where It Maps on Job Postings |
|---|---|
| Salesforce object/field administration | "Salesforce CRM" |
| OAuth-based live connector setup | "System integration," "API/connector experience" |
| Power Query data cleaning | "Data Analysis" |
| Semantic / relational data modeling | Core BI skill |
| DAX measure development | "Microsoft Power BI" |
| Scheduled refresh / workspace administration | "BI operations," "reporting automation" |
| Dashboard/UX design | Direct portfolio deliverable |

---

## Roadmap (Not Yet Built — Next Iteration)

These are deliberately listed as planned work, not completed features, since I haven't built them yet:

- **Row-Level Security** — restrict each rep to seeing only their own deals within the report, mirroring Salesforce's own sharing rules.
- **Pipeline snapshot model** — a live connection only shows today's pipeline; adding a periodic snapshot table would enable true pipeline velocity tracking (how the pipeline changed month over month), which is what mature enterprise dashboards actually track instead of a single point-in-time view.
- **Multi-source blending** — join in a secondary dataset (e.g., marketing spend by region) to calculate ROI per region, mirroring what most real-world integration requests actually want: Salesforce data sitting next to data from another system.
- **Cases/Leads expansion** — extend the model to service and marketing data for visibility beyond sales alone.
- **Publish and share a live link** via the workspace instead of static screenshots, removing me from the reporting loop entirely.
