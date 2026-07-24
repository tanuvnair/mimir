---
title: "MDX Queries"
date: 2026-07-24
tags:
  - database
  - qubefini
  - tm1
  - work
publish: false
---

# MDX Queries

Qubefini / HUL TM1 MDX snippets. Related: [[qubefini]], [[planning-analytics]], [[scheduler-sql-snippets]]

## Basepack State Level Input (cashup + history)

```sql
SELECT NON EMPTY
   {
      FILTER(
         TM1SUBSETALL([Month].[Month]) , (
            [Month].[Month].CURRENTMEMBER.PROPERTIES("at_UDL_Month") = "YES"))
   } ON 0, NON EMPTY
   {
      {
         FILTER(
            TM1SubsetAll([Basepack_Final].[Basepack_Final]), (
               [Basepack_Final].[Basepack_Final].CURRENTMEMBER.PROPERTIES("LIBI_Cashup") = "Yes"))
      }
   }*{
      {
         FILTER(
            TM1SubsetAll([State].[State]), (
               [State].[State].CURRENTMEMBER.PROPERTIES("LIBI_History") = "Yes"))
      }
   }*{
      [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[MRP_per_piece],
      [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[Grammage_per_piece]
   } ON 1
FROM
   [Basepack_State_Level_Input]
WHERE (
   [Elist].[Elist].[Elist],
   [Data_Source].[Data_Source].[Trading Trend],
   [Version].[Version].[CV],
   [Year].[Year].[Current_Year])
```

## Basepack State Level Input (HCPC descendants)

```sql
SELECT
{} ON 0, NON EMPTY
{
  FILTER(
    DISTINCT(
      DESCENDANTS([Basepack_Final].[Basepack_Final].[HCPC])
    ),
    ([Basepack_Final].[Basepack_Final].CURRENTMEMBER.PROPERTIES("LIBI_History") = "Yes")
  )
}*{
  FILTER(
    TM1SubsetAll([State].[State]),
    ([State].[State].CURRENTMEMBER.PROPERTIES("LIBI_History") = "Yes")
  )
}*{
  FILTER(
    TM1SubsetAll([Month].[Month]),
    ([Month].[Month].CURRENTMEMBER.PROPERTIES("at_UDL_Month") = "Yes")
  )
}*{
  [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[MRP_per_piece],
  [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[Grammage_per_piece]
} ON 1
FROM [Basepack_State_Level_Input]
WHERE (
  [Elist].[Elist].[Elist],
  [Data_Source].[Data_Source].[Trading Trend],
  [Version].[Version].[CV],
  [Year].[Year].[Current_Year]
)
```

## HUL_HCPC_New

### Hardcoded Nov-CY

```sql
SELECT  
   {  
   } ON 0, NON EMPTY  
   {  
      FILTER(  
         TM1SUBSETALL([Basepack_Final].[Basepack_Final]) , (  
            [Basepack_Final].[Basepack_Final].CURRENTMEMBER.PROPERTIES("LIBI_Cashup") = "Yes"))  
   }*{  
      FILTER(  
         TM1SUBSETALL([State].[State]) , (  
            [State].[State].CURRENTMEMBER.PROPERTIES("LIBI_History") = "Yes"))  
   }*{  
      [Month].[Month].[Nov-CY]  
   }*{  
      [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[Grammage_per_piece],  
      [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[MRP_per_piece]  
   } ON 1  
FROM  
   [Basepack_State_Level_Input]  
WHERE (  
   [Elist].[Elist].[Elist],  
   [Data_Source].[Data_Source].[Trading Trend],  
   [Version].[Version].[CV],  
   [Year].[Year].[Current_Year])
```

### Getting Month by Attribute

```SQL
SELECT   
   {  
   } ON 0, NON EMPTY   
   {  
      {  
         FILTER(  
            TM1SubsetAll([Basepack_Final].[Basepack_Final]), (  
               [Basepack_Final].[Basepack_Final].CURRENTMEMBER.PROPERTIES("LIBI_Cashup") = "Yes"))  
      }  
   }*{  
      {  
         FILTER(  
            TM1SubsetAll([State].[State]), (  
               [State].[State].CURRENTMEMBER.PROPERTIES("LIBI_History") = "Yes"))  
      }  
   }*{  
      {  
         FILTER(  
            TM1SubsetAll([Month].[Month]), (  
               [Month].[Month].CURRENTMEMBER.PROPERTIES("at_UDL_Month") = "Yes"))  
      }  
   }*{  
      [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[Grammage_per_piece],  
      [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[MRP_per_piece]  
   } ON 1   
FROM  
   [Basepack_State_Level_Input]   
WHERE (  
   [Elist].[Elist].[Elist],   
   [Data_Source].[Data_Source].[Trading Trend],   
   [Version].[Version].[CV],   
   [Year].[Year].[Current_Year])
```

```SQL
SELECT 
   {
   } ON 0, NON EMPTY 
   {
      FILTER(
         DISTINCT(
            DESCENDANTS(
               [Basepack_Final].[Basepack_Final].[HCPC])) , (
            [Basepack_Final].[Basepack_Final].CURRENTMEMBER.PROPERTIES("LIBI_History") = "Yes"))
   }*{
      {
         FILTER(
            TM1SubsetAll([State].[State]), (
               [State].[State].CURRENTMEMBER.PROPERTIES("LIBI_History") = "Yes"))
      }
   }*{
      {
         FILTER(
            TM1SubsetAll([Month].[Month]), (
               [Month].[Month].CURRENTMEMBER.PROPERTIES("at_UDL_Month") = "Yes"))
      }
   }*{
      [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[MRP_per_piece],
      [Basepack_State_Level_Input_M].[Basepack_State_Level_Input_M].[Grammage_per_piece]
   } ON 1 
FROM
   [Basepack_State_Level_Input] 
WHERE (
   [Elist].[Elist].[Elist], 
   [Data_Source].[Data_Source].[Trading Trend], 
   [Version].[Version].[CV], 
   [Year].[Year].[Current_Year])
```

```SQL
SELECT 
    {
FILTER(
    TM1SubsetAll([Month].[Month]),
    [Month].[Month].CurrentMember.Properties("ML_Cashup") = "YES"
)

    } ON COLUMNS,

    (
        {
            FILTER(
                TM1SubsetAll([Basepack_Final].[Basepack_Final]),
                [Basepack_Final].[Basepack_Final].CurrentMember.Properties("at_Brand_ML_Push") = "YES"
            )
        }
        *
        {
            FILTER(
                TM1SubsetAll([State].[State]),
                [State].[State].CurrentMember.Properties("at_ML_FC_Total_Region") = "YES"
            )
        }
        *
        {
            FILTER(
                TM1SubsetAll([Channels].[Channels]),
                [Channels].[Channels].CurrentMember.Properties("at_ML_FC_Extract_flag") = "Yes"
            )
        }
        *
        {
            FILTER(
                TM1SubsetAll([Cashup_M].[Cashup_M]),
                [Cashup_M].[Cashup_M].CurrentMember.Properties("at_ML_FC_Extract_flag") = "Yes"
            )
        }
    ) ON ROWS

FROM 
    [Category_Cashup]

WHERE 
    (
        [Data_Source].[Data_Source].[Trading Trend],
        [Version].[Version].[CV],
        [Elist].[Elist].[Elist],
        [Year].[Year].[Current_Year]
    );
```

## Related

- [[qubefini]]
- [[planning-analytics]]
- [[scheduler-sql-snippets]]
