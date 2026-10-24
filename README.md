# Asteroid Close-Approach Database Analysis

This project uses PostgreSQL and Python to organize and analyze two related asteroid datasets. The notebook imports asteroid properties and close-approach records, reduces redundant attributes, and uses SQL to explore patterns in orbital class, size, velocity, and hazardous status.

The goal is to demonstrate relational database design, data preparation, normalization, and analytical SQL using asteroid data.

## Project Files

- [`126_Project.ipynb`](126_Project.ipynb) - Main Google Colab / Jupyter notebook containing data loading, schema creation, normalization, SQL queries, and saved analysis outputs.
- [Project report](CSCI%20126%20Report%20-%20Asteroid%20Close-Approach%20Database%20Analysis.pdf) - Original CSCI 126 report explaining the datasets, database design, queries, and findings.
- [`requirements.txt`](requirements.txt) - Python dependencies for running the notebook.

## Datasets Used

This project uses two Kaggle datasets:

1. **[Asteroid Dataset](https://www.kaggle.com/datasets/sakhawat18/asteroid-dataset)**
   - Downloaded with `sakhawat18/asteroid-dataset`.
   - Contains asteroid identifiers, orbital measurements, absolute magnitude, and orbit classifications.
   - `spkid` identifies each asteroid.
2. **[NeoWs Hazardous Asteroid Dataset](https://www.kaggle.com/datasets/alvinb/neows-hazardous-asteroid-dataset)**
   - Downloaded with `alvinb/neows-hazardous-asteroid-dataset`.
   - Contains close-approach dates, estimated diameters, relative velocities, miss distances, and hazardous labels.
   - Five CSV files are combined; `neo_reference_id` connects events to asteroid records.

The source datasets are downloaded separately and are not bundled in this repository.

## Tools and Libraries

- Python
- PostgreSQL (originally hosted on Neon)
- pandas
- SQLAlchemy
- psycopg2
- kagglehub
- Google Colab / Jupyter Notebook

## Workflow

1. Connect to a user-provided PostgreSQL database.
2. Download the two datasets using `kagglehub`.
3. Create initial asteroid and close-approach tables.
4. Combine the close-approach CSV files and remove redundant measurement columns.
5. Retain records with matching asteroid identifiers across the two datasets.
6. Load the reduced data into PostgreSQL and verify insert counts.
7. Remove redundant attributes and move asteroid-level diameter estimates and hazardous status into the asteroid table.
8. Run analytical queries using joins, aggregation, subqueries, and `HAVING`.
9. Create a reusable view joining asteroid properties to close-approach events.

## Database Structure

| Table / View | Purpose | Key |
|---|---|---|
| `asteroid_reduced` | Asteroid identity, orbital properties, estimated diameter, and hazardous status | `spkid` primary key |
| `close_approach_reduced` | Individual approach dates, velocities, distances, and orbiting bodies | `approach_id` primary key |
| `asteroid_close_approach_view` | Reusable join of asteroid properties and approach events | Joins `spkid` to `neo_reference_id` |

One asteroid can have multiple close-approach records. The normalization exercise separates asteroid-level properties from event-level properties. The notebook joins the tables by identifier; unlike the schema example in the report, its table-creation SQL does not declare a foreign-key constraint.

## SQL Analysis

The notebook answers 13 analysis questions:

- Which asteroids have the most recorded close approaches?
- Which asteroids have the smallest recorded miss distances?
- What are the five closest individual approach events?
- Which orbit classes contain the most asteroids?
- Which orbit classes have the smallest average minimum orbit intersection distance (MOID)?
- Which years have the most recorded close approaches?
- Which orbit classes have the highest average approach velocity?
- Which orbit classes have the largest average estimated diameter?
- Which hazardous asteroids have the smallest miss distances?
- How does average velocity compare between hazardous and non-hazardous records?
- Which hazardous approaches fall below the overall average miss distance?
- Which orbit classes have an average MOID below the overall average?
- How can a view simplify repeated asteroid / approach joins?

## Results Summary

The following values come from the saved notebook outputs, rather than a new database run:

| Analysis | Saved result |
|---|---|
| Most recorded approaches for one asteroid | 277810 (2006 FV35), with 251 records |
| Smallest recorded miss distance | (2011 CQ1), approximately 11,851.67 km |
| Most common retained orbit class | APO, with 12,547 asteroids |
| Lowest average MOID | ATE, followed by APO |
| Highest average approach velocity by class | IEO, approximately 58,840.61 km/h |

## Key Findings

- Separating asteroid records from approach events supports analysis at both the object and event levels.
- Joins connect physical and orbital properties to repeated approach records.
- Aggregation and subqueries make it possible to compare orbit classes and isolate records below dataset-wide averages.
- A joined view provides a reusable starting point for later analysis.

These are descriptive findings from the selected historical datasets. Hazardous status is a source label, not an estimate of impact probability. The closest-approach queries do not explicitly filter `orbiting_body`; interpreting every result as an Earth approach requires confirming that field in the data.

## How to Run

### Read the existing analysis

Open the notebook on GitHub to review its saved query outputs, or read the PDF report. No database connection is needed to view these files.

### Run against your own PostgreSQL database

1. Install the dependencies:

   ```bash
   python -m pip install -r requirements.txt
   ```

2. Provide your own PostgreSQL connection string as an environment variable named `DATABASE_URL`, or as a Google Colab Secret with that name. No shared database login is included. Keep credentials outside committed files.
3. Open `126_Project.ipynb` in Jupyter or Colab.
4. Choose the mode in the connection cell:
   - `preview_user = True` (default): run analysis against an already prepared database containing the final tables and view. Import and schema-changing cells are skipped.
   - `preview_user = False`: perform the original import and normalization workflow, using your own empty database with permission to create and alter tables.
5. Run the cells in order. For a fresh import, Kaggle access may require authentication according to the datasets' access settings.

The import changes table structure and appends data. It is intended for a fresh database, and is not safe to rerun as an incremental update. Use a separate database for a new import; do not repeat the preparation cells against an already normalized schema.

## Reproducibility Notes

- The uploaded notebook replaces the original embedded connection with user-provided configuration, uses the paths returned by `kagglehub`, and skips table creation in preview mode. The analytical SQL and saved result tables are preserved.
- Code cells were checked for Python syntax. The notebook has not been rerun end-to-end against a fresh PostgreSQL database for this upload; doing so requires your own database, dataset downloads, and the setup above.
- The PDF is the original course report. It differs from the notebook in a few details: the report groups yearly counts by `a_date`, while the notebook uses `close_approach_date`; some report examples keep `hazardous` on the event table, while the notebook moves it to the asteroid table. The README's numeric results follow the saved notebook outputs.
- CSV columns are assigned to the initial database schema by position, so changes to upstream dataset column order require review before import.

## Related Project

[Hazardous Asteroid Classification](https://github.com/Dexless/PHA-Asteroid-Classification) uses the same dataset sources to compare supervised machine learning classifiers.
