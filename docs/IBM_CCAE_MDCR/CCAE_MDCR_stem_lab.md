---
layout: default
title: Lab
nav_order: 7
grand_parent: IBM CCAE & MDCR
parent: IBM CCAE & MDCR to STEM
description: "LAB to STEM table description"
---

## Table name: stem_table

The STEM table is a staging area where source codes like ICD9 codes will first be mapped to concept_ids. The STEM table itself is an amalgamation of the OMOP event tables to facilitate record movement. This means that all fields present across the OMOP event tables are present in the STEM table. After a record is mapped and staged, the domain of the concept_id dictates which OMOP table (Condition_occurrence, Drug_exposure, Procedure_occurrence, Measurement, Observation, Device_exposure) the record will move to. Please see the STEM -> CDM mapping files for a description of which STEM fields move to which STEM tables.

### Key conventions

* VISIT_DETAIL must be built before STEM (refer to [VISIT_DETAIL file](https://ohdsi.github.io/ETL-LambdaBuilder/IBM_CCAE_MDCR/CCAE_MDCR_visit_detail.html))
  
* Referential integrity is maintained with VISIT_DETAIL.
For every record in STEM there should be 1 row record in VISIT_DETAIL (n:1 join).

* For every record in VISIT_DETAIL there may be 0 to n rows in STEM.

* Units are mapped to UNIT_CONCEPT_IDs in the OMOP **VOCABULARY** (VOCABULARY_ID = UCUM - Unified Code for Units of Measure).  Please note that mapping a UNIT_SOURCE_VALUE to a UNIT_CONCEPT_ID is both case sensitive and accent sensitive.

* Lab result in **LAB** is stored in three fields: ABNORMAL, RESULT (numeric) and RESLTCAT (character). Numeric results can be in both RESULT and RESLTCAT. RESULT usually has the following values if the lab result is string: 0 or large negative value (<-999999.99999).  ABNORMAL is the abnormal indicator set by the lab vendors: ‘A’ means “abnormal”, ‘N’ means “normal”, ‘H’ means “Above the normal range”, ‘L’ means “Below the normal range”, ‘+’ means “Positive” and ‘-’ means “Negative”.  
* Use the following to set VALUE_AS_STRING and VALUE_AS_CONCEPT_ID:<br>
```sql
/*Result as string*/
VALUE_AS_STRING = CATS(RESLTCAT);
/*Result as concept code*/
IF UPCASE(VALUE_AS_STRING) ='LOW' OR ABNORMAL ='L' 	
    THEN VALUE_AS_CONCEPT_ID = 4267416;
ELSE IF UPCASE(VALUE_AS_STRING) ='HIG' OR ABNORMAL ='H' 
    THEN VALUE_AS_CONCEPT_ID =4328749;
ELSE IF UPCASE(VALUE_AS_STRING) ='NRM' OR ABNORMAL ='N' 
    THEN VALUE_AS_CONCEPT_ID =4069590;
ELSE IF UPCASE(VALUE_AS_STRING) ='ABN' OR ABNORMAL ='A' 
    THEN VALUE_AS_CONCEPT_ID =4135493;
ELSE IF UPCASE(VALUE_AS_STRING) ='ABS' 
    THEN VALUE_AS_CONCEPT_ID =4132135;
ELSE IF UPCASE(VALUE_AS_STRING) ='PRS' 
    THEN VALUE_AS_CONCEPT_ID =4181412;
ELSE IF UPCASE(VALUE_AS_STRING) ='POS' OR ABNORMAL ='+' 
    THEN VALUE_AS_CONCEPT_ID =9191;
ELSE IF UPCASE(VALUE_AS_STRING) ='NEG' OR ABNORMAL ='-' 
    THEN VALUE_AS_CONCEPT_ID =9189;
ELSE IF UPCASE(VALUE_AS_STRING) IN ('FIN','FIR') 
    THEN VALUE_AS_CONCEPT_ID =9188;
ELSE IF UPCASE(VALUE_AS_STRING) ='NON' 
    THEN VALUE_AS_CONCEPT_ID =9190;
ELSE IF UPCASE(VALUE_AS_STRING) ='TRA' 
    THEN VALUE_AS_CONCEPT_ID = 9192;
IF RESULT NE 0 AND RESULT > -999999.99999 THEN DO;
/*Result as number*/
    VALUE_AS_NUMBER = RESULT;END;
```


### Reading from **LAB**

![](images/image8.png)

| Destination Field | Source field | Logic | Comment field |
| --- | --- | --- | --- |
| DOMAIN_ID | - | - | This should be the domain_id of the standard concept in the CONCEPT_ID field. If a code is mapped to CONCEPT_ID 0, put the domain_id as Observation |
| PERSON_ID | ENROLID | - | - |
| VISIT_OCCURRENCE_ID | **VISIT_DETAIL**<br>VISIT_OCCURRENCE_ID | - | - |
| VISIT_DETAIL_ID | **VISIT_DETAIL**<br>VISIT_DETAIL_ID | - | - |
| PROVIDER_ID | **VISIT_DETAIL**<br>PROVIDER_ID | - | - |
| ID | - | System generated. | - |
| CONCEPT_ID | LOINCCD | Use the <a href="https://ohdsi.github.io/CommonDataModel/sqlScripts.html">Source-to-Standard Query</a>.<br><br> `WHERE SOURCE_VOCABULARY_ID IN ('LOINC')`<br>`AND TARGET_STANDARD_CONCEPT = 'S'`<br>`AND TARGET_INVALID_REASON IS NULL` | - |
| SOURCE_VALUE | LOINCCD | The LOINCCD as it appears in the **LAB** table | - |
| SOURCE_CONCEPT_ID | LOINCCD | Use the <a href="https://ohdsi.github.io/CommonDataModel/sqlScripts.html">Source-to-Source Query</a>.<br><br> `WHERE SOURCE_VOCABULARY_ID IN (‘LOINC’)`<br>`AND TARGET_VOCABULARY_ID IN (‘LOINC’)` | - |
| TYPE_CONCEPT_ID | - | All rows will have CONCEPT_ID `32856` | `32856` = 'Lab' |
| START_DATE | **VISIT_DETAIL**<br>VISIT_DETAIL_START_DATE | - | - |
| START_DATETIME | - | START_DATE + Midnight | - |
| END_DATE | - | NULL | - |
| END_DATETIME | - | NULL | - |
| VERBATIM_END_DATE | - | NULL | - |
| DAYS_SUPPLY | - | NULL | - |
| DOSE_UNIT_SOURCE_VALUE | - | NULL | - |
| LOT_NUMBER | - | NULL | - |
| MODIFIER_CONCEPT_ID | - | NULL | - |
| MODIFIER_SOURCE_VALUE | - | NULL | - |
| OPERATOR_CONCEPT_ID | - | 0 | - |
| QUANTITY | - | NULL | - |
| RANGE_HIGH | REFHIGH | - | - |
| RANGE_LOW | REFLOW | - | - |
| REFILLS | - | NULL | - |
| ROUTE_CONCEPT_ID | - | 0 | - |
| ROUTE_SOURCE_VALUE | - | NULL | - |
| SIG | - | NULL | - |
| STOP_REASON | - | NULL | - |
| UNIQUE_DEVICE_ID | - | NULL | - |
| UNIT_CONCEPT_ID | RESUNIT | Use the <a href="https://ohdsi.github.io/CommonDataModel/sqlScripts.html">Source-to-Standard Query</a>.<br><br> Filters:<br>`WHERE SOURCE_VOCABULARY_ID IN ('UCUM')`<br>  `AND TARGET_VOCABULARY_ID IN ('UCUM')`<br>`AND TARGET_INVALID_REASON IS NULL`<br><br>If you do not get a map from UCUM use the JNJ_UNIT vocabulary. | - |
| UNIT_SOURCE_VALUE | RESUNIT | RESUNIT as it appears in the **LAB** table | - |
| VALUE_AS_CONCEPT_ID | RESLTCAT | Refer to logic above for defining this field. | - |
| VALUE_AS_NUMBER | RESULT<br>RESLTCAT | Refer to logic above for defining this field. <br><br>For the following LOINCs (3142-7, 29463-7, 3141-9) if the RESULT > 100000 and the last digits are 0000 and RESUNIT = ‘LBS’, trim the last for digits 0000. | - |
| VALUE_AS_STRING | RESULT<br>RESLTCAT | Refer to logic above for defining this field. | - |
| VALUE_SOURCE_VALUE | RESLTCAT | RESLTCAT as it appears in the **LAB** table. | - |
| ANATOMIC_SITE_CONCEPT_ID | - | 0 | - |
| DISEASE_STATUS_CONCEPT_ID | - | 0 | - |
| SPECIMIN_SOURCE_ID | - | NULL | - |
| ANATOMIC_SITE_SOURCE_VALUE | - | NULL | - |
| DISEASE_STATUS_SOURCE_VALUE | - | NULL | - |
| CONDITION_STATUS_CONCEPT_ID | - | 0 | - |
| CONDITION_STATUS_SOURCE_VALUE | - | NULL | - |
| EVENT_ID | - | NULL | - |
| EVENT_FIELD_CONCEPT_ID | - | 0 | - |
| VALUE_AS_DATETIME | - | NULL | - |
| QUALIFIER_CONCEPT_ID | - | 0 | - |
| QUALIFIER_SOURCE_VALUE | - | NULL | - |

## Change Log

### June 9, 2021
* Updated type concept

* Removed the following logic:
  * LOINCs
      * Valid LOINC codes have the following layouts #-#, ##-#, ###-#, ####-#, and #####-# .
      * When mapping to valid LOINCs in the OMOP Vocabulary there are a few invalid LOINC codes.  Implementing a check for the second to last character is a ‘-‘ensures you pull a valid LOINC from the **VOCABULARY**.  
  * Only use records with SVCDATE that fall within an OBSERVATION_PERIOD available for this person.