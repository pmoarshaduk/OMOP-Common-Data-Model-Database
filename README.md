# OMOP-Common-Data-Model-Database
OPMG CDM Unofficial SQL DB – Fast Start for Data Engineering, Research &amp; Clinical Analytics
# Unofficial OMOP Common Data Model Database with Synthetic Patient Data

Welcome to my open source project: a fully functional **OMOP CDM 5.4** database populated with realistic synthetic healthcare data from **Synthea**. This repository is designed for data engineers, epidemiologists, and healthcare analysts who want to practice working with the Observational Medical Outcomes Partnership (OMOP) standard without using real patient records.

Whether you are learning OMOP, testing ETL pipelines, or building clinical research tools, this dataset gives you a production ready environment with **174 synthetic patients**, thousands of clinical events, and a complete set of OMOP vocabulary tables.

## Why This Project Exists

The OMOP Common Data Model is the industry standard for transforming disparate healthcare data into a unified format. However, getting started with OMOP is hard. You need to understand the schema, load vocabulary tables, map source codes to standard concepts, and ensure referential integrity. This repository removes those barriers by providing a pre built, ready to query OMOP database (Microsoft SQL Server) that you can restore, explore, and extend immediately.

I built this to help the community learn faster and to showcase my skills as a data engineer specializing in healthcare data interoperability.

## What Is Included

### Full OMOP CDM 5.4 Schema
All core clinical tables are present and correctly linked.

### Realistic Synthetic Data
All data comes from **Synthea 3.0** (an open source synthetic patient generator). The dataset includes 174 patients with demographics, encounters, conditions, procedures, medications, immunizations, observations, measurements, devices, providers, care sites, and insurance coverage periods.

### Complete Vocabulary Tables
The repository includes the essential OMOP vocabulary tables (SNOMED, RxNorm, LOINC, etc.) with over 2.2 million standard concepts. This allows you to map source codes to standard concept IDs and run OMOP compliant analytics.

### ETL Scripts (SQL Server)
All transformation logic is provided as SQL scripts that you can inspect, modify, and reuse. The scripts load CSV files, stage raw data, map to OMOP tables, and apply concept mappings.

### Data Quality Checks
A set of validation queries ensures referential integrity, date consistency, and concept coverage.

## What Is Excluded

This is a **synthetic dataset** generated for demonstration and learning. It does not contain real patient information.

The following Synthea modules are **not** included because they add complexity without significant value for most learning scenarios:

* claims and claims transactions (cost data)
* imaging studies (complex mapping)
* supplies and careplans
* note and note NLP tables

If you need those domains, you can extend the ETL scripts using the provided patterns.

The vocabulary tables are not the full OHDSI vocabulary (that would be over 10 GB). Instead, they include the most commonly used concepts for drugs, conditions, procedures, measurements, and vaccines. This is sufficient for 99% of learning and prototyping tasks.

## Table Summary (Row Counts)

| OMOP Table | Number of Rows |
|------------|----------------|
| PERSON | 174 |
| OBSERVATION_PERIOD | 174 |
| VISIT_OCCURRENCE | 8,375 |
| CONDITION_OCCURRENCE | 11,807 |
| PROCEDURE_OCCURRENCE | 24,543 |
| DRUG_EXPOSURE | 11,139 |
| OBSERVATION | 40,925 |
| MEASUREMENT | 66,279 |
| DEVICE_EXPOSURE | (varies, see script) |
| PROVIDER | 374 |
| CARE_SITE | 374 |
| PAYER_PLAN_PERIOD | 5,868 |
| CONCEPT | 2,206,792 |
| CONCEPT_ANCESTOR | 50+ million (loaded but not listed) |
| VOCABULARY | 152 |
| All other vocabulary tables | Fully loaded |

**Total clinical event rows** (conditions + procedures + drugs + observations + measurements + devices): over **170,000** records.

## Database Diagram

Below is a simplified view of how the core tables relate to each other. You can find a full entity relationship diagram in the `docs` folder of this repository.

```
PERSON ──┬── VISIT_OCCURRENCE ──┬── CONDITION_OCCURRENCE
         │                      ├── PROCEDURE_OCCURRENCE
         │                      ├── DRUG_EXPOSURE
         │                      ├── OBSERVATION
         │                      └── MEASUREMENT
         ├── PROVIDER ─── CARE_SITE
         └── PAYER_PLAN_PERIOD
```

Every clinical event is linked to a person and a visit. Providers and care sites are linked to visits and to providers themselves.

## How to Use This Repository

### Prerequisites
* Microsoft SQL Server 2019 or newer (including Express Edition)
* SQL Server Management Studio (SSMS) or Azure Data Studio
* At least 20 GB free disk space for the vocabulary tables

### Setup Instructions

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/omop-synthea-demo.git
   cd omop-synthea-demo
   ```

2. **Restore the database backup** (if you want a ready to use database)
   * The file `OPMGCDM.bak` is a full backup of the database.
   * In SSMS, right click on Databases → Restore Database → Device → select the `.bak` file.

3. **Or run the ETL scripts from scratch**
   * Execute `00_create_schema.sql` to create all OMOP tables.
   * Run `01_load_vocabulary.sql` to import the vocabulary CSV files (you must download them from Athena first – see instructions below).
   * Execute `02_stage_synthea.sql` to create staging tables.
   * Run `03_load_synthea_tables.sql` to populate the OMOP tables from the staged data.
   * Finally, run `04_map_concepts.sql` to replace source codes with standard concept IDs.

4. **Verify the installation**
   ```sql
   SELECT COUNT(*) FROM PERSON;
   SELECT COUNT(*) FROM VISIT_OCCURRENCE;
   ```
   You should see 174 and 8,375 respectively.

## Vocabulary Download Instructions

The vocabulary tables are essential for OMOP but too large to store directly on GitHub. You can download them for free from the official OHDSI Athena tool.

1. Go to [https://athena.ohdsi.org](https://athena.ohdsi.org) and create an account.
2. Select the following vocabularies: SNOMED, RxNorm, LOINC, CVX, CPT 4, ICD10CM, ICD10PCS, and all domain vocabularies (Gender, Race, Ethnicity, Visit, etc.).
3. Download the package (a ZIP file about 1.5 GB).
4. Extract the CSV files into the `vocab` folder of this repository.
5. Run the `01_load_vocabulary.sql` script – it will bulk insert the CSVs into the appropriate tables.

The repository already includes a small subset of the vocabulary for testing, but for full functionality you should complete this step.

## Example Queries to Get Started

Here are a few SQL queries to explore the data.

### Count of patients by gender
```sql
SELECT gender_concept_id, COUNT(*) FROM PERSON GROUP BY gender_concept_id;
```

### Most common conditions
```sql
SELECT c.concept_name, COUNT(*) as frequency
FROM CONDITION_OCCURRENCE co
JOIN CONCEPT c ON co.condition_concept_id = c.concept_id
GROUP BY c.concept_name
ORDER BY frequency DESC;
```

### Average blood pressure measurement
```sql
SELECT AVG(value_as_number) as avg_systolic
FROM MEASUREMENT
WHERE measurement_source_value = '8480-6';   -- Systolic blood pressure LOINC code
```

### Number of visits per provider
```sql
SELECT p.provider_name, COUNT(*) as visits
FROM VISIT_OCCURRENCE v
JOIN PROVIDER p ON v.provider_id = p.provider_id
GROUP BY p.provider_name
ORDER BY visits DESC;
```

## Project Structure

```
omop-synthea-demo/
│
├── README.md
├── LICENSE
├── OPMGCDM.bak                 (database backup ~2 GB)
│
├── sql/
│   ├── 00_create_schema.sql
│   ├── 01_load_vocabulary.sql
│   ├── 02_stage_synthea.sql
│   ├── 03_load_synthea_tables.sql
│   ├── 04_map_concepts.sql
│   └── 05_data_quality.sql
│
├── staging/                    (sample CSVs from Synthea)
├── docs/
│   ├── ER_diagram.png
│   └── ETL_workflow.pdf
│
└── vocab/                      (place vocabulary CSVs here)
```

## Technologies Used

* **OMOP CDM 5.4** – Standard data model
* **Synthea 3.0** – Synthetic data generator
* **SQL Server 2022 Express** – Relational database
* **OHDSI Athena** – Vocabulary distribution
* **GitHub Actions** – For future CI/CD (coming soon)

## Who Is This For

* **Data engineers** who want to see a complete ETL pipeline from raw CSV to OMOP compliant database.
* **Epidemiologists and biostatisticians** who need a sandbox to test cohort definitions before using real data.
* **Healthcare data analysts** learning SQL on realistic clinical data.
* **Recruiters and hiring managers** looking for candidates with hands on OMOP experience.

## About the Author

Hi, I am Arshad, a data engineer with a passion for healthcare interoperability and open source. I have built this project to demonstrate my ability to:

* Design and implement complex ETL pipelines.
* Work with clinical standards (OMOP, SNOMED, RxNorm, LOINC).
* Write efficient SQL for Microsoft SQL Server.
* Document and share data engineering work for the community.

I am currently open for full time remote or contract roles where I can apply these skills. If you like this project, please reach out.

**Contact me**  
Email: arshad@example.com (replace with real email)  
LinkedIn: [linkedin.com/in/arshad](https://linkedin.com/in/arshad)  
GitHub: [github.com/arshad](https://github.com/arshad)  
For urgent questions, feel free to open an issue on this repository or send a direct message.

## SOS / Ask for Help

If you run into trouble restoring the database or running the scripts, please open a GitHub issue with a clear description of the error. I try to respond within 48 hours.

## License

This project is licensed under the MIT License – you can use it freely for learning, prototyping, or even in production, but no warranty is provided.

## Official Links

* OMOP CDM specification: [https://ohdsi.github.io/CommonDataModel](https://ohdsi.github.io/CommonDataModel)
* Synthea project: [https://synthetichealth.github.io/synthea](https://synthetichealth.github.io/synthea)
* OHDSI community: [https://ohdsi.org](https://ohdsi.org)
* Athena vocabulary tool: [https://athena.ohdsi.org](https://athena.ohdsi.org)
* SQL Server downloads: [https://www.microsoft.com/en us/sql server/sql server downloads](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)

## Star the Repository

If you find this project useful, please give it a star on GitHub. It helps others discover it and motivates me to keep improving the code.

Thank you for visiting, and happy querying
