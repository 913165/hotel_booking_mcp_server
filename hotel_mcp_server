-- ═══════════════════════════════════════════════════════════════
-- STEP 2: SEMANTIC VIEWS + MCP SERVER
-- Run this AFTER 01_hotel_schema_data.sql
-- ═══════════════════════════════════════════════════════════════

USE DATABASE HOTEL_DB;
USE SCHEMA HOTEL;

-- ── Semantic View 1: RESERVATIONS_SV ─────────────────────────
-- Main semantic view joining all 4 tables
-- Used by Cortex Analyst for reservation queries
CREATE OR REPLACE SEMANTIC VIEW HOTEL_DB.HOTEL.RESERVATIONS_SV
  TABLES (
    r  AS HOTEL_DB.HOTEL.RESERVATIONS    PRIMARY KEY (ID),
    g  AS HOTEL_DB.HOTEL.GUESTS          PRIMARY KEY (ID),
    rm AS HOTEL_DB.HOTEL.ROOMS           PRIMARY KEY (ID),
    rt AS HOTEL_DB.HOTEL.ROOM_TYPES      PRIMARY KEY (ID)
  )
  RELATIONSHIPS (
    r (GUEST_ID)   REFERENCES g  (ID),
    r (ROOM_ID)    REFERENCES rm (ID),
    rm (ROOM_TYPE_ID) REFERENCES rt (ID)
  )
  FACTS (
    TOTAL_PRICE       AS r.TOTAL_PRICE         COMMENT 'Total price of the reservation',
    NIGHTS            AS DATEDIFF('day', r.CHECK_IN, r.CHECK_OUT)
                                               COMMENT 'Number of nights stayed',
    PRICE_PER_NIGHT   AS rt.PRICE_PER_NIGHT    COMMENT 'Nightly rate for the room type'
  )
  DIMENSIONS (
    -- Reservation dimensions
    REF               AS r.REF                 COMMENT 'Unique reservation reference e.g. HB-10001',
    CHECK_IN          AS r.CHECK_IN            COMMENT 'Guest check-in date',
    CHECK_OUT         AS r.CHECK_OUT           COMMENT 'Guest check-out date',
    RESERVATION_STATUS AS r.STATUS             COMMENT 'Status: confirmed, checked_in, checked_out, cancelled',

    -- Guest dimensions
    GUEST_NAME        AS g.NAME                COMMENT 'Full name of the guest',
    GUEST_EMAIL       AS g.EMAIL               COMMENT 'Guest email address',
    GUEST_PHONE       AS g.PHONE               COMMENT 'Guest phone number',
    NATIONALITY       AS g.NATIONALITY         COMMENT 'Guest nationality',

    -- Room dimensions
    ROOM_NUMBER       AS rm.ROOM_NUMBER        COMMENT 'Room number e.g. 101, 202',
    FLOOR             AS rm.FLOOR              COMMENT 'Floor the room is on',
    ROOM_STATUS       AS rm.STATUS             COMMENT 'Room status: available, occupied, maintenance',

    -- Room type dimensions
    ROOM_TYPE         AS rt.TYPE_NAME          COMMENT 'Room category: Standard, Deluxe, Suite, Presidential',
    ROOM_DESCRIPTION  AS rt.DESCRIPTION        COMMENT 'Description of the room type'
  )
  METRICS (
    TOTAL_RESERVATIONS AS COUNT(r.ID)
                       COMMENT 'Total number of reservations',

    TOTAL_REVENUE      AS SUM(CASE WHEN r.STATUS != 'cancelled' THEN r.TOTAL_PRICE ELSE 0 END)
                       COMMENT 'Total revenue excluding cancelled reservations',

    CONFIRMED_COUNT    AS COUNT(CASE WHEN r.STATUS = 'confirmed'   THEN 1 END)
                       COMMENT 'Number of confirmed reservations',

    CHECKED_IN_COUNT   AS COUNT(CASE WHEN r.STATUS = 'checked_in'  THEN 1 END)
                       COMMENT 'Number of guests currently checked in',

    CHECKED_OUT_COUNT  AS COUNT(CASE WHEN r.STATUS = 'checked_out' THEN 1 END)
                       COMMENT 'Number of guests checked out',

    CANCELLED_COUNT    AS COUNT(CASE WHEN r.STATUS = 'cancelled'   THEN 1 END)
                       COMMENT 'Number of cancelled reservations',

    AVG_STAY_NIGHTS    AS AVG(DATEDIFF('day', r.CHECK_IN, r.CHECK_OUT))
                       COMMENT 'Average length of stay in nights',

    AVG_REVENUE_PER_BOOKING AS AVG(CASE WHEN r.STATUS != 'cancelled' THEN r.TOTAL_PRICE END)
                       COMMENT 'Average revenue per non-cancelled booking'
  );


-- ── Semantic View 2: ROOMS_SV ─────────────────────────────────
-- Room availability and occupancy analytics
CREATE OR REPLACE SEMANTIC VIEW HOTEL_DB.HOTEL.ROOMS_SV
  TABLES (
    rm AS HOTEL_DB.HOTEL.ROOMS           PRIMARY KEY (ID),
    rt AS HOTEL_DB.HOTEL.ROOM_TYPES      PRIMARY KEY (ID)
  )
  RELATIONSHIPS (
    rm (ROOM_TYPE_ID) REFERENCES rt (ID)
  )
  FACTS (
    PRICE_PER_NIGHT AS rt.PRICE_PER_NIGHT  COMMENT 'Nightly rate for this room type'
  )
  DIMENSIONS (
    ROOM_NUMBER     AS rm.ROOM_NUMBER       COMMENT 'Room number e.g. 101, 202',
    FLOOR           AS rm.FLOOR             COMMENT 'Floor level',
    ROOM_STATUS     AS rm.STATUS            COMMENT 'Current status: available, occupied, maintenance',
    ROOM_TYPE       AS rt.TYPE_NAME         COMMENT 'Room category: Standard, Deluxe, Suite, Presidential',
    DESCRIPTION     AS rt.DESCRIPTION       COMMENT 'Room type description'
  )
  METRICS (
    TOTAL_ROOMS       AS COUNT(rm.ID)
                      COMMENT 'Total number of rooms',

    AVAILABLE_ROOMS   AS COUNT(CASE WHEN rm.STATUS = 'available'   THEN 1 END)
                      COMMENT 'Rooms currently available for booking',

    OCCUPIED_ROOMS    AS COUNT(CASE WHEN rm.STATUS = 'occupied'    THEN 1 END)
                      COMMENT 'Rooms currently occupied',

    MAINTENANCE_ROOMS AS COUNT(CASE WHEN rm.STATUS = 'maintenance' THEN 1 END)
                      COMMENT 'Rooms under maintenance',

    OCCUPANCY_RATE    AS ROUND(
                           COUNT(CASE WHEN rm.STATUS = 'occupied' THEN 1 END)
                           / NULLIF(COUNT(rm.ID), 0) * 100, 1)
                      COMMENT 'Occupancy rate as a percentage'
  );


-- ── Grants ────────────────────────────────────────────────────
-- Adjust role name as needed
GRANT CREATE MCP SERVER ON SCHEMA HOTEL_DB.HOTEL TO ROLE SYSADMIN;
GRANT USAGE  ON SCHEMA HOTEL_DB.HOTEL            TO ROLE SYSADMIN;
GRANT SELECT ON SEMANTIC VIEW HOTEL_DB.HOTEL.RESERVATIONS_SV TO ROLE SYSADMIN;
GRANT SELECT ON SEMANTIC VIEW HOTEL_DB.HOTEL.ROOMS_SV        TO ROLE SYSADMIN;


-- ── CREATE MCP SERVER ─────────────────────────────────────────
CREATE OR REPLACE MCP SERVER HOTEL_DB.HOTEL.HOTELBOOK_MCP
FROM SPECIFICATION $$
tools:
  - title: "Hotel Reservations Analyst"
    name: "reservations_analyst"
    type: "CORTEX_ANALYST_MESSAGE"
    identifier: "HOTEL_DB.HOTEL.RESERVATIONS_SV"
    description: >
      Answer natural language questions about hotel reservations.
      Covers guest details, room info, check-in/out dates, revenue,
      booking status (confirmed, checked_in, checked_out, cancelled),
      average stay duration, and revenue breakdowns by room type or nationality.

  - title: "Hotel Rooms Analyst"
    name: "rooms_analyst"
    type: "CORTEX_ANALYST_MESSAGE"
    identifier: "HOTEL_DB.HOTEL.ROOMS_SV"
    description: >
      Answer natural language questions about hotel rooms and occupancy.
      Covers room availability, occupancy rates, rooms by floor or type,
      pricing per room category, and maintenance status.

  - title: "SQL Query Tool"
    name: "sql_query"
    type: "SYSTEM_EXECUTE_SQL"
    description: >
      Execute any direct SQL query against HOTEL_DB.HOTEL tables
      for custom ad-hoc reports or data lookups not covered by the analyst tools.
$$;


-- ── Verify ────────────────────────────────────────────────────
SHOW SEMANTIC VIEWS IN SCHEMA HOTEL_DB.HOTEL;
DESCRIBE MCP SERVER HOTEL_DB.HOTEL.HOTELBOOK_MCP;
SHOW MCP SERVERS IN SCHEMA HOTEL_DB.HOTEL;
