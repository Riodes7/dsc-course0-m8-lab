# **Commercial & Passenger Jet Airline Safety Analysis**

## **Updated Implementation Blueprint & Precision Code Hints**

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

* **Location**: Cell 2 (*Make relevant library imports*).  
* **Hint**:  
  * Load core numerical and visualization libraries:  
    Python  
    \# Hint: Import standard aliases for data handling and plotting  
    \# pd, np, plt, sns

### **Step 2: Data Loading & Initial Inspection**

* **Location**: Cell 5 (*Data Loading and Inspection*).  
* **Hint**:  
  * Avoid UnicodeDecodeError by specifying the encoding:  
    Python  
    \# Hint: Specify encoding='ISO-8859-1' and low\_memory=False  
    \# df \= pd.read\_csv('AviationData.csv', encoding='ISO-8859-1', low\_memory=False)  
    \# Check dimensions and null counts: df.shape, df.info(), df.isna().sum()

### **Step 3: Filtering Aircraft and Events**

* **Location**: Cell 9 (*Filtering aircrafts and events*).  
* **Hint**:  
  * Convert dates and filter to valid professional airplanes from 1983 onward:  
    Python  
    \# 1\. Parse Event.Date to datetime: pd.to\_datetime(df\['Event.Date'\])  
    \# 2\. Extract year and filter: df\['Event.Date'\].dt.year \>= 1983  
    \# 3\. Filter Aircraft.Category \== 'Airplane' (handle or inspect NaN cases first)  
    \# 4\. Filter Amateur.Built: keep rows where df\['Amateur.Built'\] \== 'No'

### **Step 4: Cleaning & Constructing Key Measurables**

* **Location**: Cells 12 and 14 (*Cleaning and constructing Key Measurables*).  
* **Hint**:  
  * **Cell 12 (Severe Injury Fraction)**:  
    Python  
    \# Fill NaN values in injury columns with 0:  
    \# \['Total.Fatal.Injuries', 'Total.Serious.Injuries', 'Total.Minor.Injuries', 'Total.Uninjured'\]  
    \# Total on board \= sum of the 4 columns above  
    \# Severe injury fraction \= (Fatal \+ Serious) / Total on board  
    \# Guard against 0 occupants: np.where(total \> 0, (fatal \+ serious) / total, np.nan)

  * **Cell 14 (Aircraft Destruction Binary)**:  
    Python  
    \# Note exact column name: 'Aircraft.damage' (lowercase 'd')  
    \# Create binary flag: (df\['Aircraft.damage'\] \== 'Destroyed').astype(int)

### **Step 5: Cleaning the Make Column & Sample Gating**

* **Location**: Cell 16 (*Investigate the Make column*).  
* **Hint**:  
  * Standardize casing, trim whitespace, and enforce the 50-incident threshold:  
    Python  
    \# Clean text: df\['Make'\].astype(str).str.strip().str.upper()  
    \# Unify synonyms: e.g., mapping 'MCDONNELL DOUGLAS' and 'MCDONNELL-DOUGLAS'  
    \# Count frequencies: df\['Make'\].value\_counts()  
    \# Retain only makes with count \>= 50

### **Step 6: Inspecting & Creating Unique Model Identifiers**

* **Location**: Cell 18 (*Inspect Model column*).  
* **Hint**:  
  * Drop null models and combine Make \+ Model to form a unique identifier:  
    Python  
    \# Drop rows where df\['Model'\].isna()  
    \# Clean text: df\['Model'\].astype(str).str.strip().str.upper()  
    \# Create compound identifier: df\['Make'\] \+ '\_' \+ df\['Model'\]

### **Step 7: Cleaning Other Auxiliary Columns & Optional State Mapping**

* **Location**: Cell 20 (*Cleaning other columns*).  
* **Hint**:  
  * Standardize target factor columns: Engine.Type, Weather.Condition, Number.of.Engines, Broad.phase.of.flight, Purpose.of.flight.  
  * *(Optional Geographic Enrichment using USState\_Codes.csv)*:  
    Python  
    \# Load state codes: states \= pd.read\_csv('USState\_Codes.csv')  
    \# Extract postal abbreviation: df\['State\_Code'\] \= df\['Location'\].str.split(',').str\[-1\].str.strip()  
    \# Merge on abbreviation to obtain full state names:  
    \# df.merge(states, left\_on='State\_Code', right\_on='Abbreviation', how='left')

### **Step 8: Dropping Sparse Columns & Exporting**

* **Location**: Cells 22 and 24 (*Column Removal* and *Save DataFrame to csv*).  
* **Hint**:  
  * Drop high-missingness columns and save the intermediate output:  
    Python  
    \# Cell 22: Drop columns with fewer than 20,000 valid values:  
    \# df.dropna(axis=1, thresh=20000)  
    \# Cell 24: Save clean dataset:  
    \# df.to\_csv('cleaned\_aviation\_data.csv', index=False)

## **Part 2: Exploratory Data Analysis (Aviation Accident Data Analysis.ipynb)**

### **Step 1: Loading Clean Data**

* **Location**: Cell 3 (*Exploratory Data Analysis*).  
* **Hint**:  
  * Load cleaned\_aviation\_data.csv using pd.read\_csv() and verify shapes.

### **Step 2: Passenger Threshold & Manufacturer (Make) Benchmarks**

* **Location**: Cells 5, 7, 9, 11, 13, and 15\.  
* **Hint**:  
  * **Cell 5 (Subsetting by Size)**:  
    * small\_aircraft \= df\[df\['Total\_Occupants'\] \< 20\]

    * large\_aircraft \= df\[df\['Total\_Occupants'\] \>= 20\]

  * **Cell 7 (Safest 15 Makes)**:  
    * Group by Make, compute mean injury fraction, extract .nsmallest(15) for both small and large subsets, and plot side-by-side using sns.barplot.  
  * **Cell 9 (Violin Plot \- Small Makes)**:  
    * Select the 10 lowest mean injury small makes and plot with sns.violinplot(data=..., x='Make', y='Injury\_Fraction').  
  * **Cell 11 (Strip Plot \- Large Makes)**:  
    * Select the 10 lowest mean injury large makes and plot with sns.stripplot(data=..., x='Make', y='Injury\_Fraction', jitter=True).  
  * **Cell 13 (Lowest 15 Destruction Rates)**:  
    * Group by Make, calculate mean of destroyed binary flag, sort ascending, and take .head(15).  
  * **Cell 15 (Markdown Summary)**:  
    * Identify manufacturers that excel in both low injury rates and low airframe destruction rates.

### **Step 3: Individual Plane Type (Model) Evaluation**

* **Location**: Cells 18, 20, and 22 (*Analyze plane types*).  
* **Hint**:  
  * Enforce the minimum incident sample size (\$\\ge 10\$ events per plane type):  
    Python  
    \# Count occurrences: plane\_counts \= df\['Make\_Model'\].value\_counts()  
    \# Filter valid models: valid\_models \= plane\_counts\[plane\_counts \>= 10\].index  
    \# Subset: df\_filtered \= df\[df\['Make\_Model'\].isin(valid\_models)\]

  * Compute mean injury ratios and plot distributions (sns.boxplot or sns.stripplot) for large planes (Cell 18\) and small planes (Cell 20).  
  * In Cell 22, provide Markdown recommendations highlighting the specific models demonstrating the lowest risk profiles.

### **Step 4: Investigating External Contributing Factors**

* **Location**: Cell 24 (*Exploring Other Variables*).  
* **Hint**:  
  * Select 2 variables from: Weather.Condition, Engine.Type, Number.of.Engines, Broad.phase.of.flight, or Purpose.of.flight.  
  * Group by each variable against injury fraction and destruction rate:  
    Python  
    \# factor\_summary \= df.groupby('Weather.Condition')\[\['Injury\_Fraction', 'Destroyed\_Binary'\]\].mean()  
    \# Visualize using sns.barplot or stacked charts

  * Provide Markdown interpretation explaining how poor conditions (e.g., IMC vs. VMC) or high-workload phases (e.g., takeoff/landing) influence accident outcomes.
just the heading styles as desired, and go to **File → Download → PDF Document (.pdf)**.