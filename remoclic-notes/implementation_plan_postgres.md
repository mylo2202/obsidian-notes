# Implementation Plan - PostgreSQL Integration for Historical NetCDF Data

The goal is to rewrite the NetCDF repository to retrieve and store all NetCDF data files in a PostgreSQL database, rather than serving only the latest discovered file directly from memory/disk fallback. This enables query operations over the entire historical range of data and removes direct web request latency on data queries.

## User Review Required

> [!IMPORTANT]
> Since PostgreSQL is not currently configured in the codebase, we will introduce:
> - `psycopg2-binary` or `asyncpg` for database connectivity.
> - PostgreSQL database connection configuration variables in `.env` and `app/core/config.py`.
> - A migration/initialization script or automatic table creation on startup.

## Open Questions

> [!NOTE]
> 1. Do you have a running PostgreSQL database connection URI we should use for development, or should we default to a standard local PostgreSQL docker container setup?
> 2. How frequently should the background task check for new NetCDF files? (e.g., daily, hourly, or weekly?)

## Proposed Changes

### Database Setup and Configuration

#### [NEW] [database.py](file:///home/mylo2202/remoclic-v2-be/app/core/database.py)
- Establish connection pool and session generator using SQLAlchemy (or standard database client).
- Manage database connection lifecycle.

#### [MODIFY] [config.py](file:///home/mylo2202/remoclic-v2-be/app/core/config.py)
- Add `DATABASE_URL` setting with a default fallback (e.g. `postgresql://postgres:postgres@localhost:5432/remoclic`).

#### [MODIFY] [requirements.txt](file:///home/mylo2202/remoclic-v2-be/requirements.txt)
- Add `sqlalchemy` and `psycopg2-binary` (or `asyncpg`).

### Database Model and Ingestion Task

#### [NEW] [models.py](file:///home/mylo2202/remoclic-v2-be/app/models/forecast.py)
- Define `DroughtForecast` database model:
  - `id` (Primary Key, Serial)
  - `ref_date` (Date)
  - `lat` (Numeric/Float)
  - `lon` (Numeric/Float)
  - `timescale` (Float)
  - `lead` (Integer)
  - `mild` (Float)
  - `mord` (Float)
  - `seve` (Float)
- Add indexes on `(lat, lon)` and `ref_date`.

#### [NEW] [ingestion.py](file:///home/mylo2202/remoclic-v2-be/app/services/ingestion.py)
- Implement logic to:
  - Fetch base URL directory listing.
  - Parse available YYYYMM subdirectories.
  - Check which `ref_date`s are already in the database.
  - For missing `ref_date`s: download the corresponding NetCDF file, parse using `xarray`, and insert all data points as rows.
- Expose a background task/scheduler function.

### Repository and Service Layer Rewrite

#### [MODIFY] [netcdf_repository.py](file:///home/mylo2202/remoclic-v2-be/app/repositories/netcdf_repository.py)
- Refactor the class to query the database instead of accessing raw NetCDF files.
- Add queries to get the historical data or a specific point's forecast.

#### [MODIFY] [dataset_service.py](file:///home/mylo2202/remoclic-v2-be/app/services/dataset_service.py)
- Adapt mapping logic to structure database results into the response format required by `/forecast` and `/` endpoints.

## Verification Plan

### Automated Tests
- Run ingestion function against a test/mock local database and verify the number of rows inserted matches expectations.
- Run endpoint tests to verify API response formats.

### Manual Verification
- Deploy and trigger the ingestion task manually, checking database rows count and logs.
- Call the `/forecast` endpoint with coordinate values and inspect the response.
