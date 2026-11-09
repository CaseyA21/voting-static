# Local development setup

How to run this project on your laptop: Python environment, tests, lint, DynamoDB Local, and the web app.


## Prerequisites

- **Python 3.12 or newer**
- **Java 17 or newer** (DynamoDB Local runs as a Java process)


Confirm with `python3 --version` and `java -version`.


## 1. Create a virtual environment

From the root of the project:

```bash
python3 -m venv .venv
source .venv/bin/activate
```


## 2. Install libraries

```bash
pip install -r requirements.txt
pip install -e .
```

The editable install (`-e .`) makes the `voting` package importable while you edit source under `src/voting/`.


## 3. Run tests

With the venv active:

```bash
pytest
```

Unit tests use **moto** (an in-process fake DynamoDB). You do **not** need DynamoDB Local running for `pytest`.


## 4. Run the linter

```bash
ruff check src tests scripts tests_ui
```

Fix reported issues before you commit.


## 5. Configure `.env`

Copy the example file and adjust if needed:

```bash
cp config/example.env .env
```

`.env` holds AWS credentials and DynamoDB configuration for the API. Keep it out of git (it is listed in `.gitignore`). The pages do not read `.env`.

Required keys are documented in `config/example.env`. For DynamoDB Local you will set the endpoint to `http://localhost:8000` and use dummy AWS keys.

`ui/config.js` sets `DEPLOY_URL`. Replace `YOUR_DNS_NAME` with your hostname before you upload the pages. Local runs and the acceptance tests still call `http://127.0.0.1:5000`, because those pages are served from `127.0.0.1`.


## 6. Download DynamoDB Local

This project runs [DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html) locally on your laptop. boto3 reaches it on port **8000**.

From the root of the project, download the zip into `db/` and unzip it there:

```bash
mkdir -p db
curl -o db/dynamodb_local_latest.zip \
  https://s3.us-west-2.amazonaws.com/dynamodb-local/v2.x/dynamodb_local_latest.zip
unzip db/dynamodb_local_latest.zip -d db
```

Leave the zip in `db/`. When you are done, that folder should also contain `DynamoDBLocal.jar` and a `DynamoDBLocal_lib` folder (among other files from the archive).

`DynamoDBLocal_lib` holds native libraries DynamoDB Local needs at startup. Keep the JAR and that folder together in `db/`. Any on-disk database files DynamoDB Local creates also land in `db/` when you start it from there. The project `.gitignore` ignores all of `db/`, so none of this is committed.


## 7. Start DynamoDB Local

From the project root:

```bash
cd db
java --enable-native-access=ALL-UNNAMED -jar DynamoDBLocal.jar -sharedDb -inMemory
```

Leave that process running. Stop the process (Ctrl+C) to shut down the database.

The command includes these switches:

- **`--enable-native-access=ALL-UNNAMED`** — Lets DynamoDB Local load its native SQLite library under current Java rules. Without this, Java prints a warning (and may refuse the load in a future release).
- **`-sharedDb`** — Use one shared database for every client, regardless of which access key or region string they send. Without this, each set of credentials gets a separate empty database, and it is easy to think your table “disappeared.”
- **`-inMemory`** — Keep data in memory only. When you stop the process, all data is gone, so you will need to create the table again (and re-seed if you use sample data) the next time you start.


DynamoDB Local listens on port **8000**. If something else on your laptop already uses 8000, free that port before starting.


## 8. Create the Polls table

With the venv active, DynamoDB Local up, and `.env` in place:

```bash
python scripts/create_table.py
```

This fails if the table already exists. To wipe and recreate:

```bash
python scripts/delete_table.py
python scripts/create_table.py
```

(`delete_table.py` prompts for confirmation; pass `-y` to skip the prompt.)

If DynamoDB Local was started with `-inMemory` and you restart it, the table is gone — run `create_table.py` again.


## 9. Seed sample data (optional)

The app works with an empty `Polls` table. Seed only if you want the sample polls from `data/sample-data.json`.

With the venv active, DynamoDB Local up, `.env` in place, and the table already created:

```bash
python scripts/seed.py
```

If the table is missing, the script exits with an error telling you to run `create_table.py` first.


## 10. Launch the API and the pages

With the venv active, DynamoDB Local running, and the `Polls` table present, start the API:

```bash
python -m voting.app
```

The API listens on port **5000**. In a second terminal, from the project root, serve the pages:

```bash
python -m http.server 8080 --directory ui
```

Open [http://127.0.0.1:8080/](http://127.0.0.1:8080/).


## 11. Run acceptance tests (Playwright)

Unit tests (`pytest`) use `moto` and do **not** need DynamoDB Local. Acceptance tests drive a real browser against the pages on port **8080** and the API on port **5000**.

One-time browser install:

```bash
playwright install chromium
```

Before running the acceptance tests:

* Start DynamoDB Local
* Create table
* Start the API (`python -m voting.app`)
* Start the page server (`python -m http.server 8080 --directory ui`)


With the database, the API, and the page server running, execute:

```bash
pytest tests_ui
```

The suite checks `http://127.0.0.1:5000/health` and `http://127.0.0.1:8080/`, and stops immediately if either is down. Individual tests reset table data as needed.
