# Import necessary libraries
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Load the dataset
df = pd.read_csv(r"C:\Users\alexf\OneDrive\Documents\Pandas Tutorial\world_population.csv")

# Display the dataframe
df

# Set pandas options to display float values to two decimal places
pd.set_option('display.float_format', lambda x: '%.2f' % x)

# Get information about the dataframe
df.info()

# Describe the dataframe to get summary statistics
df.describe()

# Check for missing values in the dataframe
df.isnull().sum()

# Check for unique values in each column
df.nunique()

# Display the top 10 countries by World Population Percentage
df.sort_values(by="World Population Percentage", ascending=False).head(10)

# Calculate correlation matrix
correlation_matrix = df.corr()

# Plot heatmap of the correlation matrix
sns.heatmap(correlation_matrix, annot=True)
plt.rcParams['figure.figsize'] = (20,7)
plt.show()

# Group by continent and calculate the mean values, then sort by 2022 Population
df.groupby('Continent').mean().sort_values(by="2022 Population", ascending=False)

# Filter the dataframe to display only countries in Oceania
df[df['Continent'].str.contains('Oceania')]
