import pandas as pd

# Load the Excel file
df = pd.read_excel(r"C:\Users\alexf\OneDrive\Documents\Pandas Tutorial\Customer Call List.xlsx")

# Drop duplicate rows
df = df.drop_duplicates()

# Drop the 'Not_Useful_Column'
df = df.drop(columns="Not_Useful_Column")

# Clean up 'Last_Name' column
df["Last_Name"] = df["Last_Name"].str.strip("123._/")

# Standardize phone numbers to the format 'XXX-XXX-XXXX'
df["Phone_Number"] = df["Phone_Number"].str.replace('[^0-9]', '', regex=True)
df["Phone_Number"] = df["Phone_Number"].apply(lambda x: f"{x[:3]}-{x[3:6]}-{x[6:10]}" if len(x) == 10 else '')

# Split 'Address' into 'Street_Address', 'State', and 'Zip_Code'
df[["Street_Address", "State", "Zip_Code"]] = df["Address"].str.split(',', 2, expand=True)

# Standardize 'Do_Not_Contact' values
df["Do_Not_Contact"] = df["Do_Not_Contact"].replace({'Yes': 'Y', 'No': 'N'})

# Fill NaN values with empty strings
df = df.fillna('')

# Drop rows where 'Do_Not_Contact' is 'Y'
df = df[df["Do_Not_Contact"] != 'Y']

# Reset the index after dropping rows
df.reset_index(drop=True, inplace=True)

# Display the cleaned DataFrame
print(df)
