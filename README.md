![Sheen Banner](https://raw.githubusercontent.com/74Thirsty/74Thirsty/main/assets/easybay.svg)

---

# eBay PowerLister 🚀

**A fast, reliable toolkit for power sellers to create, manage, and optimize large batches of eBay listings.**
Designed for automation-first workflows: CSV/JSON imports, templated listings, image hosting, scheduled publishes, variations support, analytics — all backed by robust eBay API integrations and forensic logging.

⚠️ **Important:** PowerLister interacts with the eBay API. You must comply with eBay’s API terms of service and applicable listing laws in your jurisdiction. Use only with accounts you control or have explicit written permission to manage.

---

## 📑 Table of Contents

1. [Features](#features)
2. [Quick Demo](#quick-demo)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Usage](#usage)
6. [Templates & CSV/JSON Schema](#templates--csvjson-schema)
7. [Scheduling & Execution](#scheduling--execution)
8. [Monitoring & Logs](#monitoring--logs)
9. [Testing / Sandbox](#testing--sandbox)
10. [Security & Best Practices](#security--best-practices)
11. [Troubleshooting](#troubleshooting)
12. [Roadmap](#roadmap)
13. [Contributing](#contributing)
14. [License](#license)

---

## ✨ Features

* Bulk create, update, relist, and end eBay listings via CSV/JSON batches.
* Powerful template engine for titles, descriptions, shipping, and item specifics (Jinja-style syntax).
* Variation & inventory management (multi-SKU, multi-variation).
* Image hosting / CDN integration with auto-optimization.
* Pricing strategies: fixed, calculated, markdown, or market-suggested.
* Scheduler for timed publishes, relists, and delists.
* Multi-account support with isolated credentials.
* Robust retry/exponential backoff + forensic logging of every API call.
* Dry-run mode and preview export (“what would be posted”).
* Analytics: impressions, sell-through rates, performance tracking.
* CLI + REST API + Docker-ready.

---

## ⚡ Quick Demo

Import & dry-run validate a CSV:

```bash
powerlister import --file products.csv --account my-ebay-account --dry-run
```

Publish a validated batch:

```bash
powerlister publish --batch-id 20250921-001 --account my-ebay-account
```

Preview a single templated listing:

```bash
powerlister preview --template vintage-watch.tpl --vars '{"title":"Rolex 1970","price":1299.99}'
```

---

## 🔧 Installation

**Requirements**

* Python 3.10+ (3.11 recommended)
* PostgreSQL or SQLite
* Redis (optional, for scheduler)
* Docker / docker-compose (recommended for production)

From **PyPI**:

```bash
pip install ebay-powerlister
```

From **source**:

```bash
git clone https://github.com/you/ebay-powerlister.git
cd ebay-powerlister
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python setup.py install
```

With **Docker**:

```bash
docker compose up -d
docker compose exec powerlister-app powerlister migrate
```

---

## ⚙️ Configuration

Environment variables or `.env`:

```ini
POWERLISTER_ENV=production
DATABASE_URL=postgresql://user:pass@db:5432/powerlister
REDIS_URL=redis://redis:6379/0
EBAY_CLIENT_ID=YOUR_EBAY_CLIENT_ID
EBAY_CLIENT_SECRET=YOUR_EBAY_CLIENT_SECRET
EBAY_REDIRECT_URI=https://yourapp.example.com/oauth/callback
DEFAULT_ACCOUNT=my-ebay-account
IMAGE_CDN_URL=https://cdn.example.com
LOG_LEVEL=INFO
```

Secrets should be kept in a **secret manager** (Vault, AWS Secrets Manager, Keyring).

---

## 🖥 Usage

**CLI Commands**

* `powerlister init` — initialize DB & templates
* `powerlister import` — import CSV/JSON into batch
* `powerlister validate` — validate batch schema/templates
* `powerlister publish` — publish or schedule batch
* `powerlister preview` — render single listing preview
* `powerlister status` — get status + logs for batch
* `powerlister analytics` — fetch metrics

**REST API** (JWT/API key auth)

* `POST /api/v1/batches` — upload batch (CSV/JSON)
* `POST /api/v1/batches/:id/publish` — publish/schedule
* `GET /api/v1/batches/:id` — batch status & logs
* `GET /api/v1/previews` — preview templated listing

Docs available at `/docs` (dev mode).

---

## 📄 Templates & Schema

**CSV Required Columns**

* `sku` (required)
* `title` or template vars
* `description` or template ref
* `price`, `currency`
* `condition`
* `category_id`
* `image_urls`
* `stock_qty`
* Profiles: `shipping_profile`, `return_profile`, `payment_profile`

**Example Template (Jinja-like):**

```tpl
{{ brand }} {{ model }} - {{ condition }}
{% if features %}
Features:
{% for f in features %}
- {{ f }}
{% endfor %}
{% endif %}
Ships from: {{ ship_from }}
```

---

## ⏱ Scheduling & Execution

* Scheduler uses Redis + RQ (or Celery).
* Jobs are durable and survive restarts.
* Execution pipeline runs in idempotent steps:

  1. Upload images
  2. Create draft listing
  3. Attach variations / inventory
  4. Publish
* Per-step logging and retry with backoff.

---

## 📊 Monitoring & Logs

* **Forensic logging**: full request/response (without secrets).
* Logs: `logs/powerlister.log`, plus per-batch logs.
* **Metrics**: Prometheus exporter at `/metrics`.
* **Alerts**: webhook/email hooks on failures.

---

## 🧪 Testing / Sandbox

* Full eBay **Sandbox** support.
* Dry-run mode simulates every step.
* Always validate before live publish.

---

## 🔒 Security & Best Practices

* Store **secrets in a vault**, never Git.
* Respect **eBay API rate limits**.
* Rotate OAuth tokens regularly.
* Only manage accounts you legally control.

---

## 🛠 Troubleshooting

* `401 Unauthorized` → re-run OAuth, check credentials.
* `429 Too Many Requests` → reduce concurrency.
* Image errors → check CDN credentials and source URLs.
* Schema errors → run `powerlister validate`.

---

## 🗺 Roadmap

* AI-driven price optimization.
* Auto-relist with smart repricing.
* Shipping label integrations.
* Multi-currency + VAT support.
* Marketplace expansion: Etsy, Shopify.

---

## 🤝 Contributing

1. Open an issue.
2. Branch `feat/...` or `fix/...`.
3. Write tests + update docs.
4. Submit PR.

Coding standards: **Black**, **ESLint**, pre-commit hooks.

---

## 📜 License

**Private License — Not for Redistribution**
This software is proprietary and provided under a private license.
You may not copy, distribute, sublicense, or sell this software without explicit written consent from the author.

For commercial licensing inquiries, contact the repository owner.

---

## ⚠️ Final Note

PowerLister is powerful. Automating listings can scale mistakes as quickly as profits. **Always dry-run, always validate, always log.**
