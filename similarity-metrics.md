# 📘 Similarity Metrics Library

This Python module implements a variety of **similarity metrics** for comparing values of different types—**nominal**, **numeric**, **textual**, **geographic**, and **temporal**. These metrics are rule-based and configurable, and can be used for record linkage, deduplication, and feature comparison.

All methods return a **user-defined result label** (e.g., `"MATCH"`, `"SIMILAR"`) if a similarity condition is met.

---

## 📦 Class: `SimilarityObject`

The class provides modular functions for comparing different kinds of data. Each function accepts a tuple of inputs and returns a defined output based on similarity logic.

---

## 📗 Metric Categories and Methods

### 1. 🏷 Nominal Similarity

#### `nominal_metric(x)`

- **Purpose**: Checks exact match of two nominal (categorical) values.
- **Input**: `(value1, value2, result)`
- **Output**: Returns `result` if `value1 == value2`, else `None`.
- **Calculation Process**:
  - Uses direct equality comparison (`==`) between two values.
- **Use Case**: Matching IDs, gender, binary flags.

---

### 2. 📋 List-Based Similarity

#### a. Jaccard Similarity

#### `num_metric_jaccard_similarity(x)`

- **Purpose**: Compares overlap between two lists of values.
- **Input**: `(list_str1, list_str2, threshold, result)`
- **Formula**:
  \[
  J(A, B) = \frac{|A \cap B|}{|A \cup B|}
  \]
- **Calculation Process**:
  1. Convert both comma-separated strings into sets.
  2. Compute intersection and union.
  3. Calculate Jaccard similarity.
  4. Return `result` if similarity > threshold.
- **Use Case**: Matching product tags, skills, preferences.

---

#### b. Euclidean Distance (on Lists)

#### `num_metric_euclidean(x)`

- **Purpose**: Measures distance between two numeric vectors.
- **Input**: `(list_str1, list_str2, threshold, result)`
- **Formula**:
  \[
  d = \sqrt{\sum_{i=1}^{n} (x_i - y_i)^2}
  \]
- **Calculation Process**:
  1. Convert both strings to numeric lists.
  2. Apply the Euclidean distance formula.
  3. Return `result` if distance < threshold.
- **Use Case**: Comparing multi-dimensional numeric values (e.g., vectors, ratings).

---

### 3. 📝 Textual Similarity

#### `cosine_similarity_value(x)`

- **Purpose**: Measures similarity between two strings using cosine angle.
- **Input**: `(text1, text2, threshold, result)`
- **Formula**:
  \[
  \cos(\theta) = \frac{A \cdot B}{||A|| \cdot ||B||}
  \]
- **Calculation Process**:
  1. Clean text (lowercase, remove punctuation and stopwords).
  2. Vectorize text using bag-of-words (`CountVectorizer`).
  3. Compute cosine similarity.
  4. Return `result` if similarity > threshold.
- **Use Case**: Comparing names, addresses, or titles with possible variations.

---

### 4. 🔢 Range-Based Numeric Similarity

#### a. Basic Range

#### `numeric_metric_range(x)`

- **Purpose**: Checks if both numbers fall within a defined interval.
- **Input**: `(val1, val2, range_str, result)`
- **Calculation Process**:
  1. Parse range string into numeric bounds.
  2. Check inclusion based on interval type (`[`, `]`, `(`, `)`).
  3. Return `result` if both values fall within the range.
- **Use Case**: Verifying whether values are inside valid or accepted ranges.

---

#### b. Range with Segments

#### `numeric_metric_range_with_segments(z)`

- **Purpose**: Checks if two values fall within the same segment of a range.
- **Input**: `(val1, val2, range_str, num_segments, result)`
- **Calculation Process**:
  1. Divide range into equal segments.
  2. Check which segment each value falls into.
  3. Return `result` if both values fall in the same segment.
- **Use Case**: Classification into brackets (e.g., salary or age bands).

---

#### c. Multi-Range

#### `numeric_metric_multi_range(z)`

- **Purpose**: Matches values against a list of accepted numeric intervals.
- **Input**: `(val1, val2, "range1-range2", result)`
- **Calculation Process**:
  1. Split string into individual ranges.
  2. For each range, check if both values fall into it.
  3. Return `result` on first match.
- **Use Case**: Accepting values that fall in multiple valid intervals.

---

### 5. 🌍 Geographic Similarity

#### `distance_cal_for_location_haversine(z)`

- **Purpose**: Measures the geospatial distance between two points.
- **Input**: `(lon1, lat1, lon2, lat2, unit, threshold, result)`
- **Formula**:
  \[
  d = 2r \cdot \arcsin\left(\sqrt{\sin^2\left(\frac{\Delta \phi}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta \lambda}{2}\right)}\right)
  \]
- **Calculation Process**:
  1. Parse latitude and longitude.
  2. Use Haversine formula (via `haversine` library).
  3. Compare distance against threshold.
  4. Return `result` if condition met.
- **Use Case**: Delivery zone check, nearest facility comparison.

---

### 6. 📅 Date-Based Similarity

#### a. Date Component Equality

#### `numeric_metric_date_equality(x)`

- **Purpose**: Compares specific part of two dates (day, month, or year).
- **Input**: `(date1, date2, format, part, result)`
- **Calculation Process**:
  1. Split dates using the provided format.
  2. Extract relevant part (`DAY`, `MONTH`, `YEAR`).
  3. Return `result` if parts are equal.
- **Use Case**: Date match for recurring events, partial date validation.

---

#### b. Date Euclidean Distance

#### `numeric_metric_date_euc(z)`

- **Purpose**: Measures difference between date components.
- **Input**: `(date1, date2, format, part, threshold, result)`
- **Calculation Process**:
  1. Extract the specified part of both dates.
  2. Compute absolute difference.
  3. Return `result` if difference < threshold.
- **Use Case**: Age or year-gap checks, timeline comparison.

---

### 7. ⏰ Time-Based Similarity

#### a. Time Euclidean Distance

#### `numeric_metric_time_euc(x)`

- **Purpose**: Measures time difference in hours.
- **Input**: `(time1, time2, format, threshold, result)`
- **Calculation Process**:
  1. Convert time strings to decimal hours.
  2. Compute absolute difference.
  3. Return `result` if difference < threshold.
- **Use Case**: Comparing appointment or shift proximity.

---

#### b. Time Range Match

#### `numeric_metric_time_range(x)`

- **Purpose**: Checks if both times fall into a single valid time interval.
- **Input**: `(time1, time2, format, "[a,b]", result)`
- **Calculation Process**:
  1. Convert times to decimal hours.
  2. Check if both times fall within range bounds.
- **Use Case**: Time-of-day validation (e.g., office hours).

---

#### c. Time Multi-Range Match

#### `numeric_metric_time_multirange(x)`

- **Purpose**: Matches times against multiple time ranges.
- **Input**: `(time1, time2, format, "range1-range2", result)`
- **Calculation Process**:
  1. Convert times to decimal hours.
  2. Iterate through each range and check for inclusion.
  3. Return `result` if any match is found.
- **Use Case**: Shift overlaps, multi-session validation.

---

## 🛠 Utility Functions

### `isFloat(s)`
- **Purpose**: Checks if a string can be parsed as float.
- **Returns**: `True` or `False`.

### `intersection(lst1, lst2)`
- **Purpose**: Returns list of common elements between two lists.

---

## ⚠️ Error Handling

All metric methods are wrapped in `try-except` blocks. Any parsing or comparison error results in `None` being returned safely.

---

## ✅ Example Use Cases

| Use Case                | Relevant Metric                     |
|-------------------------|-------------------------------------|
| Name matching           | `cosine_similarity_value`           |
| Location proximity      | `distance_cal_for_location_haversine` |
| Tag overlap             | `num_metric_jaccard_similarity`     |
| Age group check         | `numeric_metric_range_with_segments` |
| Event year check        | `numeric_metric_date_equality`      |
| Schedule proximity      | `numeric_metric_time_euc`           |

---

## 📎 Dependencies

```bash
pip install numpy scikit-learn nltk haversine
