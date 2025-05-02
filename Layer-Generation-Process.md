# Layer Generation Process

## 1. Overview

The Layer Generation Module is part of a Multilayer Network System that automates the creation of intralayer and interlayer files from structured CSV data using configuration-based inputs. 

### Key Definitions

* **Intralayer**: A layer that represents relationships among entities (nodes) within the same dataset or layer. For example, connecting movies based on genre similarity.
* **Interlayer**: A layer that connects nodes from two different layers or datasets, representing cross-layer relationships. For example, linking actors to directors based on collaboration.
* **Node**: An entity in a dataset, such as a movie, actor, or accident record.
* **Edge**: A connection between two nodes, possibly with a weight and/or label depending on similarity or relationship.
* **Layer**: A single network constructed from a dataset where nodes are connected based on defined similarity criteria.

---

## 2. Input Files and Configuration

### 2.1 Input File Types

| File Type                  | Extension | Description                         |
| -------------------------- | --------- | ----------------------------------- |
| Input File                 | `.csv`    | Source data with node attributes    |
| Configuration File         | `.gen`    | Defines parameters and layers       |
| Primary Key Converter File | `.map`    | Maps primary key values to node IDs |

### 2.2 Configuration File Example

```ini
INPUT_DIRECTORY=$MLN_USR/IMDb/data_files
OUTPUT_DIRECTORY=$MLN_USR/IMDb/layers_generated
USERNAME=itlab
```

Block definitions:

```ini
BEGIN_LAYER
...parameters...
END_LAYER

BEGIN_INTERLAYER
...parameters...
END_INTERLAYER
```

---

## 3. Output Files

### 3.1 Output File Types

| File Type       | Extension | Description                                |
| --------------- | --------- | ------------------------------------------ |
| Layer File      | `.net`    | Output of intralayer generation            |
| Interlayer File | `.ilf`    | Output linking nodes between layers        |
| Log File        | `.log`    | Records layer creation details             |
| Hash Table File | `.bin`    | Stores attributes used in layer generation |

### 3.2 Output Format (Intralayer)

```
LayerName
#Nodes
#Edges
nodeID1
nodeID2
...
nodeID1,nodeID2,weight
```

### 3.3 Sample Output File (Excerpt)

```
movies_in_a_5year_period
64
83
0
1
2
3
...
0,1,1.0
0,2,1.0
2,4,1.0
...
```

**Explanation:**

* `Line 1`: Name of the layer
* `Line 2`: Number of nodes (64)
* `Line 3`: Number of edges (83)
* `Lines 4–(4+n)`: Node IDs
* `Remaining lines`: Edges in format `Node1,Node2,Weight`

### 3.4 Output Format (Interlayer)

```
InterLayerName
nodeID1,nodeID2,relationship
...
```

---

## 4. Intralayer Generation

### 4.1 Purpose

Links nodes in the same dataset using similarity, equality, or range.

### 4.2 Parameters

| Parameter                         | Example                   | Description                                             |
| --------------------------------- | ------------------------- | ------------------------------------------------------- |
| INPUT\_FILE\_NAME                 | `large-8kMovies.csv`      | Input file located in the input directory               |
| LAYER\_NAME                       | `similar_movie_rating`    | Name of the generated intralayer file                   |
| PRIMARY\_KEY\_COLUMN              | `MLN_MID,MovieName`       | From CSV: uniquely identifies each row/node             |
| FEATURE\_COLUMN                   | `MovieRating`             | From CSV: used to compare similarity between nodes      |
| FEATURE\_TYPE                     | `NUMERIC`                 | Indicates data type of the FEATURE\_COLUMN              |
| SIMILARITY\_METRIC                | `EUCLIDEAN`               | Method to calculate similarity between nodes            |
| THRESHOLD                         | `0.2`                     | Maximum allowed difference for similarity connection    |
| RANGE                             | `[1970,1980]`             | (optional) connect nodes within value limits            |
| MULTI\_RANGE                      | `[1970,1980]-[1981,1990]` | (optional) multiple intervals to check value membership |
| NUMBER\_OF\_EQUI\_SIZED\_SEGMENTS | `5`                       | (optional) splits a range into equal-sized segments     |

### 4.3 Feature Type vs Metric

| Feature Type | Metrics             | Extra Parameters    |
| ------------ | ------------------- | ------------------- |
| NUMERIC      | EQUALITY, EUCLIDEAN | THRESHOLD           |
| GEOGRAPHIC   | HAVERSINE           | LAT/LONG, THRESHOLD |
| SET          | JACCARD             | THRESHOLD           |
| TEXT         | COSINE              | THRESHOLD           |
| DATE/TIME    | RANGE, EQUALITY     | FORMAT, RANGE       |

### 4.4 Example Configuration with CSV Fields

Assume `large-8kMovies.csv` contains:

```
MLN_MID,MovieName,MovieRating,MovieYear,MovieGenreList
101,Inception,8.7,2010,{ACTION,SCI-FI}
102,The Matrix,8.6,1999,{ACTION,SCI-FI}
...etc
```

Then configuration for mapping:

```ini
BEGIN_LAYER
INPUT_FILE_NAME=large-8kMovies.csv
LAYER_NAME=similar_movie_rating
LAYER_GENERATION_TYPE=System_Generated
PRIMARY_KEY_COLUMN=MLN_MID,MovieName
FEATURE_COLUMN=MovieRating
FEATURE_TYPE=NUMERIC
SIMILARITY_METRIC=EUCLIDEAN
THRESHOLD=0.2
END_LAYER
```

This will generate edges between movies with similar ratings (difference <= 0.2).

---

## 5. Interlayer Generation

### 5.1 Mapping-Based

In mapping-based interlayer generation, the input file must contain a list of pairs of node identifiers (one from each layer) along with a relationship label. These are used to link existing nodes from Layer 1 and Layer 2.

| Parameter                   | Example                |
| --------------------------- | ---------------------- |
| INPUT\_FILE\_NAME           | `actors_directors.csv` |
| LAYER 1 NAME                | `Actors`               |
| LAYER 2 NAME                | `Directors`            |
| INTER LAYER NAME            | `works_with`           |
| INTER LAYER GENERATION TYPE | `System_Generated`     |

#### Example

Assume `actors_directors.csv` contains:

```
ActorID,DirectorID,Relation
182,1,works with
182,308,works with
183,2,works with
```

This file defines mappings between actors and directors based on their collaborations.

This represents relationships between actors and directors who have worked together. Each row maps one actor to one director with the label 'collaborated'.

Then configuration:

```ini
BEGIN_INTERLAYER
INPUT_FILE_NAME=actors_directors.csv
LAYER 1 NAME=Actors
LAYER 2 NAME=Directors
INTER LAYER NAME=actor_director
INTER LAYER GENERATION TYPE=System Generated
END_INTERLAYER
```

### 5.2 Join-Based

| Parameter               | Example           |
| ----------------------- | ----------------- |
| LAYER 1 INPUT FILE NAME | `actors.csv`      |
| LAYER 2 INPUT FILE NAME | `directors.csv`   |
| JOIN COLUMN NAME        | `MovieID`         |
| RELATIONSHIP NAME       | `worked_together` |

#### Example

Assume `actors.csv` and `directors.csv` both contain the field `MovieID`:

```ini
BEGIN_INTERLAYER
LAYER 1 NAME=Actors
LAYER 2 NAME=Directors
JOIN COLUMN NAME=MovieID
RELATIONSHIP NAME=worked_together
INTER LAYER NAME=actor_director
INTER LAYER GENERATION TYPE=System_Generated
END_INTERLAYER
```

This links actors and directors who share the same `MovieID`.

---

