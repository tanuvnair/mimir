---
title: "Hersheys Contract Management"
date: 2026-07-24
tags:
  - hersheys
  - work
publish: false
---

# Hersheys Contract Management

## Implementation Plan

This is my master file which will be only uploaded once (one more file with the plant code and plant name mapping).

Basically the goal of this project is to create a contract management system.

See there are two plans:

- **AOP:** Which is uploaded on january and septemeber every year. In january, data for the current year is uploaded. In september, data for the current year + next year is uploaded. AOP will be automatically fetched from TM1 using our internal application loaded to database accordingly. This implementaion will be done later. Only need to know about this for context.
- **DR:** This is uploaded every month. For a certain year and a certain version, which is basically "january_contract", "february_contract", etc. The is manually uploaded by the user.

The first step of our user is to upload a few master files. The master file will have all the codes and mapping. We need to store this data somewhere. And later the user will be able to perform CRUD on each of these. Like lets say they want to add a new material, a new plant, brand, category, etc. Lets say the user wants to add a new material. They will be able to add a new material, etc; Whatever it is.

The hiearchy of the data is as follows: PPG -> Material/FERT -> Component

Our current goal is to calcualte "Required Quantity". How we will be achieveing this is by uploading two files. The files to be uploaded are:

- **Production Plan:** Contains the volume for each PPG
- **Bill Of Material:** Contains the quantity for each component level (we have material column too)

For production plan upload we will ask the user for the year and version, and then upload for the production plan for that year and version combination.

For bill of material upload we don't need to ask for the year and version, we will store the data as it is, as it is used according to our logic.

All of this stuff data might be extracted from sources like SAP or TM1, so you know we also need to store the relations and stuff like they do…

**For PH Master Upload:**

- We need table for materials, ppgs, brand_skus, brands, parent_brands, categories and finally one that stores the data and combination of these rows with net weight
- Basically user will upload one PH master file and we will extract these data from it and put it on their individual tables.

**For Production Plan Upload:**

- We will ask the user for year and version
- User will upload the production plan
- Had a question on should we store the data as a "year" and "version" combination or "year", "plan", "version" combination?
- We will store the data in a table called "production_plan_data" or something; One thing I wanted to ask is should we keep the "AOP" and "DR" data in same table or have different tables for each?
- Basically however we decide to store the data, we will be storing the PPG (foreign key to the master maybe I am not sure how to handle this), UOM (also foreign key to a table with all the UOMs), Plant (foreign key to the master), and then the volume.

**For Bill Of Material Upload:**

- We will just ask the user to upload the bill of material, it will act like a master. We will use foreign keys for the plant, material etc, from the file we require Unit of measure and quantity other metrics we can store too if needed as per requirement.

Finally we come to the final page where the user calculates the Required Quantity.

- This page will take the year, plan and version as input and then we generate the Required Quanity metrics and store them.. For now lets just blackboxs the logic, but we will pull from the prodution plan data and bill of material data we had stored.

I want your help in creating the initial schema and architecture/diagrams for this requirement.

### 16 June, 2026

- What to do when uploading master and net weight is empty for a row? (should we just assume it as 0?)
- When uploading production plan, UOM comes as empty sometimes
- Is it possible for products to be removed from the PH master file? Or stuff is always
- Need to fix bug in import history parse errors column section, where is showing same column for all the errors, even though the error was due to a different column
- Change name to Contract Tracking
- Add a way to download calculated data and parse errors data
- 9999-90099-000 duplicate materials
- Duplicate material codes, then dont load, log error
- If a material is in multiple ppgs, dont load, log error
- UOM mismatch in Production Plan and BOM

### 24 June, 2026

- Fix year input field in /dr/file-uploads, currently when I change it I don't see the input changing (FIXED)
- When uploading any type of file (specifically bill of material where i noticed this bug), ideally for all skipped or failed outcomes, in the message we need to make sure that in the message, the exact reason with the exact code etc is listed. For example in the case if a material has no net weight in product hierarchy, I don't see a material code or anything like that. (FIXED)
- Remove Plan UOM and Bill Of Material UOM from the required quantity columns in /dr/required-quantity (FIXED)
- Currently when user selects a route to navigate to, the scroll on the navigation side bar resets, which is bad UX
- Add is_active field in plants table so that the user can toggle if a plant is active or not
- Everywhere for any lines of data, such as bill of material lines, product hierarchy lines… It should be named as such… Currently in UI and you will have to check code, but for product hierarchy lines its referred to as "Product Hierarchy Enteries" in alot of places
- In /masters/bill-of-material-lines the "Status" column is not visible, ideally we should be able to see
- Update product hierarchy lines to be consistent with how we display bill of material lines…. you should refer to the styling, columns etc from bills of materials and present it in a similar format
- Hovering a material, ppg, plant or any foreign key in product hiearchy lines or bill of material lines should show the description for it
- Add better search or filters to filter out on combinations like for xyz plant, xyz material, xyz component
- Remove the logic for if the same plant + material + component appears more than once, those required quantities are added together.

### AOP Production Plan Implementation

Okay so the file which we will read for AOP production plan, we will have a .csv file, and the stuff is a little bit different.

So I think what you should do is @backend/.env.example:82-91 Have one for DR and one for AOP…

But ideally required are same. Just the ordering and stuff is elsewhere.

Okay what the flow is basically:

-> User loads the Production Plan

-> We load the file from the file path (.csv file)

-> We have a new set of .env variables for the columns and mapping

-> We parse it and we have some rules for uploading

-> Basically our end goal is to store the data in KGs, we were able to directly store it without any calculations in DR plan as we already had the converted values in DR

-> But in AOP plan, we need to convert the value to KGs,

-> To do this, what we will do is

## Current Implementation Documentation

## June 24, 2026 - Hersheys Contract Tracking Blocker Points

**For AOP Production Plan:**

- AOP production plan data will come from TM1.
- All full-year volumes should eventually be stored in KG.
- When plan UOM is Case, discussed conversion is FY volume (kg) = FY volume (cases) × net weight (kg).
- Net weight comes from Product Hierarchy master, which is at material level.
- **Production plan rows are at PPG + plant level, so when one PPG has multiple materials with different net weights, business must confirm which net weight to use, since there are multiple materials per PPG and there are cases where each material has a different Net weight.**
- **Until that is decided, AOP Case to KG conversion cannot be implemented correctly.**

**For Bill Of Material:**

- Rows with older valid_from for the same PPG are skipped; latest valid_from per PPG is kept.
- LB UOM: calculated_quantity = quantity ÷ net_weight_kg ÷ 2.2046; requires net weight from product hierarchy.
- EA UOM: calculated_quantity = quantity ÷ net_weight_kg; requires net weight from product hierarchy.
- CA (Case) UOM: calculated_quantity = quantity ÷ net_weight_kg; requires net weight from product hierarchy.
- **KG UOM: Logic need to be confirmed**.
- **M UOM: Logic need to be confirmed**.
- Net weight for BOM is always taken from the material on that row via product hierarchy, which matches BOM being material-grain.

**For Quantity Required Calculation:**

- User runs calculate required quantity for a DR production plan by year + contract version (or dataset ID).
- For each plan line with a FY volume, all active materials under that PPG are loaded from product hierarchy.
- For each material, active BOM lines are loaded for that plant + material.
- For each BOM component, required quantity = full_year_volume_kg (for that ppg) × calculated BOM quantity (with the above conversion logic).
- **The full PPG plan volume (kg) is currently applied to every material under that PPG, so if a PPG has three materials each material’s components are calculated using the full FY volume.**
- Need confirmation as the values we are getting seems off. Please verify if the logic is correct.

## June 25, 2026 - Hersheys Contract Tracking Client Meeting

**For AOP Production Plan:**

- A flow has been finalized for the "Case" to "Kg" conversion and finally converted to tons by dividing by 1000 and a worksheet will be shared by Kaushal sir with the business to confirm.

**For DR Production Plan:**

- We have values in "Kg". Convert them to "Tons" by dividing them by 1000.

**For Bill Of Material Calculations**

- The **"BUn" Business Unit**, will be given in "CA" or "KG"; If it is given in KG then we don't need to do the divide by net weight (from PH master) in our logic.
- **"LB" UOM:** calculated_quantity = (quantity / net_weight_kg) / 2.2046; requires net weight from product hierarchy.
- **"EA" UOM:** calculated_quantity = (quantity / net_weight_kg); requires net weight from product hierarchy.
- **"CA" UOM:** calculated_quantity = (quantity / net_weight_kg); requires net weight from product hierarchy.
- **"KG" UOM:** No need of conversion (to be confirmed if any logic required in case of BUn as KG).
- **"M" UOM:** (quantity / 65) / net_weight_kg, then divide by net weight. Where standard roll size is 65.
- The above calculations are for converting the quantity from different unit of measures into tons.

**For Quantity Required Calculation:**

- Only issue was before multiplying, we were not converting the DR Production Plan volume to tons.

## June 30, 2026 - Hershesy Contract Management Changes

### Personal Changes

## July 22, 2026 - Hershesy Contract Management Changes

- Check why in DR volume upload, an empty UOM field is loaded, it it because for DR uploads, the UOM is not read, it it is not being read at all and directly we are setting it as KGs, then we don't need to define it in env file for it either
- Do full end to end testing for all the file uploads/loads and check if the parse error is properly coming or not
- Need to check how and why there is an entry for the volume upload, if no rows fail to load into it
- Add "(KG/EA)" in Required Quantity header in Bill Of Material Lines
- In Required Quantity page, the UOM measurement of that BOM line should also be shown as a column beside the required quantity.
- Valid From Format should be in DD MM YYYY (Please create a reusable function and make the date consistent everywhere and anywhere we are showing dates)
- In parse errors dialog box, the dialog box shrinks when loading, need to make it consistent
- Add search in sidebar
- Check column header and row text alignment; Align them to left side
- In parse errors dialog box, the dialog box shrinks when loading, need to make it consistent
- Rows alignment for minimal wrapping
- Suggestion is that maybe when only one value is selected, no need of showing the column header
- Add a small info icon on headers like required quantity, calculated quantity, we need to show the working that we have done, how we have derived the stuff

## Related

- [[hersheys]]
- [[hersheys-price-change-automation]]
- [[qubefini]]
