> ChatGPT: this workflow was converted from a skill written for another assistant. Read `chatgpt-runtime.md` for what its tool names mean here, and `../AGENTS.md` for the rules that always apply.

# Aivre report fill: condo

For a unit in a condominium or cooperative project. A fee simple townhouse or PUD house is aivre-fill-single-family.

This skill only adds to the aivre-report-fill core. If the core is not loaded in this conversation yet, read it from workflows/ first and follow it as the workflow, applying the lines below in the sections of the same name. For a hybrid assignment, also load aivre-fill-hybrid.

## Where things are
- project.csv is sometimes in the workfile: the assessor list of the project's units (unit, owner, owner occupied). It feeds Project Information.
- Subject monthly HOA: the subject's MLS listing PDF (mls-listing-<address>.pdf), Assessments block: Amount and Frequency. Only from a listing active within the past year (core rule on older listings).
- Comp monthly HOA: the connectMLS TSV, column "As/Asc Dues" (MRED condo dues are monthly). "Spec Assess" and "Master Association Fee($)" are in the same TSV.

## Subject Property
- Property in a Project Yes, Condominium Yes (Cooperative or Condop instead when the documents say so), Subject Site Owned in Common Yes.
- Planned Unit Development: No unless the documents show the project is in a PUD (association dues and common areas alone do not make a condo a PUD).
- Homeowner Responsible for Exterior Maintenance: No when the association maintains the exterior, as it does for the condo comps in aivre-sales-comparison; Yes only when documents say the owner does. List it under Judgment calls.

## Site
- Site size: the public record may show the whole development area rather than the unit; say so in the Site Commentary.
- APN Description: "Condominium Unit".

## Sketch
Fannie Mae (Selling Guide B4-1.3-05 and its ANSI FAQ Q4) decides the measurement standard by the building's architectural design, not by condo ownership. This overrides the core's "Measurement Standard ANSI".
- Unit in an apartment-style or multifamily building (Structure Design Apartment, Low-rise, Mid-rise, High-rise; garden condos): Measurement Standard "Interior perimeter" (free text field, 50 characters). ANSI Z765-2021 cannot be used for these.
- Townhouse, rowhouse, side-by-side or detached (site) condo: Measurement Standard "ANSI", same as a house.
- Sketch Commentary for an interior-perimeter unit: keep the appraiser's existing sketch note if there is one, then add this as its own paragraph: "Unit is in an apartment-style building, so ANSI Z765-2021 does not apply. Measured to the interior perimeter of the unit per Fannie Mae guidance."
- If the appraiser asks ChatGPT to draw the sketch of an apartment-style unit: interior perimeter dimensions, rounded to 0.1 ft, area rounded to the nearest sq ft. Label the area by level, never "GLA".

## Dwelling
- Structure Design: the same as the project's buildings. Oakwood Hills (Elgin) style garden condos are "Apartment", not Rowhouse/Townhouse or Low-rise. Changing Structure Design clears Construction Method, so redo that after.
- Front Door Elevation: always "Ground Level" for a unit in an apartment-style building (the field is the building's entrance at grade, not the unit's door or floor). Do not flag it. A townhouse, rowhouse or site condo follows the core rule (from photos, flagged).

## Unit Interior
- Non-Residential Use and Live/Work Space only appear after Subject Site Owned in Common is set to Yes; answer them No for a residential unit.

## Project Information
- Clean project.csv first: remove rows whose street is outside the development and rows with no unit or no owner. Count units, sold, for sale, rented (owner occupied No), and the largest number of units held by one owner name. Save the cleaned copy in the scratchpad.
- Fill project name, data source (Assessor Record), total units, units sold, units for sale, units rented with "estimated" Yes and a reason. Units complete Yes (check this; a mis-click spawns extra fields). Observed deficiencies No, converted in past 3 years No, ground rent No, developer in control No unless documents show otherwise.
- Single entity ownership: greatest number of units owned and impact Neutral with a short comment.
- Commercial space No for a residential-only project.
- Monthly HOA amount: always enter it, from the subject's MLS listing (convert to monthly if the listing frequency is not monthly). The grid's subject Monthly Fee cell reads from this field. Flag it only when there is no current listing or the listing and questionnaire disagree (state both amounts in Subject Property Commentary per the core).
- Not in the workfile, so leave empty and flag: amenities, utilities included, known legal actions, transfer fees, special assessments, tax abatement. These need the condo questionnaire.
- Project commentary: short, naming the data source and what is unverified.

## Sales Comparison grid (Project Information rows)
- Monthly Fee row: always fill it for every comp from the TSV "As/Asc Dues" column, matched by address and unit to the comp column. The row may need adding with the section's Rows button ("Monthly Fee"). The subject cell is read-only and fills from Project Information.
- Leave the fee row's adjustment blank; adjustments are the appraiser's.

## When done
- Judgment calls also include: units rented (estimated), single entity ownership, Subject Site Owned in Common and Homeowner Responsible for Exterior Maintenance, and the Measurement Standard when the building design was not clear-cut.
- To request: the condo questionnaire, for the Project Information items left empty.