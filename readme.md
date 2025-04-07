# Manage and retrieve patient custom fields via UI & DWH

## Overview

This guide describes retrieving patient custom field data from the User Interface (UI) and the Data Warehouse (DWH). 
It covers creating a patient custom field in the UI, filling in the field during patient creation or editing, linking the patient record to the custom field in the DWH, and finally querying the relevant tables (`dimpatientcustomfield`, `dimpatient`, and `lnkpatientcustomfieldpatient`) for the required data.

## Create a patient custom field

**Note!** The patient itself has been previously created through UI.

To create a patient custom field, follow the steps below:

1. In the **UI** navigate to 

    `Management > Application Settings > Custom Fields`.

2. **Add a new custom field**:

  - Click **Add Field**.
  
  - Enter a **Field Name** (e.g., “Test Patient Custom Field”).
        
  - In **Field Location**, select **Patient**.
        
  - Choose an appropriate type for **Field Type**(e.g., “Text”).
        
  - Optionally toggle **Pinned** to make the field appear prominently in the patient view.
        
  - Click **Save**.

   **Screenshot of the UI process for adding a field**:
   
   ![picture1](https://github.com/user-attachments/assets/8ed214b2-7bb9-4c44-9a0e-bbeb6e442442)


3. **Record created in the DWH**:

  - A new row is inserted into **`dimpatientcustomfield`**.
  
  - In this example, its **`sk_id`** is **`13`** (see the `dimpatientcustomfield.sk_id` column).

   **Screenshot of the `dimpatientcustomfield` table showing `sk_id=13`**:
   
   ![picture2](https://github.com/user-attachments/assets/355cbae2-f263-464e-968c-53485b06214f)


## Fill in the patient custom field via UI

1. In the **UI Control Panel** navigate to the

   `Patients > Create Patient` or `Patients > Edit Patient`

2. **Enter data for the newly created field**:

  - Find the field named “Test Patient Custom Field”.
  
  - Input the necessary value (e.g., “Test Value”).
  
  - Save the changes.

   **Screenshot of the UI showing the “Test Patient Custom Field”**:
   
   ![picture3](https://github.com/user-attachments/assets/e9f42515-7aac-4322-8c4f-ee272aadbea4)


3. **New or updated record in the DWH (`dimpatient` table)**:

  - When a new patient is created, a new row is inserted into **`dimpatient`**.  
  
  - In this example, the **`sk_id`** for this patient is **`331`**.

   **Screenshot of the `dimpatient` table showing `sk_id=331`**:
   
   ![picture4](https://github.com/user-attachments/assets/2abd745f-5163-43f2-a38b-6517aa4018c9)


## Link the custom field to the patient

When assigning (filling in) a custom field for a patient, the system automatically creates a “link” record in **`lnkpatientcustomfieldpatient`**. This record specifies which patient is tied to which custom field and stores the entered value.

  - **`lnkpatientcustomfieldpatient.sk_patient_custom_field_id`** -- references **`dimpatientcustomfield.sk_id`**.
      
  - **`lnkpatientcustomfieldpatient.sk_patient_id`** -- references **`dimpatient.sk_id`**.
      
  - **`lnkpatientcustomfieldpatient.custom_field_value`** -- stores the actual value entered (e.g., “Test Value”).

In the following example:

  - `sk_patient_custom_field_id = 13`. 
      
  - `sk_patient_id = 331`.
      
  - `custom_field_value = "Test Value"`.

**Screenshot of the `lnkpatientcustomfieldpatient` table showing the link record:**

![picture5](https://github.com/user-attachments/assets/5f05ea2a-6e32-480d-b59f-e24121936495)


**Screenshot of a diagram or table reference illustrating these relationships:**

![picture6](https://github.com/user-attachments/assets/98bdf7be-3051-4f92-a48b-aa191cbcf9c4)


## Retrieve custom field data via SQL

Below is an example SQL query that retrieves the patient details, custom field name, and the entered custom field value:

```sql

SELECT

    p.sk_id AS patient_sk_id,

    p.first_name AS patient_first_name,

    p.last_name AS patient_last_name,

    cf.sk_id AS custom_field_sk_id,

    cf.patient_custom_name AS custom_field_name,

    link.custom_field_value

FROM lnkpatientcustomfieldpatient link

JOIN dimpatient p

  ON link.sk_patient_id = p.sk_id

JOIN dimpatientcustomfield cf

  ON link.sk_patient_custom_field_id = cf.sk_id

WHERE p.sk_id = 331 -- The patient’s sk_id

  AND cf.sk_id = 13; -- The custom field’s sk_id

```


### Result columns

| **Column**            | **Description**                                                     |
|-----------------------|---------------------------------------------------------------------|
| `patient_sk_id`       | Surrogate key of the patient record in `dimpatient`.                |
| `patient_first_name`  | Patient’s first name.                                               |
| `patient_last_name`   | Patient’s last name.                                                |
| `custom_field_sk_id`  | Surrogate key of the custom field in `dimpatientcustomfield`.       |
| `custom_field_name`   | Name of the custom field (e.g., “Test Patient Custom Field”).       |
| `custom_field_value`  | The value entered in the UI (e.g., “Test Value”).                   |


### Data flow summary

1. **Add a custom field in the UI** -- Creates a record in `dimpatientcustomfield`.

2. **Create/Edit a patient and fill in the new custom field** -- Inserts (or updates) a record in `dimpatient`.

3. **Link the patient and the custom field** -- Creates a record in `lnkpatientcustomfieldpatient`, storing the entered value.

## Additional notes

### Archiving fields

Each of the tables (`dimpatientcustomfield`, `dimpatient`, and `lnkpatientcustomfieldpatient`) includes `archived`, `valid_from`, and `valid_to` columns, which track historical or archived data states.

### Multiple custom fields

A single patient can have multiple custom fields, each resulting in its row in `lnkpatientcustomfieldpatient`.

<details>
  <summary><strong>Disclaimer</strong></summary>
  
  **Note!** Disregard minor rendering peculiarities due to the specific GitHub Markdown preprocessor.
  
  Author's assumptions:

  - Data flow summary and Additional notes sections.
  - Go to Create/Edit Patient page. → navigate to `Patients > Create Patient` or `Patients > Edit Patient`.
  
  Style guide, voice, and tone:
  - Referred to the Google Developer Documentation.
  - Used Active Voice in most cases but only for minor exceptions.
  - Only the first letter is capitalized in headings, except for specific acronyms.
  - No pronouns.
  
  Documentation example:
  - [Query transactional data](https://docs.stripe.com/stripe-data/query-transactions)
</details>
