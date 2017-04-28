# Oracle - alternative to IN clause (sync)

## Goal

Provide an alternative to using an IN clause to specify the IDs of orders through means of a temporary table.

## Solution

The solution uses a temporary table, which is independent for each session and holds data only until the content is deleted on commit. The temporary table will be used to temporarily store ID values which can be used in the WHERE clause for retrieving data contained in the original table.

## Notes

Relevant [Oracle Documentation](https://docs.oracle.com/cd/E11882_01/server.112/e41084/statements_7002.htm#SQLRF01402).

## GLOBAL TEMPORARY

From the Oracle documentation:

Specify `GLOBAL TEMPORARY` to indicate that the table is temporary and that its **definition** is visible to all sessions with appropriate privileges. _The data in a temporary table is visible only to the session that inserts the data into the table._</p>
When you first create a temporary table, its table metadata is stored in the data dictionary, but no space is allocated for table data. Space is allocated for the table segment at the time of the first DML operation on the table. The temporary table definition persists in the same way as the definitions of regular tables, but the table segment and any data the table contains are either **session-specific** or **transaction-specific** data. You specify whether the table segment and data are session- or transaction-specific with the [ON COMMIT keywords](https://docs.oracle.com/cd/E11882_01/server.112/e41084/statements_7002.htm#i2189569).</p>
You can perform DDL operations (such as `ALTER TABLE`, `DROP TABLE`, `CREATE INDEX`) on a temporary table only when no session is bound to it. A session becomes bound to a temporary table by performing an `INSERT` operation on the table. A session becomes unbound to the temporary table by issuing a `TRUNCATE` statement or at session termination, or, for a transaction-specific temporary table, by issuing a `COMMIT` or `ROLLBACK` statement.

## Tutorial: Order

This tutorial will be based on selecting multiple Orders (`VAN_ORDER`) from our internal Vanessa Database. 

### Objective

Provide an alternative to using an IN clause to specify the IDs of orders.

### Method

Use a temporary table, insert IDs into it, and join the table with the `VAN_ORDER` table. Clean out the temporary table when done.

### Background

Using an Oracle database with the following table:

```
create GLOBAL TEMPORARY TABLE vanfact.restocking_order_to_sync
ON COMMIT DELETE ROWS
as ( select orderid as id from van_order where 1=2);

grant select, INSERT, UPDATE ON vanfact.restocking_order_to_sync TO devvanessa;
```

### Using this

```
-- clear out the temporary table
commit;

-- show that the table exists and is empty
select * from vanfact.restocking_to_sync;

-- insert two order ids into the table
insert into vanfact.restocking_to_sync (id) values (22205186);
insert into vanfact.restocking_to_sync (id) values (22205185);

-- show that there are two rows in the table
select * from vanfact.restocking_to_sync;

-- select the orders that have matching IDs in the temporary table (2 orders)
select o.orderid from van_order o, vanfact.restocking_to_sync r where o.orderid = r.id;
```

### Visual output

![Query output](http://i.imgur.com/vJCmmhk.png)

The above shows that we are now selecting Orders using the temporary table.