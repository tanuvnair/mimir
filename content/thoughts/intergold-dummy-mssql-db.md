---
title: "Intergold Dummy MSSQL DB"
date: 2026-07-24
tags:
  - intergold
  - database
  - mssql
  - work
publish: false
---

```SQL
-- ============================================================
-- intergold-dummy  –  full bootstrap script for MSSQL
-- Creates the database, all referenced tables, and populates
-- each with ~10 000 000 rows of realistic dummy data so the
-- IBM_ALgov_PPC_Planning_Data view can be created & queried.
-- ============================================================

USE [master];
GO

-- Drop if exists so the script is re-runnable
IF DB_ID(N'intergold-dummy') IS NOT NULL
BEGIN
    ALTER DATABASE [intergold-dummy] SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE [intergold-dummy];
END
GO

CREATE DATABASE [intergold-dummy];
GO

USE [intergold-dummy];
GO

-- ============================================================
-- 1.  TABLES
-- ============================================================

-- ----- IBM_Algov_Var_Source_1  (first UNION source) ---------
CREATE TABLE dbo.IBM_Algov_Var_Source_1
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    OrdNo           VARCHAR(50)   NOT NULL,
    BagNo           VARCHAR(60)   NOT NULL,
    OdOmCmcd        VARCHAR(50)   NULL,
    [Factory Type]  VARCHAR(50)   NULL,
    Oddmcd          VARCHAR(50)   NULL,
    DmPrdCtg        VARCHAR(50)   NULL,
    OrdSeq          INT           NULL,
    OrdSeqDays      INT           NULL,
    WIPQty          DECIMAL(18,4) NULL,
    WsQty           DECIMAL(18,4) NULL,
    HsQty           DECIMAL(18,4) NULL,
    OdKt            VARCHAR(30)   NULL,
    circulationdt_01 VARCHAR(20)  NULL,
    Status          VARCHAR(10)   NULL,
    [Order Type]    VARCHAR(30)   NULL,
    [Difficulty Level] VARCHAR(50) NULL
);
GO

-- ----- IBM_Algov_Var_Source_MSF_1  (CTE + second UNION) -----
CREATE TABLE dbo.IBM_Algov_Var_Source_MSF_1
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    OrdNo           VARCHAR(50)   NOT NULL,
    BagNo           VARCHAR(60)   NOT NULL,
    OdOmCmcd        VARCHAR(50)   NULL,
    [FactoryType]   VARCHAR(50)   NULL,
    Oddmcd          VARCHAR(50)   NULL,
    DmPrdCtg        VARCHAR(50)   NULL,
    OrdSeq          INT           NULL,
    OrdSeqDays      INT           NULL,
    WsQty           DECIMAL(18,4) NULL,
    HsQty           DECIMAL(18,4) NULL,
    OdKt            VARCHAR(30)   NULL,
    circulationdt_01 VARCHAR(20)  NULL,
    Status          VARCHAR(10)   NULL,
    [Order Type]    VARCHAR(30)   NULL,
    Hand            DECIMAL(18,4) NULL,
    Micro           DECIMAL(18,4) NULL,
    Fanuk           DECIMAL(18,4) NULL,
    [Fk-Micro]      DECIMAL(18,4) NULL,
    [Difficulty Level] VARCHAR(50) NULL
);
GO

-- ----- IBM_Algov_PPC_Module_Data  (alias M) -----------------
CREATE TABLE dbo.IBM_Algov_PPC_Module_Data
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    BYy             INT           NULL,
    BChr            VARCHAR(10)   NULL,
    BNo             INT           NULL,
    OdChr           VARCHAR(10)   NULL,
    OdPrdseq        VARCHAR(50)   NULL,
    Factory_PPC     VARCHAR(10)   NULL,
    BQty            DECIMAL(18,4) NULL,
    DmPrdCtg        VARCHAR(50)   NULL,
    PPC             DATE          NULL,
    PPStartDt       DATE          NULL,
    ProdEndDt       DATE          NULL,
    ZCAD            DATE          NULL,
    ZCAM            DATE          NULL,
    ZMMD            DATE          NULL,
    ZPD             DATE          NULL,
    CPX             DATE          NULL,
    WSDiaIssue      DATE          NULL,
    Waxing          DATE          NULL,
    Waxsetting      DATE          NULL,
    WTR             DATE          NULL,
    JC              DATE          NULL,
    SprueGrinding   DATE          NULL,
    SRD_SPLIT       DATE          NULL,
    BPREP           DATE          NULL,
    Filing          DATE          NULL,
    PFMG            DATE          NULL,
    EP_HTCumEP      DATE          NULL,
    OTC_DISC_FINISHING DATE       NULL,
    PrePolish       DATE          NULL,
    FanukSetting    DATE          NULL,
    MetalSetting    DATE          NULL,
    OS              DATE          NULL,
    Polish          DATE          NULL,
    FQC             DATE          NULL,
    GSI             DATE          NULL,
    Assaying        DATE          NULL,
    Packing         DATE          NULL,
    FilPPts         DECIMAL(18,4) NULL,
    PolPPts         DECIMAL(18,4) NULL,
    Bloc            VARCHAR(50)   NULL
);
GO

-- ----- IBM_ALgov_PPC_Prod_Seq -------------------------------
CREATE TABLE dbo.IBM_ALgov_PPC_Prod_Seq
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    vPmcd           VARCHAR(50)   NOT NULL,
    vPDesc225       VARCHAR(225)  NULL
);
GO

-- ----- IG_IBM_PPC_DiaDeviation ------------------------------
CREATE TABLE dbo.IG_IBM_PPC_DiaDeviation
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    OrdNo           VARCHAR(50)   NOT NULL,
    [Diamond Confirmation Date - Final] DATE NULL
);
GO

-- ----- IBM_ALgov_Design_Wax_Manpower ------------------------
CREATE TABLE dbo.IBM_ALgov_Design_Wax_Manpower
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    Drcd            VARCHAR(50)   NOT NULL,
    StoneSetPerHr   DECIMAL(18,4) NULL
);
GO

-- ----- IBM_Algov_Process_Master -----------------------------
CREATE TABLE dbo.IBM_Algov_Process_Master
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    PMCd            VARCHAR(50)   NOT NULL,
    PScd            VARCHAR(50)   NULL,
    PRem1           VARCHAR(200)  NULL
);
GO

-- ----- IBM_Algov_PPC_Waxing_Pts -----------------------------
CREATE TABLE dbo.IBM_Algov_PPC_Waxing_Pts
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    PPPrdCtg        VARCHAR(50)   NOT NULL,
    PpPts           DECIMAL(18,4) NULL
);
GO

-- ----- IBM_Algov_Design_CPX_Model_Flag ----------------------
CREATE TABLE dbo.IBM_Algov_Design_CPX_Model_Flag
(
    Id              INT IDENTITY(1,1) PRIMARY KEY,
    Dacd            VARCHAR(50)   NOT NULL,
    Model_Flag      VARCHAR(50)   NULL,
    CPX_Flag        VARCHAR(50)   NULL
);
GO

-- ============================================================
-- 1b.  EXTRA DIMENSION / LOOKUP TABLES (for perf testing)
-- ============================================================

CREATE TABLE dbo.DimCountry
(
    CountryId       INT IDENTITY(1,1) PRIMARY KEY,
    CountryCode     CHAR(2)       NOT NULL,
    CountryName     VARCHAR(100)  NOT NULL
);
GO

CREATE TABLE dbo.DimCurrency
(
    CurrencyId      INT IDENTITY(1,1) PRIMARY KEY,
    CurrencyCode    CHAR(3)       NOT NULL,
    CurrencyName    VARCHAR(50)   NOT NULL
);
GO

CREATE TABLE dbo.DimRegion
(
    RegionId        INT IDENTITY(1,1) PRIMARY KEY,
    RegionCode      VARCHAR(20)   NOT NULL,
    RegionName      VARCHAR(100)  NOT NULL,
    CountryId       INT           NULL
);
GO

CREATE TABLE dbo.DimCustomerSegment
(
    SegmentId       INT IDENTITY(1,1) PRIMARY KEY,
    SegmentCode     VARCHAR(30)   NOT NULL,
    SegmentName     VARCHAR(100)  NOT NULL
);
GO

CREATE TABLE dbo.DimCustomer
(
    CustomerId      INT IDENTITY(1,1) PRIMARY KEY,
    CustomerCode    VARCHAR(40)   NOT NULL,
    CustomerName    VARCHAR(200)  NOT NULL,
    SegmentId       INT           NULL,
    CountryId       INT           NULL,
    RegionId        INT           NULL
);
GO

CREATE TABLE dbo.DimVendor
(
    VendorId        INT IDENTITY(1,1) PRIMARY KEY,
    VendorCode      VARCHAR(40)   NOT NULL,
    VendorName      VARCHAR(200)  NOT NULL,
    CountryId       INT           NULL,
    RegionId        INT           NULL
);
GO

CREATE TABLE dbo.DimChannel
(
    ChannelId       INT IDENTITY(1,1) PRIMARY KEY,
    ChannelCode     VARCHAR(30)   NOT NULL,
    ChannelName     VARCHAR(100)  NOT NULL
);
GO

CREATE TABLE dbo.DimPaymentTerm
(
    PaymentTermId   INT IDENTITY(1,1) PRIMARY KEY,
    PaymentTermCode VARCHAR(30)   NOT NULL,
    Days            INT           NOT NULL
);
GO

CREATE TABLE dbo.DimPricingTier
(
    PricingTierId   INT IDENTITY(1,1) PRIMARY KEY,
    TierCode        VARCHAR(30)   NOT NULL,
    DiscountPct     DECIMAL(9,4)  NOT NULL
);
GO

CREATE TABLE dbo.MapCustomerPricingTier
(
    CustomerId      INT           NOT NULL,
    PricingTierId   INT           NOT NULL,
    EffectiveFrom   DATE          NOT NULL,
    EffectiveTo     DATE          NULL
);
GO

CREATE TABLE dbo.DimProductCategory
(
    ProductCategoryId INT IDENTITY(1,1) PRIMARY KEY,
    CategoryCode      VARCHAR(40)  NOT NULL,
    CategoryName      VARCHAR(200) NOT NULL
);
GO

CREATE TABLE dbo.DimMetal
(
    MetalId         INT IDENTITY(1,1) PRIMARY KEY,
    MetalCode       VARCHAR(30)   NOT NULL,
    MetalName       VARCHAR(100)  NOT NULL
);
GO

CREATE TABLE dbo.DimDesign
(
    DesignId        INT IDENTITY(1,1) PRIMARY KEY,
    DesignCode      VARCHAR(50)   NOT NULL,
    DesignName      VARCHAR(200)  NOT NULL,
    DifficultyLevel VARCHAR(50)   NULL
);
GO

CREATE TABLE dbo.MapDesignMetal
(
    DesignId        INT           NOT NULL,
    MetalId         INT           NOT NULL,
    IsAllowed       BIT           NOT NULL
);
GO

CREATE TABLE dbo.DimProduct
(
    ProductId         INT IDENTITY(1,1) PRIMARY KEY,
    ProductCode       VARCHAR(60)   NOT NULL,
    ProductName       VARCHAR(200)  NOT NULL,
    ProductCategoryId INT           NULL,
    DesignId          INT           NULL,
    MetalId           INT           NULL
);
GO

CREATE TABLE dbo.DimFactory
(
    FactoryId       INT IDENTITY(1,1) PRIMARY KEY,
    FactoryCode     VARCHAR(20)   NOT NULL,
    FactoryName     VARCHAR(100)  NOT NULL,
    CountryId       INT           NULL,
    RegionId        INT           NULL
);
GO

CREATE TABLE dbo.DimWorkCenter
(
    WorkCenterId    INT IDENTITY(1,1) PRIMARY KEY,
    WorkCenterCode  VARCHAR(30)   NOT NULL,
    WorkCenterName  VARCHAR(100)  NOT NULL,
    FactoryId       INT           NULL
);
GO

CREATE TABLE dbo.DimMachine
(
    MachineId       INT IDENTITY(1,1) PRIMARY KEY,
    MachineCode     VARCHAR(40)   NOT NULL,
    WorkCenterId    INT           NULL,
    ModelName       VARCHAR(100)  NULL,
    IsActive        BIT           NOT NULL
);
GO

CREATE TABLE dbo.DimShift
(
    ShiftId         INT IDENTITY(1,1) PRIMARY KEY,
    ShiftCode       VARCHAR(20)   NOT NULL,
    ShiftName       VARCHAR(60)   NOT NULL,
    StartHour       INT           NOT NULL,
    EndHour         INT           NOT NULL
);
GO

CREATE TABLE dbo.DimEmployee
(
    EmployeeId      INT IDENTITY(1,1) PRIMARY KEY,
    EmployeeCode    VARCHAR(40)   NOT NULL,
    EmployeeName    VARCHAR(200)  NOT NULL,
    FactoryId       INT           NULL,
    WorkCenterId    INT           NULL,
    ShiftId         INT           NULL,
    IsActive        BIT           NOT NULL
);
GO

CREATE TABLE dbo.DimProcessStep
(
    ProcessStepId   INT IDENTITY(1,1) PRIMARY KEY,
    StepCode        VARCHAR(30)   NOT NULL,
    StepName        VARCHAR(100)  NOT NULL,
    StepGroup       VARCHAR(30)   NULL
);
GO

CREATE TABLE dbo.DimRouteTemplate
(
    RouteTemplateId INT IDENTITY(1,1) PRIMARY KEY,
    TemplateCode    VARCHAR(40)   NOT NULL,
    TemplateName    VARCHAR(120)  NOT NULL
);
GO

CREATE TABLE dbo.RouteTemplateStep
(
    RouteTemplateId INT           NOT NULL,
    ProcessStepId   INT           NOT NULL,
    StepSeq         INT           NOT NULL,
    TargetMinutes   INT           NULL
);
GO

CREATE TABLE dbo.DimWarehouse
(
    WarehouseId     INT IDENTITY(1,1) PRIMARY KEY,
    WarehouseCode   VARCHAR(30)   NOT NULL,
    WarehouseName   VARCHAR(120)  NOT NULL,
    FactoryId       INT           NULL
);
GO

CREATE TABLE dbo.DimLocation
(
    LocationId      INT IDENTITY(1,1) PRIMARY KEY,
    LocationCode    VARCHAR(40)   NOT NULL,
    WarehouseId     INT           NULL,
    LocationType    VARCHAR(30)   NULL
);
GO

CREATE TABLE dbo.DimCarrier
(
    CarrierId       INT IDENTITY(1,1) PRIMARY KEY,
    CarrierCode     VARCHAR(30)   NOT NULL,
    CarrierName     VARCHAR(120)  NOT NULL
);
GO

CREATE TABLE dbo.DimOrderStatus
(
    OrderStatusId   INT IDENTITY(1,1) PRIMARY KEY,
    StatusCode      VARCHAR(30)   NOT NULL,
    StatusName      VARCHAR(120)  NOT NULL
);
GO

CREATE TABLE dbo.DimWipStatus
(
    WipStatusId     INT IDENTITY(1,1) PRIMARY KEY,
    StatusCode      VARCHAR(30)   NOT NULL,
    StatusName      VARCHAR(120)  NOT NULL
);
GO

CREATE TABLE dbo.DimInventoryReason
(
    InventoryReasonId INT IDENTITY(1,1) PRIMARY KEY,
    ReasonCode        VARCHAR(30)  NOT NULL,
    ReasonName        VARCHAR(120) NOT NULL
);
GO

CREATE TABLE dbo.DimUnitOfMeasure
(
    UomId           INT IDENTITY(1,1) PRIMARY KEY,
    UomCode         VARCHAR(10)   NOT NULL,
    UomName         VARCHAR(50)   NOT NULL
);
GO

CREATE TABLE dbo.DimDefectCode
(
    DefectCodeId    INT IDENTITY(1,1) PRIMARY KEY,
    DefectCode      VARCHAR(30)   NOT NULL,
    DefectName      VARCHAR(200)  NOT NULL,
    Severity        INT           NOT NULL
);
GO

CREATE TABLE dbo.DimInspectionType
(
    InspectionTypeId INT IDENTITY(1,1) PRIMARY KEY,
    InspectionCode   VARCHAR(30)   NOT NULL,
    InspectionName   VARCHAR(200)  NOT NULL
);
GO

CREATE TABLE dbo.DimPriority
(
    PriorityId      INT IDENTITY(1,1) PRIMARY KEY,
    PriorityCode    VARCHAR(20)   NOT NULL,
    PriorityName    VARCHAR(50)   NOT NULL
);
GO

CREATE TABLE dbo.DimStoneType
(
    StoneTypeId     INT IDENTITY(1,1) PRIMARY KEY,
    StoneCode       VARCHAR(30)   NOT NULL,
    StoneName       VARCHAR(100)  NOT NULL
);
GO

CREATE TABLE dbo.DimPackagingType
(
    PackagingTypeId INT IDENTITY(1,1) PRIMARY KEY,
    PackagingCode   VARCHAR(30)   NOT NULL,
    PackagingName   VARCHAR(120)  NOT NULL
);
GO

CREATE TABLE dbo.DimCalendar
(
    CalendarDate    DATE          NOT NULL PRIMARY KEY,
    YearNo          INT           NOT NULL,
    MonthNo         INT           NOT NULL,
    DayNo           INT           NOT NULL,
    IsoWeek         INT           NOT NULL
);
GO

CREATE TABLE dbo.DimHoliday
(
    HolidayId       INT IDENTITY(1,1) PRIMARY KEY,
    CalendarDate    DATE          NOT NULL,
    CountryId       INT           NULL,
    HolidayName     VARCHAR(200)  NOT NULL
);
GO

-- ============================================================
-- 1c.  EXTRA FACT / BRIDGE TABLES (for perf testing)
-- ============================================================

CREATE TABLE dbo.FactOrder
(
    OrderId         BIGINT IDENTITY(1,1) PRIMARY KEY,
    OrderNo         VARCHAR(40)   NOT NULL,
    CustomerId      INT          NULL,
    ChannelId       INT          NULL,
    PaymentTermId   INT          NULL,
    PriorityId      INT          NULL,
    OrderStatusId   INT          NULL,
    OrderDate       DATE         NOT NULL,
    DueDate         DATE         NULL,
    CurrencyId      INT          NULL,
    OrderValue      DECIMAL(18,4) NULL
);
GO

CREATE TABLE dbo.FactOrderLine
(
    OrderLineId     BIGINT IDENTITY(1,1) PRIMARY KEY,
    OrderId         BIGINT       NOT NULL,
    [LineNo]        INT          NOT NULL,
    ProductId       INT          NULL,
    UomId           INT          NULL,
    Qty             DECIMAL(18,4) NOT NULL,
    UnitPrice       DECIMAL(18,4) NULL,
    DiscountPct     DECIMAL(9,4)  NULL,
    LineValue       DECIMAL(18,4) NULL
);
GO

CREATE TABLE dbo.FactWipEvent
(
    WipEventId      BIGINT IDENTITY(1,1) PRIMARY KEY,
    BagNo           VARCHAR(60)   NOT NULL,
    OrderId         BIGINT        NULL,
    OrderLineId     BIGINT        NULL,
    FactoryId       INT           NULL,
    WorkCenterId    INT           NULL,
    MachineId       INT           NULL,
    ProcessStepId   INT           NULL,
    WipStatusId     INT           NULL,
    EventTs         DATETIME2(0)  NOT NULL,
    Qty             DECIMAL(18,4) NULL,
    DurationSec     INT           NULL
);
GO

CREATE TABLE dbo.FactInventoryMovement
(
    MovementId      BIGINT IDENTITY(1,1) PRIMARY KEY,
    ProductId       INT           NULL,
    WarehouseId     INT           NULL,
    LocationId      INT           NULL,
    InventoryReasonId INT         NULL,
    MovementTs      DATETIME2(0)  NOT NULL,
    QtyDelta        DECIMAL(18,4) NOT NULL,
    UomId           INT           NULL
);
GO

CREATE TABLE dbo.FactQualityInspection
(
    InspectionId    BIGINT IDENTITY(1,1) PRIMARY KEY,
    BagNo           VARCHAR(60)   NOT NULL,
    InspectionTypeId INT          NULL,
    DefectCodeId    INT           NULL,
    InspectorEmployeeId INT       NULL,
    InspectionTs    DATETIME2(0)  NOT NULL,
    IsPass          BIT           NOT NULL
);
GO

CREATE TABLE dbo.FactShipment
(
    ShipmentId      BIGINT IDENTITY(1,1) PRIMARY KEY,
    ShipmentNo      VARCHAR(50)   NOT NULL,
    OrderId         BIGINT        NULL,
    CarrierId       INT           NULL,
    WarehouseId     INT           NULL,
    ShipDate        DATE          NOT NULL,
    DeliveredDate   DATE          NULL
);
GO

CREATE TABLE dbo.FactReturn
(
    ReturnId        BIGINT IDENTITY(1,1) PRIMARY KEY,
    ReturnNo        VARCHAR(50)   NOT NULL,
    OrderId         BIGINT        NULL,
    ReturnDate      DATE          NOT NULL,
    Reason          VARCHAR(200)  NULL
);
GO

CREATE TABLE dbo.MapOrderBag
(
    OrderId         BIGINT        NOT NULL,
    BagNo           VARCHAR(60)   NOT NULL
);
GO

CREATE TABLE dbo.MapProductRouteTemplate
(
    ProductId       INT           NOT NULL,
    RouteTemplateId INT           NOT NULL,
    EffectiveFrom   DATE          NOT NULL,
    EffectiveTo     DATE          NULL
);
GO


-- ============================================================
-- 2.  HELPER:  Tally / Numbers table (used by data generation)
-- ============================================================
-- Scale knobs (edit these to change dataset sizes)
DECLARE @SmallRowCount INT = 50000;        -- dimensions / lookups
DECLARE @MedRowCount   INT = 1000000;      -- medium facts / events
DECLARE @BigRowCount   INT = 10000000;     -- large facts (can be slow to seed)

;WITH E1(N) AS (
    SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1
    UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1
    UNION ALL SELECT 1 UNION ALL SELECT 1
), -- 10
E2(N) AS (SELECT 1 FROM E1 a CROSS JOIN E1 b),       -- 100
E4(N) AS (SELECT 1 FROM E2 a CROSS JOIN E2 b),       -- 10 000
E5(N) AS (SELECT 1 FROM E4 a CROSS JOIN E2 b CROSS JOIN E1 c), -- 10 000 000
Tally(N) AS (SELECT TOP (10000000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) FROM E5)
SELECT N INTO #NumsAll FROM Tally;

SELECT TOP (@SmallRowCount) N INTO #NumsSmall FROM #NumsAll ORDER BY N;
SELECT TOP (@MedRowCount)   N INTO #NumsMed   FROM #NumsAll ORDER BY N;
SELECT TOP (@BigRowCount)   N INTO #NumsBig   FROM #NumsAll ORDER BY N;
GO


-- ============================================================
-- 3.  SEED DATA  (~10 000 000 rows per table)
-- ============================================================

-- -------------------- lookup / dimension tables first --------

-- IBM_ALgov_PPC_Prod_Seq  (small lookup, 200 rows – referenced by FK)
INSERT INTO dbo.IBM_ALgov_PPC_Prod_Seq (vPmcd, vPDesc225)
SELECT TOP 200
    'PS-' + RIGHT('0000' + CAST(N AS VARCHAR(10)),4),
    'Process Flow ' + CAST(N AS VARCHAR(10))
        + CASE N % 5
            WHEN 0 THEN ' - Standard'
            WHEN 1 THEN ' - Express'
            WHEN 2 THEN ' - Premium'
            WHEN 3 THEN ' - Eco'
            WHEN 4 THEN ' - Custom'
          END
FROM #NumsSmall
ORDER BY N;
GO

-- IBM_Algov_Process_Master  (small lookup, 300 rows)
INSERT INTO dbo.IBM_Algov_Process_Master (PMCd, PScd, PRem1)
SELECT TOP 300
    'BL-' + RIGHT('0000' + CAST(N AS VARCHAR(10)),4),
    CASE N % 6
        WHEN 0 THEN 'WAX' WHEN 1 THEN 'CAST' WHEN 2 THEN 'SET'
        WHEN 3 THEN 'POL' WHEN 4 THEN 'FIN' ELSE 'QC'
    END,
    'Remark for process ' + CAST(N AS VARCHAR(10))
FROM #NumsSmall
ORDER BY N;
GO

-- IBM_Algov_PPC_Waxing_Pts  (small lookup, 500 rows)
INSERT INTO dbo.IBM_Algov_PPC_Waxing_Pts (PPPrdCtg, PpPts)
SELECT TOP 500
    'CTG-' + RIGHT('000' + CAST(N AS VARCHAR(10)),3),
    CAST(ABS(CHECKSUM(NEWID())) % 100 + 1 AS DECIMAL(18,4)) / 10.0
FROM #NumsSmall
ORDER BY N;
GO

-- IBM_ALgov_Design_Wax_Manpower  (5 000 rows – design codes)
INSERT INTO dbo.IBM_ALgov_Design_Wax_Manpower (Drcd, StoneSetPerHr)
SELECT TOP 5000
    'DM-' + RIGHT('00000' + CAST(N AS VARCHAR(10)),5),
    CAST(ABS(CHECKSUM(NEWID())) % 60 + 1 AS DECIMAL(18,4))
FROM #NumsSmall
ORDER BY N;
GO

-- IBM_Algov_Design_CPX_Model_Flag  (5 000 rows)
INSERT INTO dbo.IBM_Algov_Design_CPX_Model_Flag (Dacd, Model_Flag, CPX_Flag)
SELECT TOP 5000
    'DM-' + RIGHT('00000' + CAST(N AS VARCHAR(10)),5),
    CASE WHEN ABS(CHECKSUM(NEWID())) % 3 = 0 THEN 'Available'
         WHEN ABS(CHECKSUM(NEWID())) % 3 = 1 THEN 'Pending'
         ELSE 'Not Available' END,
    CASE WHEN ABS(CHECKSUM(NEWID())) % 2 = 0 THEN 'Yes' ELSE 'No' END
FROM #NumsSmall
ORDER BY N;
GO

-- ============================================================
-- Extra dimension / lookup seed data (uses #NumsSmall)
-- ============================================================

INSERT INTO dbo.DimCountry (CountryCode, CountryName)
SELECT TOP 50
    CHAR(65 + (N % 26)) + CHAR(65 + ((N / 26) % 26)),
    'Country ' + CAST(N AS VARCHAR(20))
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimCurrency (CurrencyCode, CurrencyName)
SELECT TOP 25
    CHAR(65 + (N % 26)) + CHAR(65 + ((N / 26) % 26)) + CHAR(65 + ((N / 676) % 26)),
    'Currency ' + CAST(N AS VARCHAR(20))
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimRegion (RegionCode, RegionName, CountryId)
SELECT TOP 500
    'RG-' + RIGHT('0000' + CAST(N AS VARCHAR(10)), 4),
    'Region ' + CAST(N AS VARCHAR(20)),
    (N % 50) + 1
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimCustomerSegment (SegmentCode, SegmentName)
SELECT TOP 25
    'SEG-' + RIGHT('00' + CAST(N AS VARCHAR(10)), 2),
    CASE (N % 5)
        WHEN 0 THEN 'Retail'
        WHEN 1 THEN 'Wholesale'
        WHEN 2 THEN 'E-Commerce'
        WHEN 3 THEN 'Boutique'
        ELSE 'OEM'
    END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimCustomer (CustomerCode, CustomerName, SegmentId, CountryId, RegionId)
SELECT TOP 50000
    'CUST-' + RIGHT('000000' + CAST(N AS VARCHAR(10)), 6),
    'Customer ' + CAST(N AS VARCHAR(20)),
    (N % 25) + 1,
    (N % 50) + 1,
    (N % 500) + 1
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimVendor (VendorCode, VendorName, CountryId, RegionId)
SELECT TOP 20000
    'VEND-' + RIGHT('000000' + CAST(N AS VARCHAR(10)), 6),
    'Vendor ' + CAST(N AS VARCHAR(20)),
    (N % 50) + 1,
    (N % 500) + 1
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimChannel (ChannelCode, ChannelName)
SELECT TOP 10
    'CH-' + RIGHT('00' + CAST(N AS VARCHAR(10)), 2),
    CASE (N % 4)
        WHEN 0 THEN 'Direct'
        WHEN 1 THEN 'Partner'
        WHEN 2 THEN 'Online'
        ELSE 'Marketplace'
    END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimPaymentTerm (PaymentTermCode, Days)
SELECT TOP 12
    'PT-' + RIGHT('00' + CAST(N AS VARCHAR(10)), 2),
    ((N % 6) + 1) * 15
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimPricingTier (TierCode, DiscountPct)
SELECT TOP 10
    'TIER-' + RIGHT('00' + CAST(N AS VARCHAR(10)), 2),
    CAST((N % 25) AS DECIMAL(9,4)) / 100.0
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.MapCustomerPricingTier (CustomerId, PricingTierId, EffectiveFrom, EffectiveTo)
SELECT TOP 50000
    N,
    (N % 10) + 1,
    DATEADD(DAY, -(N % 365), CAST('2025-01-01' AS DATE)),
    NULL
FROM #NumsSmall
WHERE N <= 50000
ORDER BY N;
GO

INSERT INTO dbo.DimProductCategory (CategoryCode, CategoryName)
SELECT TOP 200
    'CAT-' + RIGHT('0000' + CAST(N AS VARCHAR(10)), 4),
    'Category ' + CAST(N AS VARCHAR(20))
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimMetal (MetalCode, MetalName)
SELECT TOP 20
    CASE (N % 7)
        WHEN 0 THEN '14K'
        WHEN 1 THEN '18K'
        WHEN 2 THEN 'U14K'
        WHEN 3 THEN 'S925'
        WHEN 4 THEN 'PT950'
        WHEN 5 THEN 'BRASS'
        ELSE '10K'
    END,
    CASE (N % 7)
        WHEN 0 THEN 'Gold 14K'
        WHEN 1 THEN 'Gold 18K'
        WHEN 2 THEN 'Recycled Gold 14K'
        WHEN 3 THEN 'Silver 925'
        WHEN 4 THEN 'Platinum 950'
        WHEN 5 THEN 'Brass'
        ELSE 'Gold 10K'
    END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimDesign (DesignCode, DesignName, DifficultyLevel)
SELECT TOP 50000
    'DM-' + RIGHT('00000' + CAST(N AS VARCHAR(10)), 5),
    'Design ' + CAST(N AS VARCHAR(20)),
    CASE (N % 4)
        WHEN 0 THEN 'Easy'
        WHEN 1 THEN 'Medium'
        WHEN 2 THEN 'Hard'
        ELSE 'Expert'
    END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.MapDesignMetal (DesignId, MetalId, IsAllowed)
SELECT TOP 200000
    (N % 50000) + 1,
    (N % 20) + 1,
    CASE WHEN N % 5 = 0 THEN 0 ELSE 1 END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimProduct (ProductCode, ProductName, ProductCategoryId, DesignId, MetalId)
SELECT TOP 100000
    'PRD-' + RIGHT('000000' + CAST(N AS VARCHAR(10)), 6),
    'Product ' + CAST(N AS VARCHAR(20)),
    (N % 200) + 1,
    (N % 50000) + 1,
    (N % 20) + 1
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimFactory (FactoryCode, FactoryName, CountryId, RegionId)
SELECT TOP 50
    'F' + RIGHT('00' + CAST(N AS VARCHAR(10)), 2),
    'Factory ' + CAST(N AS VARCHAR(20)),
    (N % 50) + 1,
    (N % 500) + 1
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimWorkCenter (WorkCenterCode, WorkCenterName, FactoryId)
SELECT TOP 500
    'WC-' + RIGHT('0000' + CAST(N AS VARCHAR(10)), 4),
    'WorkCenter ' + CAST(N AS VARCHAR(20)),
    (N % 50) + 1
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimMachine (MachineCode, WorkCenterId, ModelName, IsActive)
SELECT TOP 5000
    'MC-' + RIGHT('000000' + CAST(N AS VARCHAR(10)), 6),
    (N % 500) + 1,
    'Model ' + CAST((N % 250) + 1 AS VARCHAR(20)),
    CASE WHEN N % 20 = 0 THEN 0 ELSE 1 END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimShift (ShiftCode, ShiftName, StartHour, EndHour)
VALUES
('S1', 'Shift 1', 6, 14),
('S2', 'Shift 2', 14, 22),
('S3', 'Shift 3', 22, 6);
GO

INSERT INTO dbo.DimEmployee (EmployeeCode, EmployeeName, FactoryId, WorkCenterId, ShiftId, IsActive)
SELECT TOP 50000
    'EMP-' + RIGHT('000000' + CAST(N AS VARCHAR(10)), 6),
    'Employee ' + CAST(N AS VARCHAR(20)),
    (N % 50) + 1,
    (N % 500) + 1,
    (N % 3) + 1,
    CASE WHEN N % 40 = 0 THEN 0 ELSE 1 END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimProcessStep (StepCode, StepName, StepGroup)
SELECT TOP 60
    'STEP-' + RIGHT('000' + CAST(N AS VARCHAR(10)), 3),
    'Process Step ' + CAST(N AS VARCHAR(20)),
    CASE (N % 6)
        WHEN 0 THEN 'WAX'
        WHEN 1 THEN 'CAST'
        WHEN 2 THEN 'SET'
        WHEN 3 THEN 'POL'
        WHEN 4 THEN 'FIN'
        ELSE 'QC'
    END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimRouteTemplate (TemplateCode, TemplateName)
SELECT TOP 200
    'RT-' + RIGHT('0000' + CAST(N AS VARCHAR(10)), 4),
    'Route Template ' + CAST(N AS VARCHAR(20))
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.RouteTemplateStep (RouteTemplateId, ProcessStepId, StepSeq, TargetMinutes)
SELECT TOP 20000
    (N % 200) + 1,
    (N % 60) + 1,
    (N % 30) + 1,
    (ABS(CHECKSUM(NEWID())) % 120) + 5
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimWarehouse (WarehouseCode, WarehouseName, FactoryId)
SELECT TOP 200
    'WH-' + RIGHT('000' + CAST(N AS VARCHAR(10)), 3),
    'Warehouse ' + CAST(N AS VARCHAR(20)),
    (N % 50) + 1
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimLocation (LocationCode, WarehouseId, LocationType)
SELECT TOP 20000
    'LOC-' + RIGHT('000000' + CAST(N AS VARCHAR(10)), 6),
    (N % 200) + 1,
    CASE (N % 4)
        WHEN 0 THEN 'STORAGE'
        WHEN 1 THEN 'WIP'
        WHEN 2 THEN 'QC'
        ELSE 'SHIPPING'
    END
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimCarrier (CarrierCode, CarrierName)
SELECT TOP 50
    'CR-' + RIGHT('000' + CAST(N AS VARCHAR(10)), 3),
    'Carrier ' + CAST(N AS VARCHAR(20))
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimOrderStatus (StatusCode, StatusName)
VALUES
('NEW', 'New'),
('REL', 'Released'),
('WIP', 'In Progress'),
('HOLD', 'On Hold'),
('CMP', 'Completed'),
('CAN', 'Cancelled');
GO

INSERT INTO dbo.DimWipStatus (StatusCode, StatusName)
VALUES
('QUEUED', 'Queued'),
('RUN', 'Running'),
('WAIT', 'Waiting'),
('BLOCK', 'Blocked'),
('DONE', 'Done');
GO

INSERT INTO dbo.DimInventoryReason (ReasonCode, ReasonName)
VALUES
('RCV', 'Receiving'),
('ISS', 'Issue to WIP'),
('RET', 'Return to Stock'),
('ADJ', 'Adjustment'),
('SCR', 'Scrap');
GO

INSERT INTO dbo.DimUnitOfMeasure (UomCode, UomName)
VALUES
('EA', 'Each'),
('GM', 'Gram'),
('KG', 'Kilogram');
GO

INSERT INTO dbo.DimDefectCode (DefectCode, DefectName, Severity)
SELECT TOP 200
    'DEF-' + RIGHT('0000' + CAST(N AS VARCHAR(10)), 4),
    'Defect ' + CAST(N AS VARCHAR(20)),
    (N % 5) + 1
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimInspectionType (InspectionCode, InspectionName)
VALUES
('INP', 'Incoming'),
('IP', 'In Process'),
('FQC', 'Final Quality Check');
GO

INSERT INTO dbo.DimPriority (PriorityCode, PriorityName)
VALUES
('P1', 'Critical'),
('P2', 'High'),
('P3', 'Normal'),
('P4', 'Low');
GO

INSERT INTO dbo.DimStoneType (StoneCode, StoneName)
SELECT TOP 50
    'ST-' + RIGHT('000' + CAST(N AS VARCHAR(10)), 3),
    'Stone ' + CAST(N AS VARCHAR(20))
FROM #NumsSmall
ORDER BY N;
GO

INSERT INTO dbo.DimPackagingType (PackagingCode, PackagingName)
VALUES
('BOX', 'Box'),
('BAG', 'Bag'),
('WRAP', 'Wrap');
GO

INSERT INTO dbo.DimCalendar (CalendarDate, YearNo, MonthNo, DayNo, IsoWeek)
SELECT
    DATEADD(DAY, N - 1, CAST('2023-01-01' AS DATE)),
    YEAR(DATEADD(DAY, N - 1, CAST('2023-01-01' AS DATE))),
    MONTH(DATEADD(DAY, N - 1, CAST('2023-01-01' AS DATE))),
    DAY(DATEADD(DAY, N - 1, CAST('2023-01-01' AS DATE))),
    DATEPART(ISO_WEEK, DATEADD(DAY, N - 1, CAST('2023-01-01' AS DATE)))
FROM #NumsSmall
WHERE N <= 2000;
GO

INSERT INTO dbo.DimHoliday (CalendarDate, CountryId, HolidayName)
SELECT TOP 200
    DATEADD(DAY, N - 1, CAST('2023-01-01' AS DATE)),
    (N % 50) + 1,
    'Holiday ' + CAST(N AS VARCHAR(20))
FROM #NumsSmall
WHERE N <= 2000 AND N % 10 = 0
ORDER BY N;
GO

-- ============================================================
-- Extra fact / bridge seed data
-- ============================================================

INSERT INTO dbo.FactOrder
(
    OrderNo, CustomerId, ChannelId, PaymentTermId, PriorityId, OrderStatusId,
    OrderDate, DueDate, CurrencyId, OrderValue
)
SELECT
    'ORD-' + RIGHT('0000000000' + CAST(N AS VARCHAR(20)), 10),
    (N % 50000) + 1,
    (N % 10) + 1,
    (N % 12) + 1,
    (N % 4) + 1,
    (N % 6) + 1,
    DATEADD(DAY, -(N % 730), CAST('2025-01-01' AS DATE)),
    DATEADD(DAY, (N % 60), DATEADD(DAY, -(N % 730), CAST('2025-01-01' AS DATE))),
    (N % 25) + 1,
    CAST((ABS(CHECKSUM(NEWID())) % 500000) + 1000 AS DECIMAL(18,4)) / 10.0
FROM #NumsMed
ORDER BY N;
GO

-- ~5 lines per order on average (can be heavy; keep on #NumsBig but limited by TOP)
INSERT INTO dbo.FactOrderLine
(
    OrderId, [LineNo], ProductId, UomId, Qty, UnitPrice, DiscountPct, LineValue
)
SELECT
    ((N - 1) % (SELECT COUNT_BIG(*) FROM #NumsMed)) + 1,
    ((N - 1) % 10) + 1,
    (N % 100000) + 1,
    (N % 3) + 1,
    CAST((ABS(CHECKSUM(NEWID())) % 20) + 1 AS DECIMAL(18,4)),
    CAST((ABS(CHECKSUM(NEWID())) % 100000) + 100 AS DECIMAL(18,4)) / 10.0,
    CAST((N % 25) AS DECIMAL(9,4)) / 100.0,
    NULL
FROM #NumsBig
ORDER BY N;
GO

-- Compute line value after insert (cheap compared to generating inline)
UPDATE dbo.FactOrderLine
SET LineValue = Qty * UnitPrice * (1.0 - ISNULL(DiscountPct, 0));
GO

-- Map a subset of orders to BagNo keys (computed to match existing BagNo scheme)
INSERT INTO dbo.MapOrderBag (OrderId, BagNo)
SELECT
    N,
    CAST(23 + (N % 3) AS VARCHAR(10)) + '/' + CHAR(65 + (N % 10)) + '/' + CAST(N AS VARCHAR(20))
FROM #NumsMed
ORDER BY N;
GO

-- WIP events (large): use existing BagNo key shape and distribute across factories/steps
INSERT INTO dbo.FactWipEvent
(
    BagNo, OrderId, OrderLineId, FactoryId, WorkCenterId, MachineId,
    ProcessStepId, WipStatusId, EventTs, Qty, DurationSec
)
SELECT
    CAST(23 + (N % 3) AS VARCHAR(10)) + '/' + CHAR(65 + (N % 10)) + '/' + CAST(N AS VARCHAR(20)),
    ((N - 1) % (SELECT COUNT_BIG(*) FROM #NumsMed)) + 1,
    ((N - 1) % (SELECT COUNT_BIG(*) FROM #NumsBig)) + 1,
    (N % 50) + 1,
    (N % 500) + 1,
    (N % 5000) + 1,
    (N % 60) + 1,
    (N % 5) + 1,
    DATEADD(SECOND, N % 86400, DATEADD(DAY, -(N % 365), CAST('2025-01-01' AS DATETIME2(0)))),
    CAST((ABS(CHECKSUM(NEWID())) % 10) + 1 AS DECIMAL(18,4)),
    (ABS(CHECKSUM(NEWID())) % 3600) + 10
FROM #NumsBig
ORDER BY N;
GO

-- Inventory movements (medium)
INSERT INTO dbo.FactInventoryMovement
(
    ProductId, WarehouseId, LocationId, InventoryReasonId, MovementTs, QtyDelta, UomId
)
SELECT
    (N % 100000) + 1,
    (N % 200) + 1,
    (N % 20000) + 1,
    (N % 5) + 1,
    DATEADD(SECOND, N % 86400, DATEADD(DAY, -(N % 365), CAST('2025-01-01' AS DATETIME2(0)))),
    CAST(((ABS(CHECKSUM(NEWID())) % 200) - 100) AS DECIMAL(18,4)),
    (N % 3) + 1
FROM #NumsMed
ORDER BY N;
GO

-- Quality inspections (medium): link to BagNo and defects
INSERT INTO dbo.FactQualityInspection
(
    BagNo, InspectionTypeId, DefectCodeId, InspectorEmployeeId, InspectionTs, IsPass
)
SELECT
    CAST(23 + (N % 3) AS VARCHAR(10)) + '/' + CHAR(65 + (N % 10)) + '/' + CAST(N AS VARCHAR(20)),
    (N % 3) + 1,
    (N % 200) + 1,
    (N % 50000) + 1,
    DATEADD(SECOND, N % 86400, DATEADD(DAY, -(N % 365), CAST('2025-01-01' AS DATETIME2(0)))),
    CASE WHEN N % 10 = 0 THEN 0 ELSE 1 END
FROM #NumsMed
ORDER BY N;
GO

-- Shipments (small/medium)
INSERT INTO dbo.FactShipment (ShipmentNo, OrderId, CarrierId, WarehouseId, ShipDate, DeliveredDate)
SELECT
    'SHP-' + RIGHT('0000000000' + CAST(N AS VARCHAR(20)), 10),
    (N % (SELECT COUNT_BIG(*) FROM #NumsMed)) + 1,
    (N % 50) + 1,
    (N % 200) + 1,
    DATEADD(DAY, -(N % 365), CAST('2025-01-01' AS DATE)),
    DATEADD(DAY, (N % 10), DATEADD(DAY, -(N % 365), CAST('2025-01-01' AS DATE)))
FROM #NumsSmall
ORDER BY N;
GO

-- Returns (small)
INSERT INTO dbo.FactReturn (ReturnNo, OrderId, ReturnDate, Reason)
SELECT TOP (5000)
    'RET-' + RIGHT('0000000000' + CAST(N AS VARCHAR(20)), 10),
    (N % (SELECT COUNT_BIG(*) FROM #NumsMed)) + 1,
    DATEADD(DAY, -(N % 365), CAST('2025-01-01' AS DATE)),
    'Return reason ' + CAST(N AS VARCHAR(20))
FROM #NumsSmall
ORDER BY N;
GO

-- Product route mapping (small/medium)
INSERT INTO dbo.MapProductRouteTemplate (ProductId, RouteTemplateId, EffectiveFrom, EffectiveTo)
SELECT TOP (200000)
    (N % 100000) + 1,
    (N % 200) + 1,
    DATEADD(DAY, -(N % 365), CAST('2024-01-01' AS DATE)),
    NULL
FROM #NumsSmall
ORDER BY N;
GO

-- -------------------- main / fact tables ---------------------

-- We'll build BagNo as  YY/CHR/NNNN   (e.g. 25/A/1234)
-- OrdNo as  SO/NNNNN

-- IBM_Algov_PPC_Module_Data  (10 000 000 rows)
INSERT INTO dbo.IBM_Algov_PPC_Module_Data
(
    BYy, BChr, BNo, OdChr, OdPrdseq, Factory_PPC,
    BQty, DmPrdCtg,
    PPC, PPStartDt, ProdEndDt,
    ZCAD, ZCAM, ZMMD, ZPD, CPX,
    WSDiaIssue, Waxing, Waxsetting, WTR, JC,
    SprueGrinding, SRD_SPLIT, BPREP, Filing, PFMG,
    EP_HTCumEP, OTC_DISC_FINISHING, PrePolish, FanukSetting,
    MetalSetting, OS, Polish, FQC, GSI, Assaying, Packing,
    FilPPts, PolPPts, Bloc
)
SELECT
    -- BYy: year 23-25
    23 + (N % 3),
    -- BChr: single letter A-J
    CHAR(65 + (N % 10)),
    -- BNo: sequential
    N,
    -- OdChr
    CHAR(65 + (N % 10)),
    -- OdPrdseq: references IBM_ALgov_PPC_Prod_Seq.vPmcd
    'PS-' + RIGHT('0000' + CAST((N % 200) + 1 AS VARCHAR(10)),4),
    -- Factory_PPC
    CAST((N % 6) + 1 AS VARCHAR(10)),   -- values '1'..'6'
    -- BQty
    CAST((ABS(CHECKSUM(NEWID())) % 50) + 1 AS DECIMAL(18,4)),
    -- DmPrdCtg  (references Waxing_Pts.PPPrdCtg)
    'CTG-' + RIGHT('000' + CAST((N % 500) + 1 AS VARCHAR(10)),3),
    -- PPC
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    -- PPStartDt  (NOT NULL for WHERE clause)
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    -- ProdEndDt  (NOT NULL for WHERE clause)
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-06-01'),
    -- ZCAD..Packing  (DATE columns)
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    -- Missing date columns (FQC, GSI, Assaying, Packing)
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    -- FilPPts, PolPPts
    CAST(ABS(CHECKSUM(NEWID())) % 20 + 1 AS DECIMAL(18,4)) / 10.0,
    CAST(ABS(CHECKSUM(NEWID())) % 20 + 1 AS DECIMAL(18,4)) / 10.0,
    -- Bloc (references Process_Master.PMCd)
    'BL-' + RIGHT('0000' + CAST((N % 300) + 1 AS VARCHAR(10)),4)
FROM #NumsBig;
GO


-- IBM_Algov_Var_Source_1  (10 000 000 rows)
-- BagNo must match Module_Data:  BYy/BChr/BNo
INSERT INTO dbo.IBM_Algov_Var_Source_1
(
    OrdNo, BagNo, OdOmCmcd, [Factory Type], Oddmcd, DmPrdCtg,
    OrdSeq, OrdSeqDays, WIPQty, WsQty, HsQty, OdKt,
    circulationdt_01, Status, [Order Type], [Difficulty Level]
)
SELECT
    'SO/' + RIGHT('00000' + CAST(N AS VARCHAR(10)),5),
    -- BagNo = BYy/BChr/BNo  (must match IBM_Algov_PPC_Module_Data)
    CAST(23 + (N % 3) AS VARCHAR(10)) + '/'
        + CHAR(65 + (N % 10)) + '/'
        + CAST(N AS VARCHAR(20)),
    'CM-' + RIGHT('000' + CAST(N % 100 AS VARCHAR(10)),3),
    CASE N % 4
        WHEN 0 THEN 'Casting' WHEN 1 THEN 'Stamping'
        WHEN 2 THEN 'Handmade' ELSE 'Assembly'
    END,
    -- Oddmcd  (references Wax_Manpower.Drcd  &  CPX_Model_Flag.Dacd)
    'DM-' + RIGHT('00000' + CAST((N % 5000) + 1 AS VARCHAR(10)),5),
    -- DmPrdCtg
    'CTG-' + RIGHT('000' + CAST((N % 500) + 1 AS VARCHAR(10)),3),
    -- OrdSeq, OrdSeqDays
    (N % 20) + 1,
    (N % 30) + 1,
    -- WIPQty, WsQty, HsQty
    CAST((ABS(CHECKSUM(NEWID())) % 100) + 1 AS DECIMAL(18,4)),
    CAST((ABS(CHECKSUM(NEWID())) % 50)  + 1 AS DECIMAL(18,4)),
    CAST((ABS(CHECKSUM(NEWID())) % 50)  + 1 AS DECIMAL(18,4)),
    -- OdKt – mix of metal types to exercise the CASE
    CASE N % 7
        WHEN 0 THEN '14K'   WHEN 1 THEN '18K'  WHEN 2 THEN 'U14K'
        WHEN 3 THEN 'S925'  WHEN 4 THEN 'PT950' WHEN 5 THEN 'BRASS'
        ELSE '10K'
    END,
    -- circulationdt_01
    CONVERT(VARCHAR(10), DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'), 120),
    -- Status  (mostly 'A' so WHERE filter keeps most rows)
    CASE WHEN N % 20 = 0 THEN 'I' ELSE 'A' END,
    -- Order Type
    CASE WHEN N % 15 = 0 THEN 'Sample Order' ELSE 'Comm Order' END,
    -- Difficulty Level
    CASE N % 4
        WHEN 0 THEN 'Easy' WHEN 1 THEN 'Medium'
        WHEN 2 THEN 'Hard' ELSE 'Expert'
    END
FROM #NumsBig;
GO


-- IBM_Algov_Var_Source_MSF_1  (10 000 000 rows)
INSERT INTO dbo.IBM_Algov_Var_Source_MSF_1
(
    OrdNo, BagNo, OdOmCmcd, [FactoryType], Oddmcd, DmPrdCtg,
    OrdSeq, OrdSeqDays, WsQty, HsQty, OdKt,
    circulationdt_01, Status, [Order Type],
    Hand, Micro, Fanuk, [Fk-Micro], [Difficulty Level]
)
SELECT
    'SO/' + RIGHT('00000' + CAST(N AS VARCHAR(10)),5),
    CAST(23 + (N % 3) AS VARCHAR(10)) + '/'
        + CHAR(65 + (N % 10)) + '/'
        + CAST(N AS VARCHAR(20)),
    'CM-' + RIGHT('000' + CAST(N % 100 AS VARCHAR(10)),3),
    CASE N % 4
        WHEN 0 THEN 'Casting' WHEN 1 THEN 'Stamping'
        WHEN 2 THEN 'Handmade' ELSE 'Assembly'
    END,
    'DM-' + RIGHT('00000' + CAST((N % 5000) + 1 AS VARCHAR(10)),5),
    'CTG-' + RIGHT('000' + CAST((N % 500) + 1 AS VARCHAR(10)),3),
    (N % 20) + 1,
    (N % 30) + 1,
    CAST((ABS(CHECKSUM(NEWID())) % 50) + 1 AS DECIMAL(18,4)),
    CAST((ABS(CHECKSUM(NEWID())) % 50) + 1 AS DECIMAL(18,4)),
    CASE N % 7
        WHEN 0 THEN '14K'   WHEN 1 THEN '18K'  WHEN 2 THEN 'U14K'
        WHEN 3 THEN 'S925'  WHEN 4 THEN 'PT950' WHEN 5 THEN 'BRASS'
        ELSE '10K'
    END,
    CONVERT(VARCHAR(10), DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'), 120),
    CASE WHEN N % 20 = 0 THEN 'I' ELSE 'A' END,
    CASE WHEN N % 15 = 0 THEN 'Sample Order' ELSE 'Comm Order' END,
    -- Hand, Micro, Fanuk, Fk-Micro
    CAST(ABS(CHECKSUM(NEWID())) % 30 AS DECIMAL(18,4)),
    CAST(ABS(CHECKSUM(NEWID())) % 30 AS DECIMAL(18,4)),
    CAST(ABS(CHECKSUM(NEWID())) % 30 AS DECIMAL(18,4)),
    CAST(ABS(CHECKSUM(NEWID())) % 15 AS DECIMAL(18,4)),
    CASE N % 4
        WHEN 0 THEN 'Easy' WHEN 1 THEN 'Medium'
        WHEN 2 THEN 'Hard' ELSE 'Expert'
    END
FROM #NumsBig;
GO


-- IG_IBM_PPC_DiaDeviation  (10 000 000 rows)
-- OrdNo here is WITHOUT the 'SO/' prefix (the view does REPLACE(V.OrdNo,'SO/',''))
INSERT INTO dbo.IG_IBM_PPC_DiaDeviation (OrdNo, [Diamond Confirmation Date - Final])
SELECT
    RIGHT('00000' + CAST(N AS VARCHAR(10)),5),
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01')
FROM #NumsBig;
GO


-- ============================================================
-- 4.  INDEXES  (improve join performance)
-- ============================================================

CREATE NONCLUSTERED INDEX IX_Var1_BagNo_Status
    ON dbo.IBM_Algov_Var_Source_1 (BagNo, Status, [Order Type]);

CREATE NONCLUSTERED INDEX IX_MSF1_BagNo_Status
    ON dbo.IBM_Algov_Var_Source_MSF_1 (BagNo, Status, [Order Type]);

CREATE NONCLUSTERED INDEX IX_Module_BagKey
    ON dbo.IBM_Algov_PPC_Module_Data (BYy, BChr, BNo);

CREATE NONCLUSTERED INDEX IX_Module_PPDates
    ON dbo.IBM_Algov_PPC_Module_Data (PPStartDt, ProdEndDt);

CREATE NONCLUSTERED INDEX IX_ProdSeq_vPmcd
    ON dbo.IBM_ALgov_PPC_Prod_Seq (vPmcd);

CREATE NONCLUSTERED INDEX IX_DiaDev_OrdNo
    ON dbo.IG_IBM_PPC_DiaDeviation (OrdNo);

CREATE NONCLUSTERED INDEX IX_WaxMan_Drcd
    ON dbo.IBM_ALgov_Design_Wax_Manpower (Drcd);

CREATE NONCLUSTERED INDEX IX_ProcMaster_PMCd
    ON dbo.IBM_Algov_Process_Master (PMCd);

CREATE NONCLUSTERED INDEX IX_WaxPts_PPPrdCtg
    ON dbo.IBM_Algov_PPC_Waxing_Pts (PPPrdCtg);

CREATE NONCLUSTERED INDEX IX_CPXFlag_Dacd
    ON dbo.IBM_Algov_Design_CPX_Model_Flag (Dacd);

-- Extra indexes for perf testing schema
CREATE NONCLUSTERED INDEX IX_FactOrder_OrderNo
    ON dbo.FactOrder (OrderNo);

CREATE NONCLUSTERED INDEX IX_FactOrder_CustomerDate
    ON dbo.FactOrder (CustomerId, OrderDate) INCLUDE (OrderValue, OrderStatusId);

CREATE NONCLUSTERED INDEX IX_FactOrderLine_OrderId
    ON dbo.FactOrderLine (OrderId) INCLUDE (ProductId, Qty, LineValue);

CREATE NONCLUSTERED INDEX IX_FactOrderLine_ProductId
    ON dbo.FactOrderLine (ProductId) INCLUDE (OrderId, Qty, LineValue);

CREATE NONCLUSTERED INDEX IX_FactWipEvent_BagNo_EventTs
    ON dbo.FactWipEvent (BagNo, EventTs DESC) INCLUDE (WipStatusId, ProcessStepId, FactoryId, WorkCenterId, Qty);

CREATE NONCLUSTERED INDEX IX_FactWipEvent_FactoryStepDate
    ON dbo.FactWipEvent (FactoryId, ProcessStepId, EventTs);

CREATE NONCLUSTERED INDEX IX_FactInventoryMovement_ProductTs
    ON dbo.FactInventoryMovement (ProductId, MovementTs) INCLUDE (QtyDelta, WarehouseId, LocationId, InventoryReasonId);

CREATE NONCLUSTERED INDEX IX_FactQualityInspection_BagNoTs
    ON dbo.FactQualityInspection (BagNo, InspectionTs DESC) INCLUDE (IsPass, DefectCodeId, InspectionTypeId);

CREATE NONCLUSTERED INDEX IX_FactShipment_OrderId
    ON dbo.FactShipment (OrderId) INCLUDE (ShipDate, DeliveredDate, CarrierId);

CREATE NONCLUSTERED INDEX IX_MapOrderBag_BagNo
    ON dbo.MapOrderBag (BagNo) INCLUDE (OrderId);

CREATE NONCLUSTERED INDEX IX_MapProductRouteTemplate_ProductId
    ON dbo.MapProductRouteTemplate (ProductId) INCLUDE (RouteTemplateId, EffectiveFrom, EffectiveTo);

CREATE NONCLUSTERED INDEX IX_DimProduct_ProductCode
    ON dbo.DimProduct (ProductCode);

CREATE NONCLUSTERED INDEX IX_DimDesign_DesignCode
    ON dbo.DimDesign (DesignCode);

CREATE NONCLUSTERED INDEX IX_DimCustomer_CustomerCode
    ON dbo.DimCustomer (CustomerCode);

CREATE NONCLUSTERED INDEX IX_DimWorkCenter_FactoryId
    ON dbo.DimWorkCenter (FactoryId);

CREATE NONCLUSTERED INDEX IX_DimLocation_WarehouseId
    ON dbo.DimLocation (WarehouseId);
GO


-- ============================================================
-- 5.  CREATE THE VIEW
-- ============================================================

CREATE VIEW dbo.IBM_ALgov_PPC_Planning_Data
AS
WITH MSF AS
(
    SELECT
        BagNo,
        MAX(Hand)        AS Hand,
        MAX(Micro)       AS Micro,
        MAX(Fanuk)       AS Fanuk,
        MAX([Fk-Micro])  AS [Fk-Micro]
    FROM dbo.IBM_Algov_Var_Source_MSF_1
    WHERE LTRIM(RTRIM(Status)) = 'A'
      AND [Order Type] = 'Comm Order'
    GROUP BY BagNo
)

-- ================= FIRST SOURCE =================
SELECT DISTINCT
    V.OrdNo,
    M.[OdChr],
    V.BagNo,
    V.OdOmCmcd,
    V.[Factory Type],
    V.Oddmcd,
    V.DmPrdCtg,
    V.OrdSeq          AS [Prod Seq],
    V.OrdSeqDays      AS [ProdseqDays],
    M.odPrdseq,
    prod.vPDesc225    AS [Process Flow],
    V.DmPrdCtg        AS Design,

    V.WIPQty          AS BQty,
    V.WsQty,
    V.HsQty,

    CASE
        WHEN M.Factory_PPC IN ('1','2','3','4','5')
        THEN CONCAT('F', M.Factory_PPC)
        ELSE M.Factory_PPC
    END AS Factory,

    CASE
        WHEN V.OdKt LIKE 'U%K'  THEN 'Recycled Gold'
        WHEN V.OdKt LIKE '%K'   THEN 'Gold'
        WHEN V.OdKt LIKE 'S%'   THEN 'Silver'
        WHEN V.OdKt LIKE 'PT%'  THEN 'Platinum'
        WHEN V.OdKt = 'BRASS'   THEN 'Brass'
        ELSE 'Other'
    END AS Metal,

    V.OdKt,

    CONVERT(VARCHAR(10), ISNULL(M.PPC,          '1900-01-01'), 105) AS PPC,
    NULL AS SSettype,

    CONVERT(VARCHAR(10),
        ISNULL(TRY_CONVERT(DATE, V.circulationdt_01), '1900-01-01'), 105) AS circulationdt_01,

    CONVERT(VARCHAR(10), ISNULL(M.PPStartDt,     '1900-01-01'), 105) AS PPStartDt,
    CONVERT(VARCHAR(10), ISNULL(M.ProdEndDt,     '1900-01-01'), 105) AS ProdEndDt,

    flag.Model_Flag   AS Model_Availability,
    flag.CPX_Flag     AS CPX_Availability,

    CONVERT(VARCHAR(10),
        ISNULL(Dia.[Diamond Confirmation Date - Final], '1900-01-01'), 105) AS [Diamond Conf Dt],

    NULL AS Finding,

    CONVERT(VARCHAR(10), ISNULL(M.ZCAD,              '1900-01-01'), 105) AS ZCAD,
    CONVERT(VARCHAR(10), ISNULL(M.ZCAM,              '1900-01-01'), 105) AS ZCAM,
    CONVERT(VARCHAR(10), ISNULL(M.ZMMD,              '1900-01-01'), 105) AS ZMMD,
    CONVERT(VARCHAR(10), ISNULL(M.ZPD,               '1900-01-01'), 105) AS ZPD,

    CONVERT(VARCHAR(10), ISNULL(M.PPC,               '1900-01-01'), 105) AS PPC2,
    CONVERT(VARCHAR(10), ISNULL(M.CPX,               '1900-01-01'), 105) AS CPX,

    CONVERT(VARCHAR(10), ISNULL(M.WSDiaIssue,        '1900-01-01'), 105) AS WSDiaIssue,
    CONVERT(VARCHAR(10), ISNULL(M.Waxing,            '1900-01-01'), 105) AS Waxing,
    CONVERT(VARCHAR(10), ISNULL(M.Waxsetting,        '1900-01-01'), 105) AS Waxsetting,
    CONVERT(VARCHAR(10), ISNULL(M.WTR,               '1900-01-01'), 105) AS WTR,
    CONVERT(VARCHAR(10), ISNULL(M.JC,                '1900-01-01'), 105) AS JC,
    CONVERT(VARCHAR(10), ISNULL(M.SprueGrinding,     '1900-01-01'), 105) AS SprueGrinding,
    CONVERT(VARCHAR(10), ISNULL(M.SRD_SPLIT,         '1900-01-01'), 105) AS SRD_SPLIT,
    CONVERT(VARCHAR(10), ISNULL(M.BPREP,             '1900-01-01'), 105) AS BPREP,
    CONVERT(VARCHAR(10), ISNULL(M.Filing,            '1900-01-01'), 105) AS Filing,
    CONVERT(VARCHAR(10), ISNULL(M.PFMG,              '1900-01-01'), 105) AS PFMG,
    CONVERT(VARCHAR(10), ISNULL(M.EP_HTCumEP,        '1900-01-01'), 105) AS EP_HTCumEP,
    CONVERT(VARCHAR(10), ISNULL(M.OTC_DISC_FINISHING, '1900-01-01'), 105) AS OTC_DISC_FINISHING,
    CONVERT(VARCHAR(10), ISNULL(M.PrePolish,         '1900-01-01'), 105) AS PrePolish,
    CONVERT(VARCHAR(10), ISNULL(M.FanukSetting,      '1900-01-01'), 105) AS FanukSetting,
    CONVERT(VARCHAR(10), ISNULL(M.MetalSetting,      '1900-01-01'), 105) AS MetalSetting,
    CONVERT(VARCHAR(10), ISNULL(M.OS,                '1900-01-01'), 105) AS OS,
    CONVERT(VARCHAR(10), ISNULL(M.Polish,            '1900-01-01'), 105) AS Polish,
    CONVERT(VARCHAR(10), ISNULL(M.FQC,               '1900-01-01'), 105) AS FQC,
    CONVERT(VARCHAR(10), ISNULL(M.GSI,               '1900-01-01'), 105) AS GSI,
    CONVERT(VARCHAR(10), ISNULL(M.Assaying,          '1900-01-01'), 105) AS Assaying,
    CONVERT(VARCHAR(10), ISNULL(M.Packing,           '1900-01-01'), 105) AS Packing,
    process.PScd AS Bloc,

    COALESCE(W.PpPts, 0) * V.WIPQty     AS Waxing_Pts,
    COALESCE(wax.StoneSetPerHr, 0)       AS WaxStone,
    0                                    AS Hand,
    0                                    AS Micro,
    0                                    AS Fanuk,
    M.FilPPts * V.WIPQty                 AS FilPPts,
    M.PolPPts * V.WIPQty                 AS PolPPts,
    M.Bloc                               AS Bloc1,
    V.[Difficulty Level]                 AS [Design Difficulty]

FROM
(
    SELECT *
    FROM dbo.IBM_Algov_Var_Source_1
    WHERE LTRIM(RTRIM(Status)) = 'A'
      AND [Order Type] = 'Comm Order'
) V

LEFT JOIN dbo.IBM_Algov_PPC_Module_Data M
    ON V.BagNo = CAST(M.BYy AS VARCHAR(10)) + '/' + M.BChr + '/' + CAST(M.BNo AS VARCHAR(20))

LEFT JOIN dbo.IBM_ALgov_PPC_Prod_Seq prod
    ON M.OdPrdseq = prod.vPmcd

LEFT JOIN dbo.IG_IBM_PPC_DiaDeviation Dia
    ON REPLACE(V.OrdNo, 'SO/', '') = Dia.OrdNo

LEFT JOIN dbo.IBM_ALgov_Design_Wax_Manpower wax
    ON V.Oddmcd = wax.Drcd

LEFT JOIN dbo.IBM_Algov_Process_Master process
    ON M.Bloc = process.PMCd

LEFT JOIN dbo.IBM_Algov_PPC_Waxing_Pts W
    ON M.DmPrdCtg = W.PPPrdCtg

LEFT JOIN dbo.IBM_Algov_Design_CPX_Model_Flag flag
    ON V.Oddmcd = flag.Dacd

WHERE
    M.PPStartDt IS NOT NULL
    AND M.ProdEndDt IS NOT NULL

UNION

-- ================= SECOND SOURCE =================
SELECT DISTINCT
    V.OrdNo,
    M.[OdChr],
    V.BagNo,
    V.OdOmCmcd,
    V.[FactoryType],
    V.Oddmcd,
    V.DmPrdCtg,
    V.OrdSeq          AS [Prod Seq],
    V.OrdSeqDays      AS [ProdseqDays],
    M.odPrdseq,
    prod.vPDesc225    AS [Process Flow],
    V.DmPrdCtg        AS Design,

    M.BQty            AS BQty,
    V.WsQty,
    V.HsQty,

    CASE
        WHEN M.Factory_PPC IN ('1','2','3','4','5')
        THEN CONCAT('F', M.Factory_PPC)
        ELSE M.Factory_PPC
    END AS Factory,

    CASE
        WHEN V.OdKt LIKE 'U%K'  THEN 'Recycled Gold'
        WHEN V.OdKt LIKE '%K'   THEN 'Gold'
        WHEN V.OdKt LIKE 'S%'   THEN 'Silver'
        WHEN V.OdKt LIKE 'PT%'  THEN 'Platinum'
        WHEN V.OdKt = 'BRASS'   THEN 'Brass'
        ELSE 'Other'
    END AS Metal,

    V.OdKt,

    CONVERT(VARCHAR(10), ISNULL(M.PPC,          '1900-01-01'), 105) AS PPC,
    NULL AS SSettype,

    CONVERT(VARCHAR(10),
        ISNULL(TRY_CONVERT(DATE, V.circulationdt_01), '1900-01-01'), 105) AS circulationdt_01,

    CONVERT(VARCHAR(10), ISNULL(M.PPStartDt,     '1900-01-01'), 105) AS PPStartDt,
    CONVERT(VARCHAR(10), ISNULL(M.ProdEndDt,     '1900-01-01'), 105) AS ProdEndDt,

    flag.Model_flag   AS Model_Availability,
    flag.CPX_flag     AS CPX_Availability,

    CONVERT(VARCHAR(10),
        ISNULL(Dia.[Diamond Confirmation Date - Final], '1900-01-01'), 105) AS [Diamond Conf Dt],

    NULL AS Finding,

    CONVERT(VARCHAR(10), ISNULL(M.ZCAD,              '1900-01-01'), 105) AS ZCAD,
    CONVERT(VARCHAR(10), ISNULL(M.ZCAM,              '1900-01-01'), 105) AS ZCAM,
    CONVERT(VARCHAR(10), ISNULL(M.ZMMD,              '1900-01-01'), 105) AS ZMMD,
    CONVERT(VARCHAR(10), ISNULL(M.ZPD,               '1900-01-01'), 105) AS ZPD,

    CONVERT(VARCHAR(10), ISNULL(M.PPC,               '1900-01-01'), 105) AS PPC2,
    CONVERT(VARCHAR(10), ISNULL(M.CPX,               '1900-01-01'), 105) AS CPX,

    CONVERT(VARCHAR(10), ISNULL(M.WSDiaIssue,        '1900-01-01'), 105) AS WSDiaIssue,
    CONVERT(VARCHAR(10), ISNULL(M.Waxing,            '1900-01-01'), 105) AS Waxing,
    CONVERT(VARCHAR(10), ISNULL(M.Waxsetting,        '1900-01-01'), 105) AS Waxsetting,
    CONVERT(VARCHAR(10), ISNULL(M.WTR,               '1900-01-01'), 105) AS WTR,
    CONVERT(VARCHAR(10), ISNULL(M.JC,                '1900-01-01'), 105) AS JC,
    CONVERT(VARCHAR(10), ISNULL(M.SprueGrinding,     '1900-01-01'), 105) AS SprueGrinding,
    CONVERT(VARCHAR(10), ISNULL(M.SRD_SPLIT,         '1900-01-01'), 105) AS SRD_SPLIT,
    CONVERT(VARCHAR(10), ISNULL(M.BPREP,             '1900-01-01'), 105) AS BPREP,
    CONVERT(VARCHAR(10), ISNULL(M.Filing,            '1900-01-01'), 105) AS Filing,
    CONVERT(VARCHAR(10), ISNULL(M.PFMG,              '1900-01-01'), 105) AS PFMG,
    CONVERT(VARCHAR(10), ISNULL(M.EP_HTCumEP,        '1900-01-01'), 105) AS EP_HTCumEP,
    CONVERT(VARCHAR(10), ISNULL(M.OTC_DISC_FINISHING, '1900-01-01'), 105) AS OTC_DISC_FINISHING,
    CONVERT(VARCHAR(10), ISNULL(M.PrePolish,         '1900-01-01'), 105) AS PrePolish,
    CONVERT(VARCHAR(10), ISNULL(M.FanukSetting,      '1900-01-01'), 105) AS FanukSetting,
    CONVERT(VARCHAR(10), ISNULL(M.MetalSetting,      '1900-01-01'), 105) AS MetalSetting,
    CONVERT(VARCHAR(10), ISNULL(M.OS,                '1900-01-01'), 105) AS OS,
    CONVERT(VARCHAR(10), ISNULL(M.Polish,            '1900-01-01'), 105) AS Polish,
    CONVERT(VARCHAR(10), ISNULL(M.FQC,               '1900-01-01'), 105) AS FQC,
    CONVERT(VARCHAR(10), ISNULL(M.GSI,               '1900-01-01'), 105) AS GSI,
    CONVERT(VARCHAR(10), ISNULL(M.Assaying,          '1900-01-01'), 105) AS Assaying,
    CONVERT(VARCHAR(10), ISNULL(M.Packing,           '1900-01-01'), 105) AS Packing,

    process.PScd AS Bloc,

    COALESCE(W.PpPts, 0) * M.BQty                      AS Waxing_Pts,
    COALESCE(wax.StoneSetPerHr, 0)                      AS WaxStone,
    COALESCE(MSF.Hand, 0)                               AS Hand,
    COALESCE(MSF.Micro + MSF.[Fk-Micro], 0)            AS Micro,
    COALESCE(MSF.Fanuk, 0)                              AS Fanuk,
    M.FilPPts * M.BQty                                  AS FilPPts,
    M.PolPPts * M.BQty                                  AS PolPPts,
    M.Bloc                                              AS Bloc1,
    V.[Difficulty Level]                                AS [Design Difficulty]

FROM
(
    SELECT *
    FROM dbo.IBM_Algov_Var_Source_MSF_1
    WHERE LTRIM(RTRIM(Status)) = 'A'
      AND [Order Type] = 'Comm Order'
) V

LEFT JOIN MSF
    ON V.BagNo = MSF.BagNo

LEFT JOIN dbo.IBM_Algov_PPC_Module_Data M
    ON V.BagNo = CAST(M.BYy AS VARCHAR(10)) + '/' + M.BChr + '/' + CAST(M.BNo AS VARCHAR(20))

LEFT JOIN dbo.IBM_ALgov_PPC_Prod_Seq prod
    ON M.OdPrdseq = prod.vPmcd

LEFT JOIN dbo.IG_IBM_PPC_DiaDeviation Dia
    ON REPLACE(V.OrdNo, 'SO/', '') = Dia.OrdNo

LEFT JOIN dbo.IBM_ALgov_Design_Wax_Manpower wax
    ON V.Oddmcd = wax.Drcd

LEFT JOIN dbo.IBM_Algov_Process_Master process
    ON M.Bloc = process.PMCd

LEFT JOIN dbo.IBM_Algov_PPC_Waxing_Pts W
    ON M.DmPrdCtg = W.PPPrdCtg

LEFT JOIN dbo.IBM_Algov_Design_CPX_Model_Flag flag
    ON V.Oddmcd = flag.Dacd

WHERE
    M.PPStartDt IS NOT NULL
    AND M.ProdEndDt IS NOT NULL;
GO

-- ============================================================
-- 5b.  PERFORMANCE-FRIENDLY VIEWS (vPerf_*)
-- ============================================================

CREATE VIEW dbo.vPerf_OrderLifecycle
AS
SELECT
    o.OrderId,
    o.OrderNo,
    o.OrderDate,
    o.DueDate,
    s.StatusCode AS OrderStatus,
    c.CustomerCode,
    c.CustomerName,
    seg.SegmentCode,
    ch.ChannelCode,
    pt.PaymentTermCode,
    o.OrderValue
FROM dbo.FactOrder o
LEFT JOIN dbo.DimOrderStatus s ON o.OrderStatusId = s.OrderStatusId
LEFT JOIN dbo.DimCustomer c ON o.CustomerId = c.CustomerId
LEFT JOIN dbo.DimCustomerSegment seg ON c.SegmentId = seg.SegmentId
LEFT JOIN dbo.DimChannel ch ON o.ChannelId = ch.ChannelId
LEFT JOIN dbo.DimPaymentTerm pt ON o.PaymentTermId = pt.PaymentTermId;
GO

CREATE VIEW dbo.vPerf_OrderLinesEnriched
AS
SELECT
    ol.OrderLineId,
    ol.OrderId,
    ol.[LineNo],
    o.OrderNo,
    p.ProductCode,
    p.ProductName,
    cat.CategoryCode,
    d.DesignCode,
    m.MetalCode,
    ol.Qty,
    ol.UnitPrice,
    ol.DiscountPct,
    ol.LineValue
FROM dbo.FactOrderLine ol
LEFT JOIN dbo.FactOrder o ON ol.OrderId = o.OrderId
LEFT JOIN dbo.DimProduct p ON ol.ProductId = p.ProductId
LEFT JOIN dbo.DimProductCategory cat ON p.ProductCategoryId = cat.ProductCategoryId
LEFT JOIN dbo.DimDesign d ON p.DesignId = d.DesignId
LEFT JOIN dbo.DimMetal m ON p.MetalId = m.MetalId;
GO

CREATE VIEW dbo.vPerf_FactoryHierarchy
AS
SELECT
    f.FactoryId,
    f.FactoryCode,
    f.FactoryName,
    r.RegionCode,
    r.RegionName,
    co.CountryCode,
    co.CountryName
FROM dbo.DimFactory f
LEFT JOIN dbo.DimRegion r ON f.RegionId = r.RegionId
LEFT JOIN dbo.DimCountry co ON f.CountryId = co.CountryId;
GO

CREATE VIEW dbo.vPerf_WorkCenterMachines
AS
SELECT
    wc.WorkCenterId,
    wc.WorkCenterCode,
    wc.WorkCenterName,
    f.FactoryCode,
    mc.MachineId,
    mc.MachineCode,
    mc.ModelName,
    mc.IsActive
FROM dbo.DimWorkCenter wc
LEFT JOIN dbo.DimFactory f ON wc.FactoryId = f.FactoryId
LEFT JOIN dbo.DimMachine mc ON wc.WorkCenterId = mc.WorkCenterId;
GO

CREATE VIEW dbo.vPerf_WipEventsRecent30d
AS
SELECT
    e.WipEventId,
    e.BagNo,
    e.EventTs,
    ws.StatusCode AS WipStatus,
    ps.StepCode,
    f.FactoryCode,
    wc.WorkCenterCode,
    e.Qty,
    e.DurationSec
FROM dbo.FactWipEvent e
LEFT JOIN dbo.DimWipStatus ws ON e.WipStatusId = ws.WipStatusId
LEFT JOIN dbo.DimProcessStep ps ON e.ProcessStepId = ps.ProcessStepId
LEFT JOIN dbo.DimFactory f ON e.FactoryId = f.FactoryId
LEFT JOIN dbo.DimWorkCenter wc ON e.WorkCenterId = wc.WorkCenterId
WHERE e.EventTs >= DATEADD(DAY, -30, SYSUTCDATETIME());
GO

CREATE VIEW dbo.vPerf_WipLatestStatus
AS
WITH Ranked AS
(
    SELECT
        e.BagNo,
        e.EventTs,
        e.WipStatusId,
        e.ProcessStepId,
        e.FactoryId,
        e.WorkCenterId,
        ROW_NUMBER() OVER (PARTITION BY e.BagNo ORDER BY e.EventTs DESC, e.WipEventId DESC) AS rn
    FROM dbo.FactWipEvent e
)
SELECT
    r.BagNo,
    r.EventTs AS LatestEventTs,
    ws.StatusCode AS LatestWipStatus,
    ps.StepCode AS LatestStep,
    f.FactoryCode,
    wc.WorkCenterCode
FROM Ranked r
LEFT JOIN dbo.DimWipStatus ws ON r.WipStatusId = ws.WipStatusId
LEFT JOIN dbo.DimProcessStep ps ON r.ProcessStepId = ps.ProcessStepId
LEFT JOIN dbo.DimFactory f ON r.FactoryId = f.FactoryId
LEFT JOIN dbo.DimWorkCenter wc ON r.WorkCenterId = wc.WorkCenterId
WHERE r.rn = 1;
GO

CREATE VIEW dbo.vPerf_InventoryMovementsDaily
AS
SELECT
    CAST(m.MovementTs AS DATE) AS MovementDate,
    w.WarehouseCode,
    r.ReasonCode,
    SUM(m.QtyDelta) AS QtyDeltaSum,
    COUNT_BIG(*) AS MovementCount
FROM dbo.FactInventoryMovement m
LEFT JOIN dbo.DimWarehouse w ON m.WarehouseId = w.WarehouseId
LEFT JOIN dbo.DimInventoryReason r ON m.InventoryReasonId = r.InventoryReasonId
GROUP BY CAST(m.MovementTs AS DATE), w.WarehouseCode, r.ReasonCode;
GO

CREATE VIEW dbo.vPerf_QcFailRateDaily
AS
SELECT
    CAST(i.InspectionTs AS DATE) AS InspectDate,
    it.InspectionCode,
    SUM(CASE WHEN i.IsPass = 0 THEN 1 ELSE 0 END) AS FailCount,
    COUNT_BIG(*) AS TotalCount
FROM dbo.FactQualityInspection i
LEFT JOIN dbo.DimInspectionType it ON i.InspectionTypeId = it.InspectionTypeId
GROUP BY CAST(i.InspectionTs AS DATE), it.InspectionCode;
GO

CREATE VIEW dbo.vPerf_ShipmentOnTime
AS
SELECT
    s.ShipmentId,
    s.ShipmentNo,
    s.ShipDate,
    s.DeliveredDate,
    c.CarrierCode,
    o.OrderNo,
    CASE
        WHEN s.DeliveredDate IS NULL THEN NULL
        WHEN s.DeliveredDate <= DATEADD(DAY, 7, s.ShipDate) THEN 1
        ELSE 0
    END AS IsOnTime
FROM dbo.FactShipment s
LEFT JOIN dbo.DimCarrier c ON s.CarrierId = c.CarrierId
LEFT JOIN dbo.FactOrder o ON s.OrderId = o.OrderId;
GO

CREATE VIEW dbo.vPerf_ReturnsByReasonMonthly
AS
SELECT
    YEAR(r.ReturnDate) AS YearNo,
    MONTH(r.ReturnDate) AS MonthNo,
    COUNT_BIG(*) AS ReturnCount
FROM dbo.FactReturn r
GROUP BY YEAR(r.ReturnDate), MONTH(r.ReturnDate);
GO

CREATE VIEW dbo.vPerf_ProductRouteCoverage
AS
SELECT
    p.ProductCode,
    COUNT_BIG(DISTINCT pr.RouteTemplateId) AS RouteTemplateCount
FROM dbo.DimProduct p
LEFT JOIN dbo.MapProductRouteTemplate pr ON p.ProductId = pr.ProductId
GROUP BY p.ProductCode;
GO

CREATE VIEW dbo.vPerf_CustomerOrderValueMonthly
AS
SELECT
    c.CustomerCode,
    YEAR(o.OrderDate) AS YearNo,
    MONTH(o.OrderDate) AS MonthNo,
    SUM(o.OrderValue) AS OrderValueSum,
    COUNT_BIG(*) AS OrderCount
FROM dbo.FactOrder o
LEFT JOIN dbo.DimCustomer c ON o.CustomerId = c.CustomerId
GROUP BY c.CustomerCode, YEAR(o.OrderDate), MONTH(o.OrderDate);
GO

CREATE VIEW dbo.vPerf_DesignMetalAllowed
AS
SELECT
    d.DesignCode,
    m.MetalCode,
    dm.IsAllowed
FROM dbo.MapDesignMetal dm
LEFT JOIN dbo.DimDesign d ON dm.DesignId = d.DesignId
LEFT JOIN dbo.DimMetal m ON dm.MetalId = m.MetalId;
GO

CREATE VIEW dbo.vPerf_OrderBags
AS
SELECT
    o.OrderNo,
    b.BagNo
FROM dbo.MapOrderBag b
LEFT JOIN dbo.FactOrder o ON b.OrderId = o.OrderId;
GO

CREATE VIEW dbo.vPerf_BagWipWithPlanning
AS
SELECT
    p.BagNo,
    p.OrdNo,
    p.Factory,
    p.Metal,
    wl.LatestWipStatus,
    wl.LatestStep,
    wl.LatestEventTs
FROM dbo.IBM_ALgov_PPC_Planning_Data p
LEFT JOIN dbo.vPerf_WipLatestStatus wl ON p.BagNo = wl.BagNo;
GO

CREATE VIEW dbo.vPerf_WipStepThroughputDaily
AS
SELECT
    CAST(e.EventTs AS DATE) AS EventDate,
    ps.StepCode,
    COUNT_BIG(*) AS EventCount,
    SUM(ISNULL(e.Qty, 0)) AS QtySum,
    AVG(CAST(ISNULL(e.DurationSec, 0) AS FLOAT)) AS AvgDurationSec
FROM dbo.FactWipEvent e
LEFT JOIN dbo.DimProcessStep ps ON e.ProcessStepId = ps.ProcessStepId
GROUP BY CAST(e.EventTs AS DATE), ps.StepCode;
GO

CREATE VIEW dbo.vPerf_FactoryDailyThroughput
AS
SELECT
    CAST(e.EventTs AS DATE) AS EventDate,
    f.FactoryCode,
    COUNT_BIG(*) AS EventCount,
    SUM(ISNULL(e.Qty, 0)) AS QtySum
FROM dbo.FactWipEvent e
LEFT JOIN dbo.DimFactory f ON e.FactoryId = f.FactoryId
GROUP BY CAST(e.EventTs AS DATE), f.FactoryCode;
GO

CREATE VIEW dbo.vPerf_TopCustomersByValue
AS
SELECT TOP 100
    c.CustomerCode,
    SUM(o.OrderValue) AS TotalValue
FROM dbo.FactOrder o
LEFT JOIN dbo.DimCustomer c ON o.CustomerId = c.CustomerId
GROUP BY c.CustomerCode
ORDER BY SUM(o.OrderValue) DESC;
GO

CREATE VIEW dbo.vPerf_BottomCustomersByValue
AS
SELECT TOP 100
    c.CustomerCode,
    SUM(o.OrderValue) AS TotalValue
FROM dbo.FactOrder o
LEFT JOIN dbo.DimCustomer c ON o.CustomerId = c.CustomerId
GROUP BY c.CustomerCode
ORDER BY SUM(o.OrderValue) ASC;
GO

CREATE VIEW dbo.vPerf_WipAgingBuckets
AS
WITH FirstLast AS
(
    SELECT
        e.BagNo,
        MIN(e.EventTs) AS FirstEventTs,
        MAX(e.EventTs) AS LastEventTs
    FROM dbo.FactWipEvent e
    GROUP BY e.BagNo
)
SELECT
    BagNo,
    DATEDIFF(DAY, FirstEventTs, LastEventTs) AS AgeDays,
    CASE
        WHEN DATEDIFF(DAY, FirstEventTs, LastEventTs) < 1 THEN '0d'
        WHEN DATEDIFF(DAY, FirstEventTs, LastEventTs) < 3 THEN '1-2d'
        WHEN DATEDIFF(DAY, FirstEventTs, LastEventTs) < 7 THEN '3-6d'
        WHEN DATEDIFF(DAY, FirstEventTs, LastEventTs) < 14 THEN '7-13d'
        ELSE '14d+'
    END AS AgeBucket
FROM FirstLast;
GO

CREATE VIEW dbo.vPerf_CalendarCoverage
AS
SELECT
    c.YearNo,
    c.MonthNo,
    COUNT_BIG(*) AS DayCount,
    SUM(CASE WHEN h.HolidayId IS NULL THEN 0 ELSE 1 END) AS HolidayCount
FROM dbo.DimCalendar c
LEFT JOIN dbo.DimHoliday h ON c.CalendarDate = h.CalendarDate
GROUP BY c.YearNo, c.MonthNo;
GO

-- ============================================================
-- 5c.  INTENTIONALLY SLOW VIEWS (vSlow_*)
-- ============================================================

CREATE VIEW dbo.vSlow_PlanningUnionDistinct
AS
SELECT DISTINCT
    p.OrdNo,
    p.BagNo,
    p.Factory,
    p.Metal,
    p.[Process Flow],
    p.PPStartDt,
    p.ProdEndDt,
    wl.LatestWipStatus,
    wl.LatestStep
FROM dbo.IBM_ALgov_PPC_Planning_Data p
LEFT JOIN dbo.vPerf_WipLatestStatus wl ON p.BagNo = wl.BagNo
UNION
SELECT DISTINCT
    p.OrdNo,
    p.BagNo,
    p.Factory,
    p.Metal,
    p.[Process Flow],
    p.PPStartDt,
    p.ProdEndDt,
    wl.LatestWipStatus,
    wl.LatestStep
FROM dbo.IBM_ALgov_PPC_Planning_Data p
LEFT JOIN dbo.vPerf_WipLatestStatus wl ON p.BagNo = wl.BagNo;
GO

CREATE VIEW dbo.vSlow_ComputedKeyJoin
AS
SELECT
    v.OrdNo,
    v.BagNo,
    e.EventTs,
    e.Qty,
    ps.StepCode,
    ws.StatusCode
FROM dbo.IBM_Algov_Var_Source_1 v
LEFT JOIN dbo.FactWipEvent e
    ON REPLACE(v.BagNo, '/', '') = REPLACE(e.BagNo, '/', '')
LEFT JOIN dbo.DimProcessStep ps ON e.ProcessStepId = ps.ProcessStepId
LEFT JOIN dbo.DimWipStatus ws ON e.WipStatusId = ws.WipStatusId
WHERE LTRIM(RTRIM(v.Status)) = 'A'
  AND v.[Order Type] = 'Comm Order';
GO

CREATE VIEW dbo.vSlow_WipAggDistinct
AS
SELECT DISTINCT
    e.FactoryId,
    e.WorkCenterId,
    e.ProcessStepId,
    CAST(e.EventTs AS DATE) AS EventDate,
    SUM(ISNULL(e.Qty, 0)) OVER (PARTITION BY e.FactoryId, e.WorkCenterId, e.ProcessStepId, CAST(e.EventTs AS DATE)) AS QtySum,
    COUNT_BIG(*) OVER (PARTITION BY e.FactoryId, e.WorkCenterId, e.ProcessStepId, CAST(e.EventTs AS DATE)) AS EventCount
FROM dbo.FactWipEvent e;
GO

CREATE VIEW dbo.vSlow_OrderLineJoinExplode
AS
SELECT
    o.OrderNo,
    c.CustomerCode,
    ol.[LineNo],
    p.ProductCode,
    d.DesignCode,
    m.MetalCode,
    dm.IsAllowed,
    rt.TemplateCode,
    ps.StepCode
FROM dbo.FactOrder o
LEFT JOIN dbo.DimCustomer c ON o.CustomerId = c.CustomerId
LEFT JOIN dbo.FactOrderLine ol ON o.OrderId = ol.OrderId
LEFT JOIN dbo.DimProduct p ON ol.ProductId = p.ProductId
LEFT JOIN dbo.DimDesign d ON p.DesignId = d.DesignId
LEFT JOIN dbo.DimMetal m ON p.MetalId = m.MetalId
LEFT JOIN dbo.MapDesignMetal dm ON dm.DesignId = d.DesignId AND dm.MetalId = m.MetalId
LEFT JOIN dbo.MapProductRouteTemplate prt ON prt.ProductId = p.ProductId
LEFT JOIN dbo.DimRouteTemplate rt ON prt.RouteTemplateId = rt.RouteTemplateId
LEFT JOIN dbo.RouteTemplateStep rts ON rt.RouteTemplateId = rts.RouteTemplateId
LEFT JOIN dbo.DimProcessStep ps ON rts.ProcessStepId = ps.ProcessStepId;
GO

CREATE VIEW dbo.vSlow_InventoryMovementSkew
AS
SELECT
    p.ProductCode,
    w.WarehouseCode,
    l.LocationCode,
    r.ReasonCode,
    CAST(m.MovementTs AS DATE) AS MovementDate,
    SUM(m.QtyDelta) AS QtyDeltaSum
FROM dbo.FactInventoryMovement m
LEFT JOIN dbo.DimProduct p ON m.ProductId = p.ProductId
LEFT JOIN dbo.DimWarehouse w ON m.WarehouseId = w.WarehouseId
LEFT JOIN dbo.DimLocation l ON m.LocationId = l.LocationId
LEFT JOIN dbo.DimInventoryReason r ON m.InventoryReasonId = r.InventoryReasonId
GROUP BY
    p.ProductCode,
    w.WarehouseCode,
    l.LocationCode,
    r.ReasonCode,
    CAST(m.MovementTs AS DATE);
GO


-- ============================================================
-- 6.  CLEANUP temp table
-- ============================================================
EXEC sp_updatestats;
GO

DROP TABLE IF EXISTS #NumsSmall;
DROP TABLE IF EXISTS #NumsMed;
DROP TABLE IF EXISTS #NumsBig;
DROP TABLE IF EXISTS #NumsAll;
GO


-- ============================================================
-- 7.  QUICK SMOKE TEST
-- ============================================================
PRINT '=== Row counts per table ===';
SELECT 'IBM_Algov_Var_Source_1'          AS [Table], COUNT(*) AS [Rows] FROM dbo.IBM_Algov_Var_Source_1
UNION ALL
SELECT 'IBM_Algov_Var_Source_MSF_1',                 COUNT(*)          FROM dbo.IBM_Algov_Var_Source_MSF_1
UNION ALL
SELECT 'IBM_Algov_PPC_Module_Data',                  COUNT(*)          FROM dbo.IBM_Algov_PPC_Module_Data
UNION ALL
SELECT 'IBM_ALgov_PPC_Prod_Seq',                     COUNT(*)          FROM dbo.IBM_ALgov_PPC_Prod_Seq
UNION ALL
SELECT 'IG_IBM_PPC_DiaDeviation',                    COUNT(*)          FROM dbo.IG_IBM_PPC_DiaDeviation
UNION ALL
SELECT 'IBM_ALgov_Design_Wax_Manpower',              COUNT(*)          FROM dbo.IBM_ALgov_Design_Wax_Manpower
UNION ALL
SELECT 'IBM_Algov_Process_Master',                   COUNT(*)          FROM dbo.IBM_Algov_Process_Master
UNION ALL
SELECT 'IBM_Algov_PPC_Waxing_Pts',                   COUNT(*)          FROM dbo.IBM_Algov_PPC_Waxing_Pts
UNION ALL
SELECT 'IBM_Algov_Design_CPX_Model_Flag',            COUNT(*)          FROM dbo.IBM_Algov_Design_CPX_Model_Flag
UNION ALL
SELECT 'FactOrder',                                  COUNT(*)          FROM dbo.FactOrder
UNION ALL
SELECT 'FactOrderLine',                              COUNT(*)          FROM dbo.FactOrderLine
UNION ALL
SELECT 'FactWipEvent',                               COUNT(*)          FROM dbo.FactWipEvent
UNION ALL
SELECT 'FactInventoryMovement',                      COUNT(*)          FROM dbo.FactInventoryMovement
UNION ALL
SELECT 'FactQualityInspection',                      COUNT(*)          FROM dbo.FactQualityInspection
UNION ALL
SELECT 'FactShipment',                               COUNT(*)          FROM dbo.FactShipment
UNION ALL
SELECT 'FactReturn',                                 COUNT(*)          FROM dbo.FactReturn;

PRINT '';
PRINT '=== View smoke test (TOP 10) ===';
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

SELECT TOP 10 * FROM dbo.IBM_ALgov_PPC_Planning_Data;
SELECT TOP 10 * FROM dbo.vPerf_OrderLifecycle;
SELECT TOP 10 * FROM dbo.vPerf_WipLatestStatus;
SELECT TOP 10 * FROM dbo.vSlow_PlanningUnionDistinct;

SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;

PRINT 'Setup complete – database [intergold-dummy] is ready.';
GO
```

# Related

- [[intergold]]
- [[intergold-pp-optimization-cleanup]]
- [[mssql]]
