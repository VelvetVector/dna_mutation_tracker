# Genomic Mutation Tracker

A C++ project that simulates a genomic mutation tracking pipeline using synthetic DNA data. It detects known mutation patterns, compares two string-matching algorithms, calculates basic patient-level statistics, and exports the results for SQL analysis.

## How it works

```text
Synthetic Patient Data
        ↓
   DNA Sequences
        ↓
 ┌──────┴──────┐
 │             │
 KMP       Rabin-Karp
 │             │
 └──────┬──────┘
        ↓
 Mutation Detection
        ↓
 Patient Statistics
        ↓
 CSV / MySQL Analysis
```

The project searches patient DNA for a predefined set of mutation patterns using:

* **KMP** — single-pattern matching using LPS preprocessing
* **Rabin-Karp** — multi-pattern matching using rolling hashes

Both algorithms are run on the same data and their results are compared for consistency. Their execution times are also recorded.

## Features

* Synthetic patient and DNA sequence generation
* Single-pattern search using KMP
* Multi-pattern search using Rabin-Karp
* Rolling hash with DNA bases encoded in base 4
* Mutation frequency analysis
* Mutation density calculation
* Simple project-defined risk scoring
* KMP vs. Rabin-Karp performance comparison
* CSV export and MySQL-based analysis

## Project Structure

```text
genomic_tracker/
├── generator.cpp       # Generates synthetic patient data
├── patient.h           # Patient data structure
├── kmp.cpp / kmp.h     # KMP pattern matching
├── rabin_karp.cpp/.h   # Rabin-Karp multi-pattern matching
├── main.cpp             # Main processing pipeline
├── db.cpp / db.h       # CSV output
├── insert_db.py        # Loads CSV data into MySQL
└── setup.sql            # Database setup and queries
```

## Running

Generate the patient data:

```bash
g++ -std=c++17 generator.cpp -o generator
./generator
```

Compile and run the tracker:

```bash
g++ -std=c++17 main.cpp kmp.cpp rabin_karp.cpp db.cpp -o tracker
./tracker
```

The program generates CSV files containing patient statistics, mutation matches, and mutation frequencies.

The CSV files can then be loaded into MySQL using `insert_db.py` and the schema in `setup.sql`.

## Tech Stack

* **C++17**
* **Python**
* **MySQL**
* STL data structures
* KMP & Rabin-Karp string matching
* SQL aggregation and analysis

## Note

The genomic data and mutation patterns are synthetic and intended for demonstrating algorithms and data processing. The risk score is a project-defined metric and is **not a medical or clinical prediction model**.
