# Shopping Detective — Group 6

**Theme:** online price comparison for smart, budget-aware shopping. Shopping Detective
searches several online stores for a product, shows where it is cheapest, tracks how prices
move, alerts users when a price drops, and tells them how much they saved against their
shopping budget.

## What it does

When a user searches for a product, for example "iPhone 17", the app shows:

- the product name and an image
- the online stores that sell it, with each store's price and shipping cost
- the best total price (price + shipping) and a link to that store
- the price history of the product and whether now is a good time to buy

Logged-in users can also set price alerts, build a shopping list and find the cheapest way to
buy it, record what they bought and follow their savings against a monthly budget.

## Modules

Each member owns one module: its own main entity, Python package, API router
(`/api/<module>`), tests and page in the frontend.

| Module | Main entity (CRUD) | Calculated result | Owner |
|---|---|---|---|
| `users` — users and price alerts | `User`, `PriceAlert` (user, product, target price) | which alerts are triggered by the current best price, and how far each product is from its target | Blas G. |
| `stores` — stores and price acquisition | `Store` (name, website, country, shipping cost, free-shipping threshold) | total cost per store (price + shipping) and each store's share of best prices | Sara F. |
| `products` — catalogue and normalization | `Product` (name, brand, EAN, category, image URL) | matching the same product across stores into one normalized product; search ranking | Jose Luis S. |
| `prices` — price offers and history | `PriceOffer` (product, store, price, observed at, URL) | current lowest price, 30-day min / max / average and % change | Tomas M. |
| `baskets` — shopping lists | `ShoppingList`, `ShoppingListItem` | cheapest basket: total in one store vs. split across stores, including shipping | Susana P. |
| `purchases` — purchases and budget | `Purchase` (user, product, store, price paid, date), `Budget` (user, month, amount) | savings vs. average market price; budget spent and remaining this month | Emile H. |

`users` is the user-management module: every other module that holds personal data
(alerts, shopping lists, purchases, budgets) links it to a `user_id`.

## Cross-module feature: the Savings Dashboard

One page per user that brings all the modules together:

| Dashboard block | Uses |
|---|---|
| Money saved this month, and budget spent / remaining | `purchases`, which calls `prices` for the average market price of each item bought |
| Price alerts triggered today | `users`, which calls `prices` for the current best price |
| Cheapest way to buy my shopping list, and in which stores | `baskets`, which calls `prices` and `stores` (shipping costs) |
| Price trend of the products I follow | `prices` and `products` |

Modules talk to each other only through their API (`/api/<module>`), never through each
other's tables.

## Shared jobs

| Member | Shared job | Responsible for |
|---|---|---|
| Emile H. | Continuous integration | the GitHub Actions workflow: Ruff and the tests on every pull request |
| Blas G. | Containers and configuration | the Dockerfile, Docker Compose, settings from environment variables, `.env.example` |
| Jose Luis S. | Cloud deployment | the Azure resources, the deployment and its cost |
| Susana P. | Frontend shell and dashboard | the common layout, navigation and the Savings Dashboard |
| Tomas M. | Database | the shared database setup, the models' common base and the sample data |
| Sara F. | Documentation and board | the README, the architecture diagram, and the issues and project board |

With six members we cover six of the seven shared jobs. Code review is not one person's job:
every member reviews, and the rules are in "How we work" below.

## Conventions agreement

- **Stack:** Python 3.12, FastAPI, SQLAlchemy 2, PostgreSQL, pytest, Ruff, Docker Compose,
  Azure; the frontend pages are served by the same application. Same stack as the labs.
- **Layout:** one package per module under `app/<module>/` (`models.py`, `schemas.py`,
  `service.py`, `router.py`), tests in `tests/<module>/`.
- **Names:** packages and tables in lowercase plural English (`price_offers`), classes in
  singular CamelCase (`PriceOffer`).
- **API:** every route under `/api/<module>`, JSON, plural nouns
  (`GET /api/prices/offers?product_id=12`); errors as `{"detail": "..."}` with the right status code.
- **Money:** `Decimal` in Python and `NUMERIC(12, 2)` in the database, never a float. Every
  amount has a currency (`EUR` by default); in JSON, amounts are strings (`"999.00"`).
- **Dates:** timestamps in UTC as `timestamptz`, ISO 8601 in JSON
  (`2026-11-23T12:00:00Z`); calendar dates as `date` (`2026-11-23`).
- **Users:** every personal record has a `user_id` that refers to `users`.
- **External data:** store data comes in through connectors in `stores`. Tests and CI never
  call real stores: they use the sample data and recorded responses.

## How we work
- One repository, `main` protected: no direct pushes, every change through a pull request.
- One short-lived branch per issue, named `<type>/<issue>-<short-name>`
  (`feat/12-price-history`, `docs/1-m1-readme`).
- Small pull requests that link their issue (`Closes #12`), approved by at least one other
  member, merged into `main` with a merge commit. The CI must pass before merging.
- Bring `main` into your branch often (`git pull origin main`).
- Trunk-based development, not Gitflow: one long-lived branch (`main`) and short branches
  merged within a few days, so modules are integrated continuously and conflicts stay small.
- Every task is a GitHub issue, assigned to one member, on the project board
  (To do → In progress → In review → Done).
- Milestones are marked with a tag on the reviewed commit of `main` (`m1`, `m2`, …).

## How to run it

## Team

Sara F. · Jose Luis S. · Tomas M. · Blas G. · Susana P. · Emile H.
