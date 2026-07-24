---
title: "NK Proteins SAP VBRK Queries"
date: 2026-07-24
tags:
  - nk-proteins
  - work
  - sap
publish: false
---

# NK Proteins SAP VBRK Queries

## Transaction Codes

- SE16N
- SE16

```
SELECT 
   
	 vbrk~belnr, 
	   
	 vbrk~fktyp, 
	   
	 vbrk~fkdat,
	 
	 vbrk~vkorg,
	 
	 vbrk~vtweg,
	 
	 vbrk~spart,
	 
	 vbrk~kunag

 FROM 
  
 vbrk
 
 WHERE vbrk~fkdat = '20250201'
 
 SELECT DISTINCT  
   
 vbrk~fktyp,
  
   
 FROM vbrk
 
 SELECT DISTINCT 
    vbrk~fktyp, 
    dd07t~ddtext AS fktyp_text
  FROM vbrk
  LEFT OUTER JOIN dd07t ON  dd07t~domname    = 'FKTYP'
                        AND dd07t~domvalue_l = vbrk~fktyp
                        AND dd07t~ddlanguage = @sy-langu
                        
                        
SELECT DISTINCT
  vbrk~belnr,
  vbrk~fktyp,
  dd07t~ddtext AS fktyp_text
  FROM vbrk
  LEFT OUTER JOIN dd07t ON  dd07t~domname    = 'FKTYP'
  AND dd07t~domvalue_l = vbrk~fktyp
  AND dd07t~ddlanguage = @sy-langu
  
  WHERE vbrk~fkdat = '20250201'
                        
SELECT
  vbrk~belnr,
  vbrk~fktyp,
  vbrk~vkorg,
  vbrk~vtweg,
  vbrk~spart,
  vbrk~kunag,
  COUNT( * ) AS row_count
FROM vbrk
WHERE vbrk~fkdat = '20250201'
GROUP BY 
  vbrk~belnr,
  vbrk~fktyp,
  vbrk~vkorg,
  vbrk~vtweg,
  vbrk~spart,
  vbrk~kunag
HAVING COUNT( * ) > 1
```

![[Pasted image 20260508173428.png]]

1600

1900000000

2024

1300

5105614808

2024

## Related

- [[nk-proteins-poc-validation]]
