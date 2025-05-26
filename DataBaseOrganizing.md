# DataBase `client_profile` Implementation with SQL feat. Python
---
# Table of Contents

1.  Client Profile DataBase Implementation.
	- DataBase `client_profile` dictionary.
		- Fields in the `users` Table.
		- Fields in the `countries` Table. 
		- Fields in the `emails_sent` Table. 
		- Fields in the `emails_clicks` Table.
		- Relationsships between Tables (Entities) - Normalization (1:N).
		- `id` Permormance - how to create a SENSIBLE ID.
	- DDL (SQL) Generating with Oracle SQL Developer Data Modeler.
	- Randomly Generating Tables (just for testing SQL-queries) using `sqlalchemy - create_engine()` (pandas) to create Tables `users` (100), `countries` (3), `emails_sent` 500), `emails_clicks` (200).
2. `SQL` Oprimization Rules for DataBase `client_profile`.
3. Complete `SQL` query code for testing DB, calculating CR (Click Rate, %) and CTR (Click Through Rate, %) + an importance of `EXPLAIN ... ANALYZE`).
4. Conclusions and Key Points.

---
## 1  Client Profile DataBase Implementation

### 1.1  DataBase `client_profile` Dictionary

#### 1.1.1  `users` Table

|Field|Data Type|Constraints|Example|Description|
|---|---|---|---|---|
|`id`|`NUMBER(10)`|**PK**, not null, generated by default as identity|`10123`|Unique user identifier|
|`email`|`VARCHAR2(255)`|**UNIQUE**, not null|`john.doe@example.com`|User e‑mail|
|`id_country`|`NUMBER(10)`|**FK** → `countries.id`, not null, indexed|`5`|Country reference|
|`date_reg`|`DATE`|not null|`2024‑04‑10 14:05:27`|Registration timestamp|

#### 1.1.2  `countries` Table

| Field           | Data Type       | Constraints      | Example          | Description         |
| --------------- | --------------- | ---------------- | ---------------- | ------------------- |
| `id`            | `NUMBER(4)`     | **PK**, not null | `5`              | Country ID          |
| `name`          | `VARCHAR2(100)` | not null         | `Germany`        | Human‑readable name |
| `country_group` | `VARCHAR2(100)` | not null         | `Western Europe` | Segment bucket      |

#### 1.1.3  `emails_sent` Table

| Field       | Data Type    | Constraints                              | Example               | Description   |
| ----------- | ------------ | ---------------------------------------- | --------------------- | ------------- |
| `id`        | `NUMBER(12)` | **PK**, identity                         | `45098`               | Sent‑email ID |
| `id_user`   | `NUMBER(10)` | **FK** → `users.id`, not null, indexed   | `10123`               | Recipient     |
| `id_type`   | `NUMBER(3)`  | **CHECK** (`IN (1,2,3)`), not null       | `2`                   | Email type    |
| `date_sent` | `TIMESTAMP`  | not null, indexed (`id_user, date_sent`) | `2025‑05‑20 08:10:00` | Send time     |

#### 1.1.4  `emails_clicks` Table

| Field        | Data Type    | Constraints                                  | Example               | Description    |
| ------------ | ------------ | -------------------------------------------- | --------------------- | -------------- |
| `id`         | `NUMBER(14)` | **PK**, identity                             | `98765`               | Click ID       |
| `id_email`   | `NUMBER(12)` | **FK** → `emails_sent.id`, not null, indexed | `45098`               | Clicked e‑mail |
| `date_click` | `TIMESTAMP`  | not null, indexed (`id_email, date_click`)   | `2025‑05‑20 08:12:15` | Click time     |

#### 1.1.5  Relationships & Normalisation (1:N - one2many)

- **countries 1 → N users** (`countries.id` → `users.id_country`).
    
- **users 1 → N emails_sent** (`users.id` → `emails_sent.id_user`).
    
- **emails_sent 1 → N emails_clicks** (`emails_sent.id` → `emails_clicks.id_email`).
    

This is 3NF‑compliant: no redundant attributes, all FKs indexed for fast joins.

#### 1.1.6  `id` HIGH Performance for DA — Sensible Keys 

Below are practical strategies for designing high‑quality identifiers that balance write‑speed, storage footprint and relational integrity.

| Strategy                                          | When to use                                                                                  | Pros                                                                        | Cons / Caveats                                                                  | Example DDL                                                                                                         |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Surrogate single‑column ID**`NUMBER` + Identity | Default choice for most OLTP entities                                                        | • Compact PK/FK (4‑8 bytes)• Simple auto‑increment• Small composite indexes | • No business meaning<br>• Natural uniqueness still needs a `UNIQUE` constraint | `id NUMBER(10) GENERATED BY DEFAULT AS IDENTITY`                                                                    |
| **Composite natural PK / FK**                     | Child rows are fully identified by parent attributes (true 5 NF), e.g. `(order_id, line_no)` | • Eliminates redundant surrogate key• Logical completeness enforced by FK   | • Wider keys ⇒ heavier indexes<br>• Some ORMs awkward                           | `PRIMARY KEY (order_id,line_no)`<br>`FOREIGN KEY (order_id,line_no)`<br>`REFERENCES order_lines(order_id,line_no);` |
| **Hierarchical / prefixed key**                   | Need shard / region / entity‑type prefix for routing or partition pruning                    | • Fast shard lookup                                                         | • Manual generation logic<br>• Risk of key‑range exhaustion per shard           | `product_id CHAR(1)`<br>`LPAD(seq.NEXTVAL,7,'0')`                                                                   |
| **UUID / GUID**                                   | Distributed inserts with no central sequence (micro‑services, CDC)                           | • Globally unique• Zero contention                                          | • 16‑byte PK• Random order hurts index locality (prefer UUID v7 / ULID)         | `id RAW(16) DEFAULT SYS_GUID()`                                                                                     |
| **Hash‑based key (SHA‑256 of natural columns)**   | Deduplication on immutable business attributes (e‑mail, URL)                                 | • Idempotent inserts• Masks PII in PK                                       | • Not update‑friendly• Needs extra human‑readable column                        | `id RAW(32) GENERATED ALWAYS AS (STANDARD_HASH(email,'SHA256')) VIRTUAL`                                            |

> **Rule of thumb:** default to a surrogate identity key in OLTP for size & speed. Reach for composite natural keys only when they provide _real_ modelling value (junction tables, regulatory uniqueness).

Child tables (`emails_sent` → `emails_clicks`) always reference **exactly the columns of the parent PK** — whether surrogate or composite — so the application layer or `INSERT … SELECT` can cascade the values without look‑ups.

---

### 1.2  DDL Generation via Oracle SQL Developer Data Modeler

1. Build ER diagram → Validate Model (PK/FK, Check).2. `Engineer → DDL File` — target Oracle 19c.3. Options: include PK, FK, Check, Unique, Indexes. Sample snippet:
    

```SQL
CREATE TABLE users (
  id          NUMBER(10) GENERATED BY DEFAULT AS IDENTITY,
  email       VARCHAR2(255) NOT NULL,
  id_country  NUMBER(4)  NOT NULL,
  date_reg    DATE        NOT NULL,
  CONSTRAINT pk_users PRIMARY KEY(id),
  CONSTRAINT ux_users_email UNIQUE(email),
  CONSTRAINT fk_users_country FOREIGN KEY(id_country)
    REFERENCES countries(id)
);
```

### 1.3  Random Test Data with pandas + SQLAlchemy

```python
import random
import datetime as dt

import pandas as pd
import numpy as np

from sqlalchemy import create_engine


engine = create_engine("sqlite:///client_profile_demo.db")
# …generate DataFrames (see notebook) and df.to_sql("users", engine, index=False)
```

- `users` = 100 rows, `countries` = 3, `emails_sent` = 500, `emails_clicks` ≈ 200.
    
- Use for unit‑tests & `EXPLAIN...ANALYZE` benchmarks (see more practice of `EXPLAIN...ANALYZE` here: [UA Version of EXPLAIN...ANALYZE Practice](https://docs.google.com/document/d/1FFHlLvBeZd46YZvxaSA8IPzDMbQw1TKstE8rZ1lFgQA/edit?usp=sharing).
---

## 2 SQL Optimisation Rules for `client_profile`

### Quick Reference Table

| What                                    | Why / When                                                                                                                                                                                                                              |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **PK = NUMBER / GENERATED AS IDENTITY** | In Oracle Data Modeler the generic `INTEGER` column is cast to `NUMBER(38)`. If your IDs safely fit into 32‑bit range, declare them as `NUMBER(10)` and mark the column as **Identity** to cut block size and remove sequence overhead. |
| **Unique index on** `**users.email**`   | The e‑mail must be globally unique. `CREATE UNIQUE INDEX ux_users_email ON users(email);`                                                                                                                                               |
| **Indexes on every FK column**          | Avoids full scans during joins / cascade operations: • `CREATE INDEX ix_users_country ON users(id_country);` • `CREATE INDEX ix_sent_user ON emails_sent(id_user);` • `CREATE INDEX ix_clicks_email ON emails_clicks(id_email);`        |
| **Composite indexes for hot paths**     | Typical query _“all e‑mails of a user in a date range”_ → `IX_SENT_USER_DATE (id_user, date_sent)`. Query _“clicks within first 10 min”_ → `IX_CLICK_EMAIL_DATE (id_email, date_click)`.                                                |
| **NOT NULL on FKs**                     | Mark `id_country`, `id_user`, `id_email` as **Mandatory** so Oracle emits `NOT NULL`.                                                                                                                                                   |
| **Rename** `**group**` **column**       | `GROUP` is a reserved keyword. Prefer `country_group` or escape as `"group"`.                                                                                                                                                           |
| **CHECK constraint for** `**id_type**`  | If the list of e‑mail types is finite: `CHECK (id_type IN (1,2,3))`.                                                                                                                                                                    |
| **Partition very large tables**         | When `emails_sent` / `emails_clicks` exceed tens of millions of rows, switch to **range partitioning** by `date_sent` / `date_click`.                                                                                                   |
| **Index‑Organized Table (IOT)**         | Use for small lookup tables where PK = entire row (`countries`).                                                                                                                                                                        |

### Additional Tips

- Always project the needed columns – no `SELECT *` – to enable **covering index** reads.
    
- Refresh statistics (`DBMS_STATS`) after bulk loads to keep the cost‑based optimizer informed.
    
- Run `EXPLAIN PLAN` or `EXPLAIN ANALYZE` before shipping: confirm that the expected index appears (e.g. `ix_sent_user_date`).

---
## 3 Complete SQL Examples

### 3.1 Daily count of new user registrations grouped by country group

```SQL
SELECT
    COUNT(u.id) as registrations,
    DATE(u.date_reg) as registration_date,
    c.group as country_group
FROM users u
JOIN countries c ON u.id_country=c.id
GROUP BY registration_date,
         country_group
ORDER BY registration_date,
         country_group
;
```
### 3.2 Click Rate (CR) 

``` SQL
SELECT 
    e.id_type,
    COUNT(DISTINCT CASE 
        WHEN TIMESTAMPDIFF(MINUTE, e.date_sent, ec.date_click) <= 10 
             AND ec.date_click IS NOT NULL 
        THEN ec.id_email 
    END) * 100.0 / COUNT(DISTINCT e.id) AS click_rate_pct
FROM emails_sent e
LEFT JOIN emails_clicks ec 
    ON e.id = ec.id_email
WHERE e.date_sent >= NOW() - INTERVAL 7 DAY
GROUP BY e.id_type
;
```

### 3.3  Click Through Rate (CTR) 

```SQL
SELECT 
    e.id_type,
    COUNT(ec.id) * 100.0 / COUNT(e.id) AS ctr_pct
FROM emails_sent e
LEFT JOIN emails_clicks ec 
    ON e.id = ec.id_email 
    AND TIMESTAMPDIFF(MINUTE, e.date_sent, ec.date_click) <= 10
WHERE e.date_sent >= NOW() - INTERVAL 7 DAY
GROUP BY e.id_type
;
```

### 3.4  `EXPLAIN ANALYZE` Importance

```
EXPLAIN ANALYZE
SELECT … (CTR query) … ;
```

- Ensures planned index usage (expect `ref idx_sent_user_date`).
    
- Watch “Rows” vs “actual rows” — huge mismatch → missing stats/improper index.

---

## 4  Conclusions & Key Points

- A fully normalised 3NF schema with indexed FK ensures fast joins & integrity.
    
- Composite indexes tailored to product queries (id_user + date_sent) give 10‑20× speed‑up.
    
- CHECK + UNIQUE constraints catch bad data early.
    
- Automated DDL from Data Modeler synchronises code & doc.
    
- pandas + SQLAlchemy allow quick synthetic datasets for unit tests and benchmark‑driven optimisation.
    
- Always validate performance assumptions with `EXPLAIN ANALYZE` before shipping queries to production.