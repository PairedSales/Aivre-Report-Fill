> ChatGPT: this workflow was converted from a skill written for another assistant. Read `chatgpt-runtime.md` for what its tool names mean here, and `../AGENTS.md` for the rules that always apply.

# Aivre report fill: 2-4 unit

For a property with two to four dwelling units on its own fee simple lot: stacked flats, side-by-side units, or a house plus a unit in a second building. A unit in a condominium is aivre-fill-condo. A house with one accessory unit is decided in Highest and Best Use (unit or ADU).

This skill only adds to the aivre-report-fill core. If the core is not loaded in this conversation yet, read it from workflows/ first and follow it as the workflow, applying the lines below in the sections of the same name. For a hybrid assignment, also load aivre-fill-hybrid.

Left blank here for the comparables skill: the Sales Comparison grid, rental comparables, GRM comparables, market rent opinions and the income approach value. Reconciliation is left entirely to the appraiser.

## Where things are
- The appraiser's sketch and area calculation page: units labeled by unit and floor, each unit's area, Gross Building Area (GBA, no basement) and Gross Leasable Area (GLeA, units only).
- Rents.png, a rent roll or leases: the subject's current rents and lease terms.
- "MLS Listings" may hold rental listings of single units. They are not sale listings of the subject; the core's older-listing rule applies.
- Rocket "Multi Family 2-4 Units" letters: the 2-4 unit requirements are on the later pages of the PDF (rent schedule required, ADU and zoning stop items).

## Step 1. Read everything first
- From the sketch, note each unit's label, floor, area, bedrooms and baths, and the GBA and GLeA totals.
- For each unit: occupancy, rent, and lease terms (month-to-month, concessions, utilities included, furnished) when a document gives them. Match each rent to its unit against the sketch and the photos (a vacant unit has no rent).
- Zoning density: compare the unit count with the district's lot area per unit (e.g. Chicago RT-4 is 1,000 sq ft per unit).
- Stop-and-contact items to put first in the summary: illegal use (Rocket does not lend), more than one accessory unit, wrong property type ordered.
- Conflicts seen before: a rents list labeled by floor that did not match the sketch's unit numbers; Infosparks graphs captured as Single Family; an order letter requiring ANSI.

## Step 2. Review the photos and build a photo log
- Group the photos by unit (time stamps, finishes, furnishings) and log each unit's rooms against its sketch unit.
- Note per unit: occupied or vacant, the furnace, AC coil and water heater, and any meters.
- To request: meter photos and each unit's smoke and CO detectors when missing.

## Subject Property
- Add the units first. In the left nav, click the "+" next to "Dwelling - Building 1", then "Add Unit", until the read-only Dwellings Containing Units, Units Excluding ADUs and Accessory Dwelling Units counts are right. A unit in a separate building gets "Add Dwelling".
- PUD, Property in a Project and Site Owned in Common No, Homeowner Responsible for Exterior Maintenance Yes, unless documents say otherwise.

## Site
- APN Description "Land with Dwelling".
- More units than the zoning density allows, built that way: Compliance "Legal Nonconforming". This adds a required zoning Impact (Neutral) and Zoning Compliance Description (246 characters) commenting on value and marketability, which Rocket requires. Put it under Check first.
- Utilities: each utility tab's Comment is where separate metering per unit goes, when the photos or documents show it.

## Sketch
- Stacked units (an apartment-style building): Measurement Standard "Other (Describe)", 50 characters, e.g. "Exterior walls, GBA and GLeA, ANSI N/A multifamily". Fannie's ANSI FAQ (Q5) says ANSI cannot be used for 2-4 unit apartment/multifamily buildings. ANSI applies only when each unit is a standalone structure or side-by-side.
- Sketch Commentary: why ANSI does not apply, measured to the exterior walls, and the GBA and GLeA totals. An order letter that requires ANSI goes under Conflicts.
- Exhibits: the appraiser's sketch ("Sketch") and the area calculation page ("Area Calculations"), Image Category Sketch.
- Make each unit page match its sketch unit. Common stairs and halls count in GBA but in no unit.

## Dwelling
- One Dwelling Exterior page per building. Structure Identifier (required, 30 characters): "Building 1", "Building 2". Units in Structure is computed.
- Dwelling Style has no multifamily style: use Other (Describe), e.g. "Two-Flat", "Four-Flat".
- One Add Heating System entry per system type. Say in the commentary when each unit has its own furnace, cooling and water heater.
- A garden unit that the appraiser's sketch counts as a floor (no basement): Foundation Type Slab, the unit Above Grade. Flag it.

## Unit Interior
- One page per unit ("Unit Interior - <identifier>" in the nav). Every core Unit Interior rule applies to each unit.
- Unit Identifier (25 characters): the sketch label, e.g. "Unit 1". Floor Number (3 characters): the unit's floor.
- Unit is an ADU: No for a unit. Yes only for an ADU decided in Highest and Best Use.
- Occupancy per unit (Tenant, Owner or Vacant) from the rent documents and photos. It pre-fills the rent schedule.
- Utilities Separately Metered (required): from meter photos or documents, otherwise blank and flagged.
- Interior Quality and Condition Ratings are per unit.
- Finished Above Grade = the unit's area from the appraiser's calculation page.
- Rooms from the sketch, including a Laundry Room for a laundry/mechanical closet. Each unit's mechanical photo goes on that room.
- Defects go on the page of the unit where they are.

## Vehicle Storage
- A parking pad off the alley: "Open Lot", with Dedicated Parking Spaces (only the subject's spaces) and Parking Space Assignment Type (Owned unless a lease assigns them).

## Subject Property Amenities
- The section is property-wide. Enter each unit's porch, deck or patio as its own Outdoor Living entry with the sketch area, and say which unit it serves in the commentary.

## Overall Quality and Condition
- The overall ratings cover the whole property. Mention any unit that differs, such as a renovated or unfinished unit.

## Highest and Best Use
- Decide unit or ADU here and say so in the commentary. A unit is a complete, legally rentable dwelling: its own entrance, kitchen, bath and heating, plus separate meters and address when known. A 4-unit property cannot have ADUs. Rocket: contact them if there is more than one accessory unit.
- Legal nonconforming density is still legally permissible.

## Market
- Infosparks graphs must be Property Class "2 to 4 Units" (check each footnote). Graphs captured as Single Family are not used; flag them. There are no price per square foot graphs for 2-4 units. Count 2-4 unit sales and listings.

## Rental Information
- Rent Schedule Developed: Yes. Fannie Selling Guide B4-1.3-10 requires the income approach for 2-4 unit properties, and the income approach needs the rent schedule. Rocket requires a rent schedule.
- On each unit tab ("Building 1 - Unit n"): Include This Unit in Rent Schedule Yes; Occupancy (pre-filled); Monthly Rent (the current rent, 0 for a vacant unit); Currently Rented; Rent Control (No in Illinois).
- For a rented unit: Month-to-Month, Rent Concessions, Utilities / Services Included and Furnished come only from a lease or rent roll. Otherwise leave them blank and flag them.
- Actual Income: Monthly Actual Rent for Unit copies the rent (enter 0 for a vacant unit). Other Actual Income Type "None" unless documented.
- Leave Opinion of Market Income, the rental grid, Rental Analysis Commentary and its exhibits for the comparables skill.

## Income Approach
- Income Approach Developed by Appraiser?: Yes. Leave the GRM grid, Total Monthly Market Rent, GRM, Indicated Value and commentary for the comparables skill.

## Sales Comparison Approach
- Enter only the subject's Gross Building Finished Area (the editable cell in the subject column's Dwelling(s) block) = the appraiser's GBA.

## Reconciliation
- The appraiser always reconciles 2-4 unit reports themselves. Leave the whole section blank. The core's Reconciliation defaults do not apply.

## When done
- Check first also: zoning compliance, any ADU, and an ANSI requirement in the order letter.
- Judgment calls also include: which rent goes with which unit, each unit's occupancy and ratings, Dwelling Style, the garden unit's grade level, the parking count and the Measurement Standard.
- Expected red: Utilities Separately Metered without meter evidence, lease terms without a lease, Market without 2-4 unit data, the comparables parts, and Reconciliation.
- To request: meter photos, leases or a rent roll, and each unit's smoke and CO detector photos.