# SmartPark deployment fix — multi-city parking

## Why only one city appears

The frontend does not hard-code one city. It loads every available parking row from:

`backend/api/parkings.php`

The supplied project contains the expanded Maharashtra dataset in:

`database/migration_maharashtra_locations.sql`

It adds 51 additional parking locations. If the deployed database has only the original row, that migration was not imported into the deployed database.

## XAMPP

1. Start Apache + MySQL.
2. Open phpMyAdmin.
3. Select `smartpark_db`.
4. Import `database/migration_parking_info.sql` if your old database is missing the parking_type / ev_charging / contact_phone columns.
5. Import `database/migration_maharashtra_locations.sql`.
6. Open:
   `http://localhost/SmartPark_Perfect_Cinematic_UPGRADED_FIXED/frontend/index.php`
7. Hard-refresh the browser.

Verify with:

```sql
SELECT COUNT(*) AS total_locations,
       COUNT(DISTINCT city) AS total_cities
FROM parkings
WHERE remaining_slots > 0;

SELECT city, COUNT(*) AS locations
FROM parkings
WHERE remaining_slots > 0
GROUP BY city
ORDER BY city;
```

## Render / production

Do NOT use `localhost` as the database host unless MySQL is actually running inside the same Render service.

Set these environment variables on the Render web service (or in the `.env` used by your deployment):

```text
DB_HOST=<your external MySQL host>
DB_USER=<your MySQL user>
DB_PASS=<your MySQL password>
DB_NAME=smartpark_db
DB_PORT=3306
```

Then import the database/migration SQL into that same production MySQL database.

The updated `backend/db_connect.php` now reads these values from environment variables while keeping XAMPP's `localhost/root/empty-password` defaults.

## Important

The deployment and the XAMPP installation are separate databases. Adding cities to phpMyAdmin on your computer does not add them to Render's database.

After deploying the code and importing the migration, test:

`/backend/api/parkings.php`

It should return JSON containing many `parkings` entries, not one.

## Security

The original uploaded `.env` contains credentials/API keys. Do not commit that file to GitHub. Rotate any credentials that have already been exposed and configure them as deployment environment variables.
