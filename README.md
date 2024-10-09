# 🚨 FraudAnalyticsLab 🚨
## Overview
Welcome! This repository is dedicated to sharing my coursework on **Fraud Analytics**. 
I'll be updating this space regularly with new code and insights on various types of fraud detection, including card transaction fraud, identity fraud, and more. 
Feel free to explore, contribute, and share your thoughts—collaboration is always welcome!

---

## I. 🔍 Card Transaction Fraud Detection

### 1. Consecutive Transaction Distance Analysis

#### 📝 Description
In this module, I focus on detecting **anomalous card transactions** by analyzing the **geodesic distance** between consecutive transactions made using the same card. 
Transactions involving unusually large distances within short timeframes might signal suspicious or fraudulent activity.

#### 🔬 Methodology
1. **Data Preprocessing**: Map merchant transaction ZIP codes to corresponding latitudes and longitudes.
2. **Distance Calculation**: Compute the geodesic distance (in miles) between consecutive transactions for each card.
3. **Anomaly Detection**: Flag transactions that exceed a predefined distance threshold (e.g., 1000 miles) as potentially suspicious.

### 📜 Code Breakdown
- **Imports**: Import necessary libraries for data processing and distance calculations.
- **Data Loading**: Include comments for loading your transaction and ZIP code data (uncomment and specify the correct paths).
- **ZIP Code Processing**: Ensure ZIP codes are formatted correctly and check for any missing codes.
- **Distance Calculation**: Implement a function to calculate the geodesic distance between two ZIP codes.
- **Anomaly Detection**: Calculate distances between consecutive transactions and flag suspicious transactions based on a defined threshold.
- **Output**: Display the transactions that have been flagged as suspicious.

#### 🔮 Next Steps
- **Fine-tune the threshold**: Adjust the distance threshold based on specific patterns identified in the dataset.
- **Explore additional features**: Consider incorporating time-based anomalies and other relevant features to enhance detection accuracy.
- **Data Visualization**: Create visual representations of flagged transactions to provide further insights into suspicious activity.

---

## 📂 Code Implementation
```python
import pandas as pd
from geopy.distance import geodesic  # To calculate distances between coordinates

# Load your transaction and ZIP code data here
df = pd.read_csv('path_to_your_transaction_data.csv')
zip_df = pd.read_csv('path_to_your_zip_code_data.csv')

# Display the first few rows of the ZIP code database
print(zip_df.head())

# Ensure ZIP codes in the transaction data are strings
# 'Merch zip' refers to the merchant ZIP code and is one of the feature names in my dataset.
# Please replace it with the corresponding feature name from your own dataset, if necessary.
df['Merch zip'] = df['Merch zip'].astype(str)

# Create a mapping from ZIP code to coordinates
zip_df['zip'] = zip_df['zip'].astype(str)
zip_to_coords = zip_df.set_index('zip')[['latitude', 'longitude']].to_dict('index')

# Check for missing ZIP codes in the transaction data
missing_zips = df[~df['Merch zip'].isin(zip_to_coords.keys())]['Merch zip'].unique()
print(f"Missing ZIP codes in the ZIP code database: {missing_zips}")

# Function to calculate distance between two ZIP codes
def calculate_distance(zip1, zip2):
    if zip1 in zip_to_coords and zip2 in zip_to_coords:
        coords_1 = zip_to_coords[zip1]
        coords_2 = zip_to_coords[zip2]
        return geodesic((coords_1['latitude'], coords_1['longitude']),
                        (coords_2['latitude'], coords_2['longitude'])).miles
    return None

# Calculate distance between consecutive transactions
df['prev_merch_zip'] = df.groupby('Cardnum')['Merch zip'].shift(1)
df['distance_to_last_transaction'] = df.apply(
    lambda row: calculate_distance(row['prev_merch_zip'], row['Merch zip']),
    axis=1)

# Set a threshold to flag suspicious transactions
distance_threshold = 1000
df['suspicious_distance'] = df['distance_to_last_transaction'] > distance_threshold

# Inspect the flagged transactions
suspicious_transactions = df[df['suspicious_distance'] == True]
print(suspicious_transactions[['Cardnum', 'Date', 'Merch zip', 'distance_to_last_transaction']])

