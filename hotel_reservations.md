# 🏨 HotelBook — SQL Reference Guide

---

## 📁 File Overview

| File | Purpose | Run Order |
|------|---------|-----------|
| `01_hotel_schema_data.sql` | Database, tables, seed data | First |
| `02_hotel_mcp_server.sql` | Stored procedures + MCP server | Second |

---

## 📄 File 1 — `01_hotel_schema_data.sql`

### What it does
Creates the entire data foundation — database, schema, 4 tables, and inserts sample data.

---

### Step 1 — Database & Schema

```sql
CREATE DATABASE IF NOT EXISTS HOTEL_DB;
CREATE SCHEMA IF NOT EXISTS HOTEL_DB.HOTEL;
```

| Object | Type | Purpose |
|--------|------|---------|
| `HOTEL_DB` | Database | Top-level container |
| `HOTEL_DB.HOTEL` | Schema | Groups all hotel objects |

---

### Step 2 — Table: GUESTS

```sql
CREATE OR REPLACE TABLE GUESTS (
    ID          NUMBER AUTOINCREMENT PRIMARY KEY,
    NAME        VARCHAR(100) NOT NULL,
    EMAIL       VARCHAR(100),
    PHONE       VARCHAR(20),
    NATIONALITY VARCHAR(50)
);
```

| Column | Type | Description |
|--------|------|-------------|
| ID | NUMBER | Auto-generated primary key |
| NAME | VARCHAR(100) | Full name of guest |
| EMAIL | VARCHAR(100) | Contact email |
| PHONE | VARCHAR(20) | Contact phone |
| NATIONALITY | VARCHAR(50) | Country of origin |

**Insert script:**
```sql
INSERT INTO GUESTS (NAME, EMAIL, PHONE, NATIONALITY) VALUES
    ('James Carter',   'james@email.com',  '+1-555-1001',  'American'),
    ('Sofia Reyes',    'sofia@email.com',  '+63-917-2002', 'Filipino'),
    ('Ahmed Al-Farsi', 'ahmed@email.com',  '+971-50-3003', 'Emirati'),
    ('Yuki Tanaka',    'yuki@email.com',   '+81-90-4004',  'Japanese'),
    ('Emma Dubois',    'emma@email.com',   '+33-6-5005',   'French');
```

**Sample rows inserted:**
| ID | NAME | EMAIL | NATIONALITY |
|----|------|-------|-------------|
| 1 | James Carter | james@email.com | American |
| 2 | Sofia Reyes | sofia@email.com | Filipino |
| 3 | Ahmed Al-Farsi | ahmed@email.com | Emirati |
| 4 | Yuki Tanaka | yuki@email.com | Japanese |
| 5 | Emma Dubois | emma@email.com | French |

---

### Step 3 — Table: ROOM_TYPES

```sql
CREATE OR REPLACE TABLE ROOM_TYPES (
    ID              NUMBER AUTOINCREMENT PRIMARY KEY,
    TYPE_NAME       VARCHAR(50)   NOT NULL,
    DESCRIPTION     VARCHAR(200),
    PRICE_PER_NIGHT NUMERIC(10,2) NOT NULL
);
```

| Column | Type | Description |
|--------|------|-------------|
| ID | NUMBER | Auto-generated primary key |
| TYPE_NAME | VARCHAR(50) | Category name (Standard, Deluxe, Suite, Presidential) |
| DESCRIPTION | VARCHAR(200) | Short description of the room type |
| PRICE_PER_NIGHT | NUMERIC(10,2) | Nightly rate in USD |

**Insert script:**
```sql
INSERT INTO ROOM_TYPES (TYPE_NAME, DESCRIPTION, PRICE_PER_NIGHT) VALUES
    ('Standard',     'Cozy room with queen bed, TV and Wi-Fi',             80.00),
    ('Deluxe',       'Spacious room with king bed and city view',          150.00),
    ('Suite',        'Luxury suite with living area and kitchenette',      280.00),
    ('Presidential', 'Top floor presidential suite with private terrace',  500.00);
```

**Sample rows inserted:**
| ID | TYPE_NAME | PRICE_PER_NIGHT |
|----|-----------|----------------|
| 1 | Standard | $80.00 |
| 2 | Deluxe | $150.00 |
| 3 | Suite | $280.00 |
| 4 | Presidential | $500.00 |

---

### Step 4 — Table: ROOMS

```sql
CREATE OR REPLACE TABLE ROOMS (
    ID           NUMBER AUTOINCREMENT PRIMARY KEY,
    ROOM_TYPE_ID NUMBER NOT NULL REFERENCES ROOM_TYPES(ID),
    ROOM_NUMBER  VARCHAR(10) NOT NULL,
    FLOOR        NUMBER,
    STATUS       VARCHAR(20) DEFAULT 'available'
);
```

| Column | Type | Description |
|--------|------|-------------|
| ID | NUMBER | Auto-generated primary key |
| ROOM_TYPE_ID | NUMBER | FK → ROOM_TYPES.ID |
| ROOM_NUMBER | VARCHAR(10) | Room identifier e.g. 101, 202 |
| FLOOR | NUMBER | Floor number |
| STATUS | VARCHAR(20) | `available` / `occupied` / `maintenance` |

**Insert script:**
```sql
INSERT INTO ROOMS (ROOM_TYPE_ID, ROOM_NUMBER, FLOOR, STATUS) VALUES
    (1, '101', 1, 'available'),
    (1, '102', 1, 'occupied'),
    (1, '103', 1, 'available'),
    (2, '201', 2, 'available'),
    (2, '202', 2, 'occupied'),
    (2, '203', 2, 'maintenance'),
    (3, '301', 3, 'available'),
    (3, '302', 3, 'occupied'),
    (4, '401', 4, 'available'),
    (4, '402', 4, 'occupied');
```

**Sample rows inserted (10 rooms):**
| ROOM_NUMBER | FLOOR | TYPE | STATUS |
|-------------|-------|------|--------|
| 101 | 1 | Standard | available |
| 102 | 1 | Standard | occupied |
| 103 | 1 | Standard | available |
| 201 | 2 | Deluxe | available |
| 202 | 2 | Deluxe | occupied |
| 203 | 2 | Deluxe | maintenance |
| 301 | 3 | Suite | available |
| 302 | 3 | Suite | occupied |
| 401 | 4 | Presidential | available |
| 402 | 4 | Presidential | occupied |

---

### Step 5 — Table: RESERVATIONS

```sql
CREATE OR REPLACE TABLE RESERVATIONS (
    ID          NUMBER AUTOINCREMENT PRIMARY KEY,
    REF         VARCHAR(20) UNIQUE NOT NULL,
    GUEST_ID    NUMBER NOT NULL REFERENCES GUESTS(ID),
    ROOM_ID     NUMBER NOT NULL REFERENCES ROOMS(ID),
    CHECK_IN    DATE NOT NULL,
    CHECK_OUT   DATE NOT NULL,
    TOTAL_PRICE NUMERIC(10,2) NOT NULL,
    STATUS      VARCHAR(20) DEFAULT 'confirmed'
);
```

| Column | Type | Description |
|--------|------|-------------|
| ID | NUMBER | Auto-generated primary key |
| REF | VARCHAR(20) | Unique booking reference e.g. HB-10001 |
| GUEST_ID | NUMBER | FK → GUESTS.ID |
| ROOM_ID | NUMBER | FK → ROOMS.ID |
| CHECK_IN | DATE | Arrival date |
| CHECK_OUT | DATE | Departure date |
| TOTAL_PRICE | NUMERIC(10,2) | Total booking amount |
| STATUS | VARCHAR(20) | `confirmed` / `checked_in` / `checked_out` / `cancelled` |

**Insert script:**
```sql
INSERT INTO RESERVATIONS (REF, GUEST_ID, ROOM_ID, CHECK_IN, CHECK_OUT, TOTAL_PRICE, STATUS) VALUES
    ('HB-10001', 1, 2,  '2026-04-10', '2026-04-13', 240.00,  'checked_in'),
    ('HB-10002', 2, 5,  '2026-04-11', '2026-04-14', 450.00,  'confirmed'),
    ('HB-10003', 3, 8,  '2026-04-12', '2026-04-15', 840.00,  'confirmed'),
    ('HB-10004', 4, 10, '2026-04-13', '2026-04-16', 1500.00, 'checked_in'),
    ('HB-10005', 5, 1,  '2026-04-20', '2026-04-22', 160.00,  'confirmed');
```

**Sample rows inserted:**
| REF | GUEST | ROOM | CHECK_IN | CHECK_OUT | TOTAL | STATUS |
|-----|-------|------|----------|-----------|-------|--------|
| HB-10001 | James Carter | 102 | 2026-04-10 | 2026-04-13 | $240.00 | checked_in |
| HB-10002 | Sofia Reyes | 202 | 2026-04-11 | 2026-04-14 | $450.00 | confirmed |
| HB-10003 | Ahmed Al-Farsi | 302 | 2026-04-12 | 2026-04-15 | $840.00 | confirmed |
| HB-10004 | Yuki Tanaka | 402 | 2026-04-13 | 2026-04-16 | $1,500.00 | checked_in |
| HB-10005 | Emma Dubois | 101 | 2026-04-20 | 2026-04-22 | $160.00 | confirmed |

---

### Table Relationships

```
ROOM_TYPES  ──┐
              ├──► ROOMS ──────────────────┐
                                           ├──► RESERVATIONS
GUESTS ────────────────────────────────────┘
```

---

### Verification Query (runs at end of file)

```sql
SELECT 'ROOM_TYPES'   AS tbl, COUNT(*) AS rows FROM ROOM_TYPES   UNION ALL
SELECT 'ROOMS',              COUNT(*)          FROM ROOMS         UNION ALL
SELECT 'GUESTS',             COUNT(*)          FROM GUESTS        UNION ALL
SELECT 'RESERVATIONS',       COUNT(*)          FROM RESERVATIONS;
```

**Expected output:**
| TBL | ROWS |
|-----|------|
| ROOM_TYPES | 4 |
| ROOMS | 10 |
| GUESTS | 5 |
| RESERVATIONS | 5 |

---

---

## 📄 File 2 — `02_hotel_mcp_server.sql`

### What it does
Creates 5 stored procedures as MCP tools, grants privileges, and registers the MCP server object in Snowflake.

---

### Step 1 — Stored Procedure: GET_ALL_RESERVATIONS

```sql
CREATE OR REPLACE PROCEDURE GET_ALL_RESERVATIONS()
RETURNS VARIANT
LANGUAGE SQL
```

| Detail | Value |
|--------|-------|
| Parameters | None |
| Returns | VARIANT (JSON array of reservations) |
| Joins | RESERVATIONS + GUESTS + ROOMS + ROOM_TYPES |
| Order | CHECK_IN DESC |

**Output fields:** ref, guest, email, phone, room, room_type, price_per_night, check_in, check_out, nights, total_price, status

---

### Step 2 — Stored Procedure: GET_AVAILABLE_ROOMS

```sql
CREATE OR REPLACE PROCEDURE GET_AVAILABLE_ROOMS(ROOM_TYPE VARCHAR)
RETURNS VARIANT
LANGUAGE SQL
```

| Detail | Value |
|--------|-------|
| Parameters | `ROOM_TYPE VARCHAR` — optional filter |
| Returns | VARIANT (JSON array of available rooms) |
| Filter | `STATUS = 'available'` + optional type filter |

**Usage examples:**
```sql
CALL GET_AVAILABLE_ROOMS('');          -- all available rooms
CALL GET_AVAILABLE_ROOMS('Suite');     -- only available Suites
CALL GET_AVAILABLE_ROOMS('Deluxe');    -- only available Deluxe rooms
```

---

### Step 3 — Stored Procedure: UPDATE_RESERVATION_STATUS

```sql
CREATE OR REPLACE PROCEDURE UPDATE_RESERVATION_STATUS(REF_NO VARCHAR, NEW_STATUS VARCHAR)
RETURNS VARCHAR
LANGUAGE SQL
```

| Detail | Value |
|--------|-------|
| Parameters | `REF_NO VARCHAR`, `NEW_STATUS VARCHAR` |
| Returns | VARCHAR confirmation message |
| Valid statuses | `confirmed`, `checked_in`, `checked_out`, `cancelled` |

**Usage examples:**
```sql
CALL UPDATE_RESERVATION_STATUS('HB-10002', 'checked_in');
CALL UPDATE_RESERVATION_STATUS('HB-10003', 'checked_out');
CALL UPDATE_RESERVATION_STATUS('HB-10005', 'cancelled');
```

---

### Step 4 — Stored Procedure: CANCEL_RESERVATION

```sql
CREATE OR REPLACE PROCEDURE CANCEL_RESERVATION(REF_NO VARCHAR)
RETURNS VARCHAR
LANGUAGE SQL
```

| Detail | Value |
|--------|-------|
| Parameters | `REF_NO VARCHAR` |
| Returns | VARCHAR confirmation message |
| Side effects | Sets reservation STATUS = `cancelled` AND room STATUS = `available` |

**What it does internally:**
1. Looks up room number from reservation
2. Updates reservation status to `cancelled`
3. Updates room status to `available`

**Usage example:**
```sql
CALL CANCEL_RESERVATION('HB-10003');
-- Returns: "Reservation HB-10003 cancelled. Room 302 is now available."
```

---

### Step 5 — Stored Procedure: GET_HOTEL_STATS

```sql
CREATE OR REPLACE PROCEDURE GET_HOTEL_STATS()
RETURNS VARIANT
LANGUAGE SQL
```

| Detail | Value |
|--------|-------|
| Parameters | None |
| Returns | VARIANT (JSON summary object) |

**Output fields:**
| Field | Description |
|-------|-------------|
| total_rooms | Total number of rooms |
| available_rooms | Rooms with status = available |
| occupied_rooms | Rooms with status = occupied |
| maintenance | Rooms with status = maintenance |
| occupancy_rate_pct | occupied / total × 100 |
| total_reservations | All reservations count |
| confirmed | Reservations by status |
| checked_in | Reservations by status |
| checked_out | Reservations by status |
| cancelled | Reservations by status |
| total_revenue | Sum of non-cancelled reservation prices |

---

### Step 6 — Privilege Grants

```sql
GRANT CREATE MCP SERVER ON SCHEMA HOTEL_DB.HOTEL TO ROLE SYSADMIN;
GRANT USAGE ON SCHEMA HOTEL_DB.HOTEL TO ROLE SYSADMIN;
GRANT USAGE ON PROCEDURE HOTEL_DB.HOTEL.GET_ALL_RESERVATIONS() TO ROLE SYSADMIN;
-- ... (one GRANT per procedure)
```

> ⚠️ Change `SYSADMIN` to your actual role if different.

---

### Step 7 — CREATE MCP SERVER

```sql
CREATE OR REPLACE MCP SERVER HOTEL_DB.HOTEL.HOTELBOOK_MCP
FROM SPECIFICATION $$
tools:
  - ...
$$;
```

**Tools registered in MCP server:**

| Tool Name | Type | Backed By |
|-----------|------|-----------|
| `get_all_reservations` | GENERIC | SP: GET_ALL_RESERVATIONS() |
| `get_available_rooms` | GENERIC | SP: GET_AVAILABLE_ROOMS(VARCHAR) |
| `update_reservation_status` | GENERIC | SP: UPDATE_RESERVATION_STATUS(VARCHAR, VARCHAR) |
| `cancel_reservation` | GENERIC | SP: CANCEL_RESERVATION(VARCHAR) |
| `get_hotel_stats` | GENERIC | SP: GET_HOTEL_STATS() |
| `sql_query` | SYSTEM_EXECUTE_SQL | Direct SQL execution |

---

### Step 8 — Verify

```sql
DESCRIBE MCP SERVER HOTEL_DB.HOTEL.HOTELBOOK_MCP;
SHOW MCP SERVERS IN SCHEMA HOTEL_DB.HOTEL;
```

**Expected output from DESCRIBE:**
- Server name: `HOTELBOOK_MCP`
- Schema: `HOTEL_DB.HOTEL`
- URL: `https://<account>.snowflakecomputing.com/api/v2/mcp/servers/...`
- Tools: 6 tools listed

---

## 🔄 Full Execution Order

```
1. Open Snowflake Worksheet
2. Run: 01_hotel_schema_data.sql   → creates tables + data
3. Verify row counts match expected
4. Run: 02_hotel_mcp_server.sql    → creates procedures + MCP server
5. Copy MCP server URL from DESCRIBE output
6. Configure Claude Desktop with the URL
```
