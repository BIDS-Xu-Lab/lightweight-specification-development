We plan to develop a lightweight terminal-based tool for exploring patient-level data stored in OMOP Common Data Model (CDM) format. The tool is designed for quick inspection of patient records without requiring the deployment of large analytics platforms. The program operates entirely in the terminal environment and reads OMOP CDM tables stored as CSV files. 

# Technical Requirements
- Python
- Package Management: uv
- User Interface:
  - Textual for terminal UI framework
  - Rich for rendering tables, panels, and formatted content
- Data Format: CSV files representing OMOP CDM tables
- Execution Environment: Terminal application

# Data Input
When starting the program, the user must provide a PATH pointing to a directory containing OMOP CDM tables stored as CSV files.

Example directory structure:
```bash
sample_omop_data/
    person.csv
    visit_occurrence.csv
    observation.csv
    measurement.csv
    drug_exposure.csv
    ...
```

Each OMOP table is stored as a separate CSV file.
At startup, the program scans the provided directory and identifies available OMOP tables.
The folder `sample_omop_data/` contains example OMOP CSV files used for development and testing.


# Interface Layout
The terminal interface is organized into two main columns:

## Left Column: Patient List

The left column displays a list of patients derived from the person table.
Each row represents one patient and contains:
- person_id
- Visit count: Total number of visits for this patient
- Records count in other tables

Because datasets may contain many patients, the patient list should support pagination.

## Right Column: Patient Detail View
When a patient is selected in the left panel, the right panel displays detailed information about that patient.

In this view, patient basic information displays at the top. 
Then the bottom is splitted into two columns, each column has one panel.
Depends on the viewing mode, the content is different.
Two viewing modes are supported, and users can switch between modes using a keyboard shortcut.

### Mode 1: Table View

In Table View, the left panel shows the table names. 
The right panel shows the records of the current patient from the selected OMOP table.

### Mode 2: Visit View

The left panel displays all visits associated with the selected patient.
Each row includes:
- Visit ID
- Visit start date/time

Visits are ordered chronologically.

The right panel displays all records associated with the selected visit ordered by tables.
All information is organized by visit, allowing users to quickly inspect clinical events occurring during that visit.