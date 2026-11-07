> ChatGPT: this workflow was converted from a skill written for another assistant. Read `chatgpt-runtime.md` for what its tool names mean here, and `../AGENTS.md` for the rules that always apply.

# Aivre report fill: hybrid

A hybrid (e.g. Class Valuation "3.6 SFR Hyb") is an assignment where a third-party property data collector (PDC) inspects and the appraiser does not. Hybrid is how the property was inspected, not what it is, so this skill stacks on top of a property-type skill: aivre-fill-single-family for a house, aivre-fill-condo for a condo unit.

This skill only adds to the aivre-report-fill core. If the core is not loaded in this conversation yet, read it from workflows/ first, then the property-type skill, and follow the core as the workflow, applying the lines below in the sections of the same name. Where these lines differ from the core or the property-type skill, these win.

## Where things are
- The workfile has the Property Data Report ("Data collection report.pdf", the PDR), the hybrid engagement letter (e.g. Product-SFR-Hybrid-*.pdf), the INvision floor plan (*_AG11.jpg) and area calculation pages (*_AG1__1.jpg, *_AG1__2.jpg), and an "Inspection Photos" folder.
- The PDR is the source for the physical description (rooms, GLA, garage, mechanicals, deficiencies, occupancy, HOA). Wherever the core takes a value from the documents (occupancy, bedrooms and baths, special assessments, utilities, ceiling height, defects), the PDR counts as one.

## Step 1. Read everything first
- Also note the data collection date, the PDC name and type, and the PDR Property Data ID.
- Conflicts seen before in hybrids:
  - coordinates: compare the aerial pin with the PDR coordinates as well.
  - road: the PDR said public while the county tax map and property card said "(PVT)".
  - site size: the PDR "lot size" was really the builder's living area copied from the MLS.
  - garage area: the PDR summary includes the wall footprint, its car storage section gives interior area.
  - HOA dues: MLS and PDR amounts can differ.
  - room labels: one INvision calculation page labeled a full bath "Half Bath".

## Step 2. Review the photos and build a photo log
- The photos are the PDR's and the Inspection Photos folder. Review them exactly as the appraiser's photos would be, and match rooms to the PDC floor plan.
- Photos the report or the AMC needs that are missing are requested from INvision.

## Assignment Information
- Property Valuation Method Hybrid; "Was a Property Data Report used in lieu of an Inspection?" Yes.
- Scope of Inspection: Exterior and Interior "No Inspection" (the appraiser's inspection date then disappears).
- Property Data Report block: PDC first and last name (PDR or AMC status email); Occupation from the list, or Other (Describe) with the PDR collector type (e.g. Photographer); Class Valuation orders: Company Name "Class Valuation" and Class's address (2600 Bellingham Dr #100, Troy, MI 48083); Reference ID = the PDR Property Data ID; Exterior and Interior Inspection Method Physical when the PDR collection type is Onsite; Inspection Date = the data collection date.
- Commentary: right after the "The assignment is a ..." paragraph, add: "This is a hybrid appraisal. The appraiser did not inspect the subject; the property data collection was completed on MM/DD/YYYY by <PDC name>, and the appraiser relied on the property data report, its photos and floor plan, public records and MLS data."
- Class hybrid fee wording, in place of the compensation sentence: "The appraiser's fee of $X is based on the completion of a Hybrid appraisal report. Per the defined scope of work, the appraiser did not visit or inspect the property personally; other fees associated with this assignment may be related to property data collection."

## Subject Property
- Check the aerial pin against the PDR coordinates too.
- Subject Property Commentary: when the MLS and PDR association dues differ, state both amounts.

## Site
- Compare any PDR site size with the plat before relying on it.

## Sketch
- Use the PDC's floor plan and every area calculation page as provided; never draw one. Image Category Sketch, captions "Floor Plan", "Area Calculations Page 1", "Area Calculations Page 2", Photo Size Big Photo.
- Sketch Commentary is required by the hybrid engagement letter. Include any discrepancy between the floor plan, the collected data, the photos and other sources.

## Unit Interior
- Finished Above Grade from the PDC calculations. Area Data Source "Property Data Report" when the data collector measured, instead of "Physical Measurement".
- Occupancy, bedrooms and baths from the PDR and the PDC floor plan.

## Deferred maintenance / defects
- Every deficiency in the PDR is entered, as well as those in the photo log.

## Vehicle Storage
- Garage area from the PDC calculations (ANSI, including walls). Note a different PDR interior figure in the commentary.

## Rules
- Physical characteristics come from the PDR, the PDC floor plan and calculations, the photos and the record.

## When done
- To request: missing photos from INvision.