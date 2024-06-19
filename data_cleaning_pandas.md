# Import necessary libraries
import kaggle
import zipfile
import pandas as pd
import sqlalchemy as sal

# Step 1: Download dataset from Kaggle
# !pip install kaggle  # Install Kaggle API if not already installed
# import kaggle  # Import Kaggle library (not necessary since we're using command-line)

# Download dataset 'orders.csv' from Kaggle
!kaggle datasets download ankitbansal06/retail-orders -f orders.csv

# Step 2: Extract file from zip archive
zip_ref = zipfile.ZipFile('orders.csv.zip') 
zip_ref.extractall()  # Extract file to current directory
zip_ref.close()  # Close zip file

# Step 3: Read data from CSV file and handle null values
df = pd.read_csv('orders.csv', na_values=['Not Available', 'unknown'])

# Step 4: Rename columns to lowercase and replace spaces with underscores
# df.rename(columns={'Order Id':'order_id', 'City':'city'}, inplace=True)
df.columns = df.columns.str.lower().str.replace(' ', '_')  # Lowercase and replace spaces
df.head(5)  # Display first 5 rows

# Step 5: Derive new columns 'profit' from existing columns
df['profit'] = df['sale_price'] - df['cost_price']

# Step 6: Convert 'order_date' column from object data type to datetime
df['order_date'] = pd.to_datetime(df['order_date'], format="%Y-%m-%d")

# Step 7: Drop unnecessary columns 'list_price', 'cost_price', 'discount_percent'
df.drop(columns=['list_price', 'cost_price', 'discount_percent'], inplace=True)

# Step 8: Load the data into SQL Server using SQLAlchemy
# Create SQL Alchemy Engine
engine = sal.create_engine('mssql://ANKIT\SQLEXPRESS/master?driver=ODBC+DRIVER+17+FOR+SQL+SERVER')
conn = engine.connect()

# Step 9: Load the data into SQL Server table 'df_orders' with 'append' option
df.to_sql('df_orders', con=conn, index=False, if_exists='append')
