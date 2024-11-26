# Quiz Description

## Data Profiling
**Row Count:** 138<br>
**Column Count:** 22<br>
**Duplicate Records:** 0<br>
**Invalid Date Values:** Yes<br>
**Missing Values:** Yes<br>

---

<p align="center">
  <img src="https://github.com/IianWang/-/blob/master/image/%E5%9B%BE%E7%89%87%201.png" alt="Example Image">
</p>
<p align="center">
  <img src="https://github.com/IianWang/-/blob/master/image/%E5%9B%BE%E7%89%87%202.png" alt="Example Image">
</p>

<p align="center">Snapshot Processed</p>


## A1 (Process)

1. **Find Matching Field Names**
   Check the field names in sheet **"Report Data"** and find the ones that match the values in the **LABEL** column of sheet **"Database Fields"**. Record these matching fields.
2. **Analyzing and Inferring Naming Inconsistencies**
   For fields that do not match, analyze and infer their correspondence between the two sheets based on business logic.
   
   - Example: Determine that **[SOL Date]** in **"Report Data"** corresponds to **[Statue of Limitations]** in **"Database Fields"**.
3. **Add a Temporary Field in "Database Fields"**
   Add a new column **[LABEL-TEMP]** in sheet **"Database Fields"** to save the field names from **"Report Data"**.
   Write the guessed field names into this column (mark with **green shading**).
4. **Use a Formula to Map Fields**
   Use a formula in **"Report Data"** to map the fields based on the **[LABEL-TEMP]** column:
   
   ```excel
   =IFERROR(INDEX('Database Fields'!$A$2:$A$37, MATCH(A2, 'Database Fields'!$E$2:$E$37, 0)), "")
   ```

## A2 (Process)

1. **Clean the [Client Phone] Field**
   a) Extract pure numbers from the field content.
   b) Convert the extracted numbers into a numeric format.
2. **Standardize Date Fields**
   a) Convert the following fields to date format: **[DOB]**, **[SOL Date]**, **[Death Date (if deceased)]**, **[Closed Date]**.
   b) Format all these fields as **MM/DD/YYYY**.
   c) **Skip [Open Date] Field:**
   Since no correspondence was identified for **[Open Date]**, no modifications were made to this field.
3. **Modify the [Deceased?] Field**
   a) Replace all occurrences of `'y'` with **Deceased**.
   b) Fill any empty cells with **Alive**.
   c) Set up data validation, allowing only the options **Alive** or **Deceased** in this field.
4. **Format the [Contingency Fee (%)] Field**
   a) Convert the field values to percentages.
   b) Retain one decimal place for accuracy.


## The following is the code implementation,suitable for batch processing tasks.

```python
import pandas as pd
import numpy as np
from openpyxl import load_workbook
from openpyxl.worksheet.datavalidation import DataValidation

# Step 1: Clean the [Client Phone] Field
def clean_client_phone(df, column_name):
    """
    Extracts pure numbers from a phone number field and formats as numeric.
    """
    df[column_name] = df[column_name].astype(str).str.extract(r'(\d+)')  # Extract numeric parts
    df[column_name] = pd.to_numeric(df[column_name], errors='coerce')  # Convert to numeric
    return df

# Step 2: Standardize Date Fields
def standardize_date_fields(df, date_columns):
    """
    Converts specified fields to a date format and standardizes to MM/DD/YYYY.
    """
    for column in date_columns:
        df[column] = pd.to_datetime(df[column], errors='coerce')  # Convert to datetime
        df[column] = df[column].dt.strftime('%m/%d/%Y')  # Format as MM/DD/YYYY
    return df

# Step 3: Modify the [Deceased?] Field
def modify_deceased_field(df, column_name):
    """
    Replaces 'y' with 'Deceased', fills empty cells with 'Alive', and applies validation.
    """
    df[column_name] = df[column_name].replace('y', 'Deceased')  # Replace 'y' with 'Deceased'
    df[column_name] = df[column_name].fillna('Alive')  # Fill empty cells with 'Alive'
    return df

# Step 4: Format the [Contingency Fee (%)] Field
def format_contingency_fee(df, column_name):
    """
    Formats the field as a percentage with one decimal place.
    """
    df[column_name] = pd.to_numeric(df[column_name], errors='coerce')  # Convert to numeric
    df[column_name] = (df[column_name] * 100).round(1).astype(str) + '%'  # Format as percentage
    return df

# Step 5: Apply data validation
def add_data_validation(file_path, sheet_name, column_name, options):
    """
    Adds data validation (list mode) to a specified column in an Excel sheet.
    """
    wb = load_workbook(file_path)
    ws = wb[sheet_name]

    # Locate the column by its name in the first row (header)
    header_row = ws[1]
    column_letter = None

    for cell in header_row:
        if cell.value == column_name:
            column_letter = cell.column_letter
            break

    if column_letter is None:
        print(f"Column '{column_name}' not found in the sheet '{sheet_name}'!")
        return

    # Create the DataValidation object
    validation = DataValidation(
        type="list", 
        formula1=f'"{",".join(options)}"', 
        allow_blank=True
    )
    validation.prompt = f"Please select one of the following: {', '.join(options)}"
    validation.error = f"Invalid value! Please choose one of {', '.join(options)}"

    # Determine the range for validation
    max_row = ws.max_row
    if max_row < 2:
        print("No data rows found. Skipping validation.")
        return

    column_range = f"{column_letter}2:{column_letter}{max_row}"
    print(f"Adding validation to range: {column_range}")
    
    validation.add(column_range)

    # Add validation to the worksheet
    ws.add_data_validation(validation)
    wb.save(file_path)
    print(f"Data validation successfully added to column '{column_name}' in sheet '{sheet_name}'!")

# Example Usage
if __name__ == "__main__":
    # Sample DataFrame
    df = pd.read_excel('Test Case Dataset.xlsx', sheet_name='Report Data')

    # Apply functions
    df = clean_client_phone(df, "Client Phone")
    df = standardize_date_fields(df, ["DOB", "SOL Date", "Death Date (if deceased)", "Closed Date"])
    df = modify_deceased_field(df, "Deceased?")
    df = format_contingency_fee(df, "Contingency Fee (%)")
    save_excel_file = "TestCaseDatasetProcessed.xlsx"
    df.to_excel(save_excel_file, index=False)
    sheet = "Sheet1"
    column = "Deceased?"  # Field in the Excel sheet
    options = ["Alive", "Deceased"]  # List of valid options
    
    add_data_validation(save_excel_file, sheet, column, options)
```
