When merge `develop` into `main` and deploy to production server, create a new `.env` file on that server with these production values:

| Variable | Develop Branch (`.env`) | Main Branch (Production `.env`) |
| :--- | :--- | :--- |
| **`ENVIRONMENT`** | `development` | `production` |
| **`DEBUG`** | `True` | `False` |
| **`BACKEND_CORS_ORIGINS`** | `"http://localhost:4200"` | `"http://<PROD_IP>:8080"` |
| **`LOG_LEVEL`** | `DEBUG` | `INFO` (better for performance) |
