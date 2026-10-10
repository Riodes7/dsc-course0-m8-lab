# Aviation Accident Analysis

# Commercial & Passenger Jet Airline Safety Analysis

## 1. Project Overview & Business Problem
Aviation insurance providers face substantial financial exposure from airframe hull loss payouts and severe passenger injury liability claims. As part of a data consulting firm, this analysis evaluates historical National Transportation Safety Board (NTSB) aviation accident records (1983–2023) to provide risk-minimized aircraft acquisition and underwriting recommendations.

The client requires:
1. Identification of aircraft makes and models that exhibit low rates of total airframe destruction and low likelihood of fatal or serious passenger injuries.
2. Distinct recommendations for **small aircraft** (< 20 occupants) versus **large passenger aircraft** (≥ 20 occupants).
3. Identification of external operating conditions that influence accident severity.
4. Statistically robust claims by enforcing minimum incident thresholds (≥ 50 incidents for manufacturers, ≥ 10 incidents for specific airplane types).

---

## 2. Dataset & Cleaning Pipeline

The raw dataset contains over 88,000 incident records. The following cleaning procedures were executed in `Aviations Accident Cleaning.ipynb`:

* **40-Year Operational Lifespan Filter**: Filtered records from 1983 onwards (`Event.Date >= 1983`) to evaluate modern active aircraft.
* **Category & Professional Build Enforcement**: Retained certified airplanes (`Aircraft.Category == 'Airplane'`) and eliminated kit/homebuilt aircraft (`Amateur.Built == 'No'`). Missing historical aircraft categories were imputed as airplanes prior to filtering.
* **Key Measurables Construction**:
  * **Total Occupants**: Sum of fatal, serious, minor, and uninjured persons aboard.
  * **Severe Injury Fraction**: Computed as `(Fatal + Serious Injuries) / Total Occupants` (guarded against division by zero).
  * **Airframe Destruction Binary**: Created an indicator (`1` if `Aircraft.damage == 'Destroyed'`, `0` otherwise).
* **Text Normalization & Entity Resolution**: Trimmed whitespace, converted strings to uppercase, and unified manufacturer synonyms (e.g., McDonnell Douglas, Boeing, Airbus).
* **Compound Identifier**: Created a unique `Make_Model` column (`Make + '_' + Model`) to accurately distinguish models across manufacturers.
* **Data Reduction**: Dropped non-informative columns and fields with fewer than 20,000 non-null values, exporting the resulting dataset as `cleaned_aviation_data.csv`.

---

## 3. Key Findings & Exploratory Data Analysis

### A. Small Aircraft (< 20 Occupants)
* **Top Recommended Manufacturers**:
  * **Bombardier / Canadair**: Demonstrates the lowest severe injury fraction (~8.8%) and a minimal airframe destruction rate (~2.6%).
  * **Maule**: Exceptional general aviation safety profile with a low destruction rate (~9.3%) and manageable injury fractions across over 500 incidents.
  * **Boeing & Airbus (Ferry/Regional Flights)**: Show very low destruction rates (< 6%) even under lower passenger configurations.
* **Injury Spread**: Distribution analysis (violin plots) shows that the top small aircraft makes have significant density clustered near zero severe injuries, meaning the vast majority of incidents result in minor injuries or uninjured passengers.

### B. Large Passenger Aircraft (≥ 20 Occupants)
* **Top Recommended Manufacturers**:
  * **Canadair / Bombardier Inc**: Regional jets exhibit low injury fractions (~1.8% to 2.7%) and total destruction rates under 5%.
  * **Airbus Industrie & Boeing**: The core commercial airliner fleet exhibits near-zero mean severe injury fractions per incident (< 0.5%) and exceptional hull survivability across high-volume service.
* **Top Specific Aircraft Models (Minimum 10 Incidents)**:
  * **Boeing 717-200**: Mean severe injury fraction of 0.14% and 0.0% hull destruction across incidents.
  * **Boeing 737 Next Gen (-7H4 / -800 / -300)**: Consistently under 0.3% severe injury fraction with hull preservation exceeding 95%.
  * **Airbus A320 Family (A320-232 / A321)**: Mean severe injury fraction of ~0.20% and 0.0% total hull destruction in qualifying incidents.
  * **McDonnell Douglas MD-80 / MD-83**: Proven structural survivability with severe injury rates around 0.11%.

---

## 4. Contributing Safety Factors (External Conditions)

Beyond aircraft make and model, two major operational variables strongly dictate accident severity:

1. **Weather Condition (`Weather.Condition`)**:
   * **Visual Meteorological Conditions (VMC)**: Incidents have an average severe injury rate of **22.8%** and a destruction rate of **17.0%**.
   * **Instrument Meteorological Conditions (IMC)**: Incidents in low-visibility or severe instrument conditions suffer an average severe injury rate of **63.4%** and an airframe destruction rate of **55.9%**.
   * *Takeaway*: Adverse weather increases accident lethality and total hull loss by nearly 3x due to spatial disorientation and reduced reaction time.

2. **Phase of Flight (`Broad.phase.of.flight`)**:
   * **Landing**: Accounts for the single highest volume of incidents, but exhibits the lowest severity (**4.4% severe injury rate**, **4.5% destruction rate**).
   * **Cruise, Climb, and Approach**: Account for lower incident volume but represent the highest risk zones (e.g., Climb: **38.3% injury rate, 37.8% destruction rate**; Cruise: **33.9% injury rate, 32.8% destruction rate**).
   * *Takeaway*: Takeoff and landing incidents are predominantly low-speed runway excursions or gear collapses that leave passengers uninjured, whereas en-route failures carry substantially higher kinetic energy and loss of control risk.

---

## 5. Client Recommendations

1. **Fleet Underwriting (Large Passenger Aircraft)**:
   * Prioritize underwriting modern **Airbus A320/A321** and **Boeing 737 Next Generation** airframes, alongside **Bombardier CRJ / Canadair** regional platforms. These models maintain statistical injury rates below 0.5% and near-zero hull destruction.
2. **Fleet Underwriting (Small Aircraft)**:
   * For light utility and business aircraft, prioritize **Bombardier** and **Maule** builds, while avoiding kit or amateur builds.
3. **Risk Rating / Policy Conditions**:
   * Integrate policy surcharges or operational requirements for aircraft operating frequently in uncertified IMC (Instrument Meteorological Conditions).
   * Require advanced avionics (terrain awareness and weather radar) to mitigate the single largest multiplier of fatal loss identified in the data.

---
