# Walkthrough - PostgreSQL & Historical NetCDF Data Ingestion

We rewritten the data access layer to retrieve and store all monthly NetCDF files from the remote server in a PostgreSQL database (flat table structure), replacing the previous code that only retrieved the latest file on-demand and kept it in memory.

## Changes Made

### 1. Database Model & Engine
- Created [database.py](file:///home/mylo2202/remoclic-v2-be/app/core/database.py) to manage SQLAlchemy connection pooling, engine creation, and session lifecycle.
- Created [forecast.py](file:///home/mylo2202/remoclic-v2-be/app/models/forecast.py) containing the `DroughtForecast` database model. It uses a flat layout with composite indexes on `(lat, lon)` and `(lat, lon, ref_date)` to optimize spatial-temporal lookups.

### 2. Ingestion Service & Scheduling
- Implemented [ingestion.py](file:///home/mylo2202/remoclic-v2-be/app/services/ingestion.py) to scan the remote directory listing, identify missing months, download the files, parse them using `xarray`, and bulk insert them into PostgreSQL in batches of 5000 records.
- Performs a transaction rollback (`db.rollback()`) on the database session if an ingestion failure occurs for any single month, preventing transaction corruption and allowing subsequent months to continue ingestion.
- Handles dynamic dimension orders (using `.transpose()`) and different grid sizes and time horizons across NetCDF files seamlessly.
- Configured APScheduler in [main.py](file:///home/mylo2202/remoclic-v2-be/app/main.py) to run the ingestion job daily, as well as once immediately on startup.

### 3. Repository & Service Layer Refactoring
- Updated [netcdf_repository.py](file:///home/mylo2202/remoclic-v2-be/app/repositories/netcdf_repository.py) to perform database queries instead of file operations. Includes a Manhattan distance lookup to find the nearest gridded coordinates.
- Updated [dataset_service.py](file:///home/mylo2202/remoclic-v2-be/app/services/dataset_service.py) to query the repository. Extended `get_forecast` to accept an optional `ref_date_str` parameter to allow querying historical months.
- Updated [dataset.py](file:///home/mylo2202/remoclic-v2-be/app/api/routes/dataset.py) router to accept the `ref_date` query parameter.

## Testing & Validation

We validated the logic using a local SQLite database override (reproducing the exact pipeline in a test script):
- Successfully scanned the remote site and discovered **28 monthly subdirectories** (from `2024-02-01` to `2026-05-01`).
- Successfully downloaded and parsed files with both 3-month and 6-month forecast horizons, and varying grid spatial resolutions.
- Ingested **1,187,748 forecast records** successfully.
- Verified that requesting `/forecast` without parameters correctly retrieves the latest forecast (`2026-05-01`), and providing `ref_date=2026-01-01` correctly retrieves the corresponding historical forecast.
