---
title: "NK Proteins POC Validation"
date: 2026-07-24
tags:
  - nk-proteins
  - work
  - sap
publish: false
---

# NK Proteins POC Validation

## 2026-04-22 Data Extraction Points (2025-02-01 to 2025-02-28)

### Z_C_NKP_SALES_PROFIT_V1 (Sales and Profitability)

**_Total Records: 49,127_**

- **SalesOffice:** 27,372 valid records, remaining 21,755 records empty.
- **PlantStreetName:** 43,177 valid records, remaining 5,950 records empty.
- **PlantPostalCode:** 47,994 valid records, remaining 1,133 records empty.
- **NetAmount:** 48,825 valid records, remaining 302 records are 0.
- **CostAmount (COGS):** 800 valid records, remaining 48,327 records are 0.

### Z_C_NKP_CASHFLOW_V1 (Cashflow)

- Dataset is complete for the requested date range.
- Some records appear out of range based on `PostingDate` filter we have applied for `2025-02-01 to 2025-02-28`, resulting in orphaned entries without corresponding opening/closing balances.

### ZC_NKP_BOM_V1 (Bill Of Material)

- Basic extraction was completed, but material costing we were unable to find with Tushar as it is out of his area of expertise. He suggested asking someone with SAP MM functional background.

### Z_C_NKP_INVENTORY_V1 (Inventory)

**_Total Records: 100,000_**

- **StorageLocation:** 84,177 valid records, remaining 15,823 records empty.
- **StandardPrice:** 84,796 valid records, remaining 15,204 records are 0.
- **MovingAveragePrice:** 9,871 valid records, remaining 90,129 records are 0.

## 2026-05-05 Data Extraction Points (2025-02-01 to 2025-02-15)

### Sales

- **NetAmount:** We are getting extremely low values for this columns in some rows, which is causing to **GrossMarginPercentage** to be in large negative numbers such as "-44689.78", "-62131.11", "-137484.67"
- 1010004240: Internal Transfer across plant
- 6010000040:
- Filter ZGCR, ZGDB, ZNFS, ZNF8, ZNFS
- Filter out these BillingDocumentTypes which have net amount 0: ZNF8, ZGDB, S1
- Cancel Invoice, Debit Note (GST), Pro Forma Inv f. Delivery

### Receivables

- **PaymentTermsDate:** Which one to use? Currently we have used ZBD3T, if that is empty, then using ZBD2T, if that is empty then ZBD1T.
- Updated Sales data extraction. Added BillingDocumentType and BillingDocumentTypeText columns.
- Filtered out records with GrossMarginPercentageINR lower than -500.
- Filtered out row with "Cancel Invoice", "Debit Note (GST)", and "Pro Forma Inv f. Delivery" in the BillingDocumentTypeText column

## 2026-05-11 Data Extraction Points (2025-01-15 to 2025-03-15)

### Sales

- We are now only getting "Finished Goods" material type.
- We are now only getting "Invoice", "Exports Custom Inv", "Credits for Returns", "Cancel Invoice" as discused
- CostAmount: Solved, we use the price control indicator column flag and if it is 'S' we use standard price, else we use the moving average price
- GrossMargin: Negative values coming as this some of the data that we have extracted have dummy records, which causes the gross margin calculation to seem not correct

### Receivables

- InvoiceNumber: We updated it to get Billing Number if the type is 'RV', if the type is not 'RV', then we use the accounting document number
- Add DocumentType field: Added
- PaymentTermsDays: We use zb3dt if available, if not then we use zbd2t, if not then finally we use zbd1t
- DueDate: Updated due date to also use the zb3dt if available, if not then we use zbd2t, if not then finally we use zbd1t logic
- DaysOverdue: Fixed cause of above fix

### Inventory

- LastMovementDate: Partially fixed, need to note issue where the client said that the movement date showing nothing took place; Solved
- UnrestrictedStock: Coming 0 even after using the batching logic; Partially solved, batching recording still coming 0
- DaysSinceLastMovement: Same issue.

### GST

- TaxBaseAmountInLocalCurrency: Fixed, the issue was due to grouping summation, since a single item had multiple records for CGST/SGST or IGST etc, it would sum everything instead of only taking one value.

### Validation

#### Inventory (Validated using MMBE)

| MaterialNumber Plant	StorageLocation | UnrestrictedStock Extracted Data | UnrestrictedStock SAP GUI |
| ------------------------------------ | -------------------------------- | ------------------------- |
| 610110301	1101	STOR                  | 0                                | 0                         |
| 610107905    1201    STOR            | 0                                | 0                         |
| 610201459    1201    STOR            | 0                                | 0                         |
| 610800169	1101	STOR                  | 5                                | 5                         |
| 614500375	1110	STOR<br>              | 3                                | 3                         |
| 610101162	1109	STOR                  | 4                                | 4                         |
| 50507    1311    FG01  <br>          | 109043                           | 109043                    |
| 610800075    1361    STOR  <br>      | 54                               | 54                        |
| 100000001    1311    DMG1            | 80103.34                         | 80103.34                  |

## Related

- [[nk-proteins-sap-vbrk-queries]]
- [[qubefini-server-nginx-backup]]
