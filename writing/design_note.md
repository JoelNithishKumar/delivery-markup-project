# The Geography of the Delivery Markup
### A research design note (in progress)

*Status: design and data infrastructure complete; price collection in progress. Full results to follow.*

---

## Research question

Within a single U.S. metro area, does the markup that food-delivery platforms (DoorDash, Uber Eats) charge over in-store menu price vary systematically with neighborhood income — and does the direction of that relationship look more consistent with **willingness-to-pay price discrimination** or with **platforms exploiting limited consumer alternatives** in underserved areas?

**Sub-questions:**
1. Is there a detectable income gradient in markup percentage across neighborhoods, holding chain fixed?
2. Does that gradient survive controlling for local market structure (number of competing delivery options) and neighborhood density?
3. Which of the two theories below does the observed pattern best fit?

## Why this question matters

This sits at the intersection of two active research areas in applied microeconomics:

**Cost-of-living measurement.** Standard cost-of-living indices largely assume a good costs the same wherever it's purchased, after adjusting for local price levels. Work on spatial retail access (e.g., Handbury and coauthors on retail prices and cost-of-living variation) has shown that *which retailers people can actually access* drives meaningful cost-of-living differences across neighborhoods that price indices miss. If delivery platforms add a geographically *differential* markup on top of this existing gap, official cost-of-living statistics may be missing a real and growing wedge — and the share of consumption flowing through delivery platforms is large and rising.

**Platform pricing and market power.** Two competing, falsifiable theories of platform conduct make opposite predictions:

- **Willingness-to-pay discrimination:** if higher-income neighborhoods have more inelastic demand for delivery (time is more valuable, fewer easy substitutes for ordering in), a profit-maximizing platform should set *higher* markups where income is higher. This is third-degree price discrimination, applied to a digital intermediary.
- **Access exploitation:** if lower-income neighborhoods have fewer competing delivery options, less car access, or fewer easy substitutes for picking food up themselves, the platform can extract *more* markup precisely where consumers have the fewest alternatives — predicting markup *falls* with income.

The sign of the income gradient, once local competition is controlled for, lets the data discriminate between these two stories rather than just describing a correlation.

## Data and design

**Geography.** Chicago, IL. Twenty ZIP codes (ZCTAs) selected from the full set of ~58 Chicago ZCTAs, stratified into income terciles (7 low, 7 mid, 6 high) using 2018–2022 ACS 5-year estimates, after excluding low-population downtown/commercial ZCTAs (population under ~5,000) that are not representative residential neighborhoods.

**Neighborhood covariates — American Community Survey (ACS), via `tidycensus`.** Median household income (`B19013_001`) and population (`B01003_001`) at the ZCTA level. Population density computed by joining to the Census Gazetteer ZCTA file (land area in square miles). This panel is already built and saved (`data/intermediate/chicago_zip_panel.csv`).

**Price data (primary, hand-collected).** Five national quick-service chains chosen for menu standardization across locations: McDonald's, Chipotle, Domino's, Panda Express, Subway. One anchor item per chain (e.g., Big Mac Extra Value Meal; Chicken Burrito Bowl) collected across all 20 ZIPs, with two additional items per chain planned as a robustness check if time allows. For each ZIP × chain × item: in-store menu price (from the chain's own store-specific ordering page) and app price (DoorDash, Uber Eats), recorded before delivery/service fees — isolating the markup on the item itself rather than the all-in cost of delivery.

**Market structure proxy.** Number of fast-food/quick-service delivery options visible on DoorDash's restaurant feed for each ZIP, used as a control for local competitive intensity. Measured once per ZIP via a single platform for consistency, since absolute counts are not comparable across platforms with differing merchant coverage.

## Empirical approach (planned)

```r
library(fixest)

m1 <- feols(markup_pct ~ log(median_income) | chain, data = prices, cluster = "zip")
m2 <- feols(markup_pct ~ log(median_income) + density + n_competitors_visible | chain,
            data = prices, cluster = "zip")
```

Chain fixed effects absorb baseline markup differences across chains; standard errors clustered at the ZIP level. The coefficient on `log(median_income)` in the model with competition controls is the key object: a positive, robust coefficient favors willingness-to-pay discrimination; a negative, robust coefficient — especially one that strengthens once `n_competitors_visible` is added — favors access exploitation.

## What's built so far

- ACS income/population pull and Chicago ZCTA ranking script (`code/01_pull_acs.R`)
- Census Gazetteer join and density calculation (`code/02_join_gazetteer.R`)
- Final 20-ZIP stratified sample with neighborhood names and income terciles (`data/raw/chicago_zip_sample.csv`)
- Price-collection instrument, pre-populated with all ZIP × chain combinations (`data/raw/price_collection.csv`)

## What's in progress

Manual price collection across the 100 ZIP × chain anchor-item observations. Full regression results, figures, and the completed write-up will be posted to this repository as data collection finishes.

## Honest limitations (anticipated)

- Single metro area; results describe Chicago, not a national pattern.
- Manual price collection introduces some measurement noise relative to a scraped or platform-provided dataset; collection protocol is fixed in advance (item-only price, pre-fee) to minimize inconsistency.
- Cross-sectional design is descriptive, not causal — useful for characterizing the pattern, not for identifying platform intent without further data (e.g., a platform fee-policy change as a natural experiment).

---

*Repository: [https://github.com/JoelNithishKumar/delivery-markup-project] · Author: Joel Nithish Kumar Murugan*
