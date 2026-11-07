> ChatGPT: this workflow was converted from a skill written for another assistant. Read `chatgpt-runtime.md` for what its tool names mean here, and `../AGENTS.md` for the rules that always apply.

# Aivre report fill: single-family

For a house on its own lot owned fee simple, detached or attached, including a PUD house and a fee simple townhouse. A unit in a condominium is aivre-fill-condo even when it looks like a townhouse.

This skill only adds to the aivre-report-fill core. If the core is not loaded in this conversation yet, read it from workflows/ first and follow it as the workflow, applying the lines below in the sections of the same name. For a hybrid assignment, also load aivre-fill-hybrid.

## Where things are
- A survey (plat of survey) is sometimes in the workfile. It is the best source for site size, easements and sketch dimensions.

## Subject Property
- Planned Unit Development: Yes with an HOA and common areas (e.g. planned development zoning, private streets, association dues); otherwise No.
- Property in a Project: No (a PUD house is No).
- Subject Site Owned in Common: No.
- Homeowner Responsible for Exterior Maintenance: Yes unless documents say the association maintains exteriors.

## Site
- Site size from the survey if there is one, otherwise calculated from the plat, otherwise the public record. Compare any MLS site size with the plat before relying on it.
- APN Description: "Land with Dwelling".
- Easements from the survey, with impact Neutral.

## Sketch
Only when the appraiser asks ChatGPT to draw the sketch, build it as a PNG in the style of a TOTAL Sketch export:
- Dimensions: the survey first (it is better data than the assessor sketch), rounded to 0.1 ft. Use the assessor sketch for the footprint shape and the position of the patio, deck and outbuildings, and for any item the survey does not show. Use the appraiser's verified figures over either when they give them.
- ANSI Z765: exterior dimensions to the outer wall. Label the house area by level ("First Floor" etc.), never "GLA"; the summary row is "Total Living Area (Rounded)".
- A detached garage or shed is drawn as its own separate box, never touching the patio or house, even if the assessor sketch draws it attached.
- Room labels only (no interior walls) at approximate locations. The appraiser's room layout overrides any guess; if they give none, place rooms from the photo log and flag the layout.
- Dimension labels on every side, area in brackets in each box, and an "Area Calculations Summary" table at the bottom (Living Area rows with L x W = area, Total Living Area (Rounded), then Non-living Area rows for garage, patio, deck, basement). No "TOTAL Sketch by a la mode" text or any software branding.
- View the PNG before uploading, then save a copy as Sketch.png in the workfile.

## Dwelling
- Dwelling Style: Ranch, Colonial, Bungalow, Cape Cod, Split Level, Raised Ranch, Bi-Level, Craftsman, ...

## Subject Property Amenities
- Sheds are entered as Outbuildings (Add Outbuilding in the left nav), not Miscellaneous Amenities.