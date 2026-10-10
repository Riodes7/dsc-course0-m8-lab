# **Commercial & Passenger Jet Airline Safety Analysis**

### **Project Architecture & Parameters**

* **Target Objective**: Identify aircraft makes and models exhibiting low rates of total airframe destruction and low probability of serious/fatal passenger injury.  
* **Filter Constraints**:  
  * Professional builds only (Amateur.Built \== 'No').  
  * 40-year operational window (Event.Date \$\\ge\$ 1983).  
  * Aircraft category restricted to airplanes (Aircraft.Category \== 'Airplane').  
* **Segmentation & Thresholds**:  
  * Passenger size split: Small aircraft (\$\< 20\$ occupants) vs. Large aircraft (\$\\ge 20\$ occupants).  
  * Statistical reliability gates: \$\\ge 50\$ events per Make, \$\\ge 10\$ events per Model/Plane type.

## **Part 1: Data Cleaning (Aviations Accident Cleaning.ipynb)**

### **Step 1: Library Imports**

* **Make relevant library imports*.
  * Load core numerical and visualization libraries:   
    \# Import standard aliases for data handling and plotting  
    \# pd, np, plt, sns

### **Step 2: Data Loading & Initial Inspection**

* **Data Loading and Inspection*.
  * Avoid UnicodeDecodeError by specifying the encoding:   
    \# Specify encoding='ISO-8859-1' and low\_memory=False  
    \# df \= pd.read\_csv('AviationData.csv', encoding='ISO-8859-1', low\_memory=False)  
    \# Check dimensions and null counts: df.shape, df.info(), df.isna().sum()

### **Step 3: Filtering Aircraft and Events**

* **Filtering aircrafts and events*.   
  * Convert dates and filter to valid professional airplanes from 1983 onward:
    \# 1\. Parse Event.Date to datetime
    \# 2\. Extract year and filter: >= 1983  
    \# 3\. Filter Aircraft.Category \== 'Airplane' (handle or inspect NaN cases first)  
    \# 4\. Filter Amateur.Built: keep rows where df\['Amateur.Built'\] \== 'No'

### **Step 4: Cleaning & Constructing Key Measurables**

* **Cleaning and constructing Key Measurables*.
  * **Severe Injury Fraction)**:
    \# Fill NaN values in injury columns with 0:  
    \# \['Total.Fatal.Injuries', 'Total.Serious.Injuries', 'Total.Minor.Injuries', 'Total.Uninjured'\]  
    \# Total on board \= sum of the 4 columns above  
    \# Severe injury fraction \= (Fatal \+ Serious) / Total on board  
    \# Guard against 0 occupants: np.where(total \> 0, (fatal \+ serious) / total, np.nan)

  * **Aircraft Destruction Binary**: 
    \# Note exact column name: 'Aircraft.damage' (lowercase 'd')  
    \# Create binary flag: (df\['Aircraft.damage'\] \== 'Destroyed').astype(int)

### **Step 5: Cleaning the Make Column & Sample Gating**

* **Investigate the Make column*.  
  * Standardize casing, trim whitespace, and enforce the 50-incident threshold:
    \# Clean text: df\['Make'\].astype(str).str.strip().str.upper()  
    \# Unify synonyms: e.g., mapping 'MCDONNELL DOUGLAS' and 'MCDONNELL-DOUGLAS'  
    \# Count frequencies: df\['Make'\].value\_counts()  
    \# Retain only makes with count \>= 50

### **Step 6: Inspecting & Creating Unique Model Identifiers**

* **Inspect Model column*.
  * Drop null models and combine Make \+ Model to form a unique identifier:
    \# Drop rows where df\['Model'\].isna()  
    \# Clean text: df\['Model'\].astype(str).str.strip().str.upper()  
    \# Create compound identifier: df\['Make'\] \+ '\_' \+ df\['Model'\]

### **Step 7: Cleaning Other Auxiliary Columns & Optional State Mapping**

* **Cleaning other columns*.
  * Standardize target factor columns: Engine.Type, Weather.Condition, Number.of.Engines, Broad.phase.of.flight, Purpose.of.flight.   
    \# Load state codes: states
    \# Extract postal abbreviation
    \# Merge on abbreviation to obtain full state names

### **Step 8: Dropping Sparse Columns & Exporting**
*  **Drop high-missingness columns and save the intermediate output:
    \# Cell 22: Drop columns with fewer than 20,000 valid values:  
    \# df.dropna(axis=1, thresh=20000)  
    \# Cell 24: Save clean dataset:  
    \# df.to\_csv('cleaned\_aviation\_data.csv', index=False)

## **Part 2: Exploratory Data Analysis (Aviation Accident Data Analysis.ipynb)**

### **Step 1: Loading Clean Data**
  * Load cleaned\_aviation\_data.csv and verify shapes.

### **Step 2: Passenger Threshold & Manufacturer (Make) Benchmarks**
  * **Subsetting by Size**:  
    * small\_aircraft 

    * large\_aircraft 

  * **Safest 15 Makes**:  
    * Group by Make, compute mean injury fraction, extract .nsmallest(15) for both small and large subsets, and plot side-by-side using sns.barplot.  
  * **Violin Plot \- Small Makes**:  
    * Select the 10 lowest mean injury small makes and plot with sns.violinplot(data=..., x='Make', y='Injury\_Fraction').  
  * **Strip Plot \- Large Makes**:  
    * Select the 10 lowest mean injury large makes and plot with sns.stripplot(data=..., x='Make', y='Injury\_Fraction', jitter=True).  
  * **Lowest 15 Destruction Rates**:  
    * Group by Make, calculate mean of destroyed binary flag, sort ascending, and take .head(15).  
  * **Markdown Summary**:  
    * Identify manufacturers that excel in both low injury rates and low airframe destruction rates.

### **Step 3: Individual Plane Type (Model) Evaluation**
*Analyze plane types*).  
  * Enforce the minimum incident sample size (\$\\ge 10\$ events per plane type):  
    Python  
    \# Count occurrences 
    \# Filter valid models 
    \# Subset: df_filtered 

  * Compute mean injury ratios and plot distributions (sns.boxplot or sns.stripplot) for large planes (Cell 18\) and small planes (Cell 20).  
  * Provide Markdown recommendations highlighting the specific models demonstrating the lowest risk profiles.

### **Step 4: Investigating External Contributing Factors**

* **Exploring Other Variables*.
  * Select 2 variables from: Weather.Condition, Engine.Type, Number.of.Engines, Broad.phase.of.flight, or Purpose.of.flight.  
  * Group by each variable against injury fraction and destruction rate:
    \# factor\_summary \= df.groupby('Weather.Condition')...  
    \# Visualize using sns.barplot or stacked charts

  * Provide Markdown interpretation explaining how poor conditions (e.g., IMC vs. VMC) or high-workload phases (e.g., takeoff/landing) influence accident outcomes.