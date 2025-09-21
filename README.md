#eBay PowerLister

A fast, reliable toolkit for power sellers who want to create, manage, and optimize large batches of eBay listings.
Designed for automation-first workflows: CSV/JSON imports, templated listings, image hosting, scheduled publishes, variations support, and analytics — all backed by eBay API integrations and sane retry/forensic logging.

⚠️ Important: This tool interacts with the eBay API. You must comply with eBay’s API terms of service and applicable listing laws in your jurisdiction. Use only with accounts you control or have written permission to manage.

⸻

Table of contents
	1.	Features
	2.	Quick demo
	3.	Installation
	4.	Configuration
	5.	Usage
	6.	Templates & CSV/JSON schema
	7.	Scheduling & Execution
	8.	Monitoring & Logs
	9.	Testing / Sandbox
	10.	Security & Best Practices
	11.	Troubleshooting
	12.	Roadmap
	13.	Contributing
	14.	License

⸻

Features
	•	Bulk create, update, relist, and end eBay listings via CSV/JSON batches.
	•	Template engine for titles, descriptions, shipping, and item specifics (Jinja-like syntax).
	•	Variation & inventory management (multi-SKU, multi-variation support).
	•	Image hosting / CDN integration and automatic image optimization.
	•	Price strategies: fixed, calculated, dynamic markdowns, market-backed suggestions.
	•	Scheduler for timed publishes, relists and delist windows.
	•	Multi-account support (separate credentials for each eBay account).
	•	Robust retry/exponential backoff with per-step forensic logging.
	•	Dry-run mode and preview export (what would be posted).
	•	Analytics: listing performance, impressions, sell-through rate (optional eBay analytics integration).
	•	CLI + REST API + Docker-ready.

⸻

Quick demo

Create and publish a batch from a CSV:

# run a dry-run to validate a CSV
powerlister import --file products.csv --account my-ebay-account --dry-run

# publish validated batch
powerlister publish --batch-id 20250921-001 --account my-ebay-account

Preview a single templated listing:

powerlister preview --template vintage-watch.tpl --vars '{"title":"Rolex 1970","price":1299.99}'


⸻

Installation

Requirements
	•	Python 3.10+ (recommended 3.11)
	•	PostgreSQL or SQLite (for small single-user setups)
	•	Redis (optional, for job queue & scheduler)
	•	Docker / docker-compose (recommended for production)

From pip (quick)

pip install ebay-powerlister

From source

git clone https://github.com/you/ebay-powerlister.git
cd ebay-powerlister
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python setup.py install

Docker

docker-compose.yml (example service names: powerlister-app, db, redis)

docker compose up -d
# run migrations
docker compose exec powerlister-app powerlister migrate


⸻

Configuration

Place configuration in environment variables or a .env file (example below). Secrets should be stored in a secret manager (Vault / AWS Secrets Manager / Keyring) in production.

.env example:

POWERLISTER_ENV=production
DATABASE_URL=postgresql://user:pass@db:5432/powerlister
REDIS_URL=redis://redis:6379/0
EBAY_CLIENT_ID=YOUR_EBAY_CLIENT_ID
EBAY_CLIENT_SECRET=YOUR_EBAY_CLIENT_SECRET
EBAY_REDIRECT_URI=https://yourapp.example.com/oauth/callback
DEFAULT_ACCOUNT=my-ebay-account
IMAGE_CDN_URL=https://cdn.example.com
LOG_LEVEL=INFO

OAuth: Follow eBay developer docs to register your application and obtain Client ID/Secret. Store those securely.

⸻

Usage

CLI

powerlister CLI supports subcommands:
	•	powerlister init — initialize DB + sample templates
	•	powerlister import --file FILE --account ACCOUNT [--batch-name NAME] [--dry-run] — import CSV/JSON into a batch
	•	powerlister validate --batch-id BATCH — validate a batch against schema & templates
	•	powerlister publish --batch-id BATCH --account ACCOUNT [--schedule "2025-10-01T12:00:00Z"] — publish batch or schedule it
	•	powerlister preview --template NAME --vars JSON — render a single listing preview
	•	powerlister status --batch-id BATCH — get status + logs for batch
	•	powerlister analytics --account ACCOUNT --range 30d — fetch recent metrics

Example: publish a CSV immediately:

powerlister import --file listings.csv --account my-ebay-account
powerlister validate --batch-id 20250921-002
powerlister publish --batch-id 20250921-002 --account my-ebay-account

REST API

The service exposes a small API (auth via JWT / API key):
	•	POST /api/v1/batches — upload batch (CSV/JSON)
	•	POST /api/v1/batches/:id/publish — publish or schedule
	•	GET /api/v1/batches/:id — status & logs
	•	GET /api/v1/previews — get templated preview

Auth and docs (OpenAPI) are exposed at /docs when running in dev mode.

⸻

Templates & CSV/JSON schema

Template example (title + description):

{{ brand }} {{ model }} - {{ condition }}
{{#if features}}
Features:
{% for f in features %}
- {{ f }}
{% endfor %}
{% endif %}

Ships from: {{ ship_from }}

Minimal CSV columns
	•	sku (required)
	•	title or template variables to render title
	•	description (or point to template)
	•	price (number)
	•	currency (default USD)
	•	condition (New/Used/Like New etc)
	•	category_id (eBay numeric category)
	•	image_urls (pipe | separated)
	•	variation_sku / variation_name / variation_options — for variations
	•	stock_qty
	•	shipping_profile / return_profile / payment_profile — reference to configured profiles

JSON schema and a sample CSV template are included in docs/schema/.

⸻

Scheduling & Execution
	•	The scheduler uses Redis + RQ (or Celery) to queue publish jobs.
	•	Scheduled jobs persist in DB and are resilient to restarts.
	•	Execution engine performs the publish in small idempotent steps:
	1.	upload images (if needed)
	2.	create draft listing
	3.	attach variations / inventory
	4.	publish/list
	•	Each step is logged; failures trigger retry with exponential backoff. You can configure retry counts in CONFIG.RETRIES.

⸻

Monitoring & Logs
	•	Forensic logging: every API call to eBay is logged with request/response metadata (no secret tokens). Logs can be exported for auditing.
	•	Log files: logs/powerlister.log (rotating), and per-batch logs in logs/batches/.
	•	Metrics: Prometheus exporter included (metrics endpoint /metrics).
	•	Alerts: Configure webhook/email for critical failures (publish failed, rate-limited, auth expired).

⸻

Testing / Sandbox
	•	eBay Sandbox support is implemented. Use separate sandbox credentials and POWERLISTER_ENV=sandbox.
	•	Dry-run mode simulates every step and outputs the API payloads that would be sent to eBay. Always validate batches in dry-run before publishing live.

⸻

Security & Best Practices
	•	Never commit EBAY_CLIENT_SECRET or access tokens to Git. Use secret managers.
	•	Rate limits: obey eBay API rate limiting. The default client implements adaptive throttling; do not disable it.
	•	Revoke and rotate OAuth tokens periodically (and on role changes).
	•	Only operate listings for accounts you control. Automating other users’ accounts without explicit written consent may violate law and eBay policies.

⸻

Troubleshooting
	•	401 Unauthorized — re-run OAuth flow, check redirect URLs and valid client ID/secret.
	•	429 Too Many Requests — inspect throttling metrics; reduce concurrency or increase backoff.
	•	Image upload issues — confirm CDN credentials and that source URLs are reachable.
	•	Schema errors during import — run powerlister validate to get exact CSV row errors.

If you need a forensic-style error trace for a failed batch:

powerlister status --batch-id 20250921-002 --verbose
# then inspect logs:
less logs/batches/20250921-002.log


⸻

Roadmap
	•	Improved AI-driven price optimization (market scraping + ML suggestions).
	•	Smart relisting strategies (auto-reprice on low views).
	•	Built-in shipping label purchasing integrations.
	•	Multi-currency and VAT support.
	•	Marketplace expansion: Etsy / Shopify connectors.

⸻

Contributing

Contributions welcome — please follow these guidelines:
	1.	Open an issue describing the feature or bug.
	2.	Create a branch feat/… or fix/….
	3.	Write tests and update docs.
	4.	Submit a PR referencing the issue.

Code style: Black for Python, ESLint for JS, and pre-commit hooks are enabled.

⸻

License

MIT License — see LICENSE in the repo.

⸻

A final note

This tool is powerful and intended for responsible sellers. Automating listings can scale mistakes as quickly as profits — validate every batch in dry-run mode and keep thorough logs. If you want, I can generate a starter products.csv template, a sample listing template, or an example Docker Compose file for production deployment. Which one should I produce first?
