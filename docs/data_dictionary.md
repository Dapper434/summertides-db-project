# SummerTides Festival — Data Dictionary

Schema owner: Jairus - (Database Architect). Reflects `database/02_create_tables.sql`
and `database/04_constraints.sql`.

---

## attendees
People registered for the festival.

| Column             | Type          | Constraints                          | Notes                          |
|--------------------|---------------|---------------------------------------|---------------------------------|
| attendee_id        | INTEGER PK AUTOINC        | PRIMARY KEY                           |                                 |
| first_name         | VARCHAR(50)   | NOT NULL                              |                                 |
| last_name          | VARCHAR(50)   | NOT NULL                              |                                 |
| email               | VARCHAR(100)  | NOT NULL, UNIQUE, CHECK (looks like email) |                          |
| phone              | VARCHAR(20)   | nullable                              | some attendees skip this       |
| age                | INT           | CHECK (age >= 0)                      |                                 |
| city               | VARCHAR(50)   | NOT NULL                              |                                 |
| registration_date  | DATE          | NOT NULL, DEFAULT CURRENT_DATE        |                                 |

## artists
Performers booked for the festival.

| Column       | Type         | Constraints  | Notes |
|--------------|--------------|--------------|-------|
| artist_id    | INTEGER PK AUTOINC       | PRIMARY KEY  |       |
| artist_name  | VARCHAR(100) | NOT NULL     |       |
| genre        | VARCHAR(50)  | NOT NULL     |       |
| country      | VARCHAR(50)  | NOT NULL     | used to classify Local vs International |

## stages
Physical performance stages.

| Column     | Type        | Constraints            | Notes |
|------------|-------------|--------------------------|-------|
| stage_id   | INTEGER PK AUTOINC      | PRIMARY KEY              |       |
| stage_name | VARCHAR(50) | NOT NULL, UNIQUE         |       |
| capacity   | INT         | NOT NULL, CHECK (> 0)    |       |

## vendors
Food, drink, and merchandise vendors.

| Column      | Type          | Constraints                     | Notes |
|-------------|---------------|-----------------------------------|-------|
| vendor_id   | INTEGER PK AUTOINC        | PRIMARY KEY                       |       |
| vendor_name | VARCHAR(100)  | NOT NULL                          |       |
| category    | VARCHAR(50)   | NOT NULL                          |       |
| rating      | DECIMAL(2,1)  | CHECK (0–5)                       |       |

## sponsors
Companies funding the festival / individual stages.

| Column               | Type          | Constraints                | Notes |
|----------------------|---------------|-----------------------------|-------|
| sponsor_id           | INTEGER PK AUTOINC        | PRIMARY KEY                 |       |
| sponsor_name         | VARCHAR(100)  | NOT NULL, UNIQUE            |       |
| contribution_amount  | DECIMAL(10,2) | NOT NULL, CHECK (>= 0)      |       |

## tickets
Tickets purchased by attendees.

| Column        | Type          | Constraints                                         | Notes |
|---------------|---------------|-------------------------------------------------------|-------|
| ticket_id     | INTEGER PK AUTOINC        | PRIMARY KEY                                            |       |
| attendee_id   | INT           | NOT NULL, FK -> attendees(attendee_id), ON DELETE CASCADE |    |
| ticket_type   | VARCHAR(20)   | NOT NULL, CHECK IN ('Standard','VIP','Backstage')      |       |
| price         | DECIMAL(10,2) | NOT NULL, CHECK (>= 0)                                 |       |
| festival_day  | INT           | NOT NULL, CHECK IN (1,2,3)                             |       |
| purchase_date | DATE          | NOT NULL, DEFAULT CURRENT_DATE                         |       |

## performances
Which artist plays which stage, and when.

| Column          | Type  | Constraints                                                | Notes |
|-----------------|-------|--------------------------------------------------------------|-------|
| performance_id  | INTEGER PK AUTOINC| PRIMARY KEY                                                   |       |
| artist_id       | INT   | NOT NULL, FK -> artists(artist_id), ON DELETE CASCADE          |       |
| stage_id        | INT   | NOT NULL, FK -> stages(stage_id), ON DELETE CASCADE             |       |
| festival_day    | INT   | NOT NULL, CHECK IN (1,2,3)                                     |       |
| start_time      | TIME  | NOT NULL                                                       |       |
| end_time        | TIME  | NOT NULL, CHECK (end_time > start_time)                         |       |
|                 |       | UNIQUE (artist_id, festival_day, start_time) — no double-booking an artist |
|                 |       | UNIQUE (stage_id, festival_day, start_time) — no double-booking a stage |

## sales
Purchases attendees make from vendors.

| Column       | Type          | Constraints                                            | Notes |
|--------------|---------------|-----------------------------------------------------------|-------|
| sale_id      | INTEGER PK AUTOINC        | PRIMARY KEY                                                 |       |
| vendor_id    | INT           | NOT NULL, FK -> vendors(vendor_id), ON DELETE CASCADE        |       |
| attendee_id  | INT           | NOT NULL, FK -> attendees(attendee_id), ON DELETE CASCADE     |       |
| sale_amount  | DECIMAL(10,2) | NOT NULL, CHECK (>= 0)                                       |       |
| sale_date    | DATE          | NOT NULL, DEFAULT CURRENT_DATE                                |       |

## stage_sponsors (junction table)
Many-to-many link between stages and sponsors.

| Column      | Type | Constraints                                             | Notes |
|-------------|------|-------------------------------------------------------------|-------|
| stage_id    | INT  | PK (composite), FK -> stages(stage_id), ON DELETE CASCADE     |       |
| sponsor_id  | INT  | PK (composite), FK -> sponsors(sponsor_id), ON DELETE CASCADE  |       |

---

## Relationships summary

- attendees (1) → (M) tickets
- attendees (1) → (M) sales
- vendors (1) → (M) sales
- artists (1) → (M) performances
- stages (1) → (M) performances
- stages (M) ↔ (M) sponsors, via `stage_sponsors`