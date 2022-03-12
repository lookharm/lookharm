# Database Partitioning (PostgreSQL)

### Overview <a href="#_71zeo6hm9ygv" id="_71zeo6hm9ygv"></a>

* The partition can increase the speed of querying dramatically.

### Declarative <a href="#_d5vef16o3j7x" id="_d5vef16o3j7x"></a>

#### Create Partitioned Table <a href="#_nowbd3pes71b" id="_nowbd3pes71b"></a>

* PostgreSQL has two forms of partitions: RANGE and LIST

```sql
CREATE TABLE measurement (
    city_id int not null,
    logdate date not null,
    peaktemp int,
    unitsales int
) PARTITION BY RANGE (logdate);
```

#### Create partitions <a href="#_s8lkc9533ts9" id="_s8lkc9533ts9"></a>

```sql
CREATE TABLE measurement_y2006m02 PARTITION OF measurement
FOR VALUES FROM ('2006-02-01') TO ('2006-03-01')
CREATE TABLE measurement_y2006m03 PARTITION OF measurement
FOR VALUES FROM ('2006-03-01') TO ('2006-04-01')
CREATE TABLE measurement_y2006m04 PARTITION OF measurement
FOR VALUES FROM ('2006-04-01') TO ('2006-05-01')
CREATE TABLE measurement_y2006m05 PARTITION OF measurement
FOR VALUES FROM ('2006-05-01') TO ('2006-06-01')
```

#### Insert into partitioned table <a href="#_6y001blwcrlu" id="_6y001blwcrlu"></a>

* Data will be routed to its partition range

```sql
INSERT INTO measurement(city_id, logdate, peaktemp, unitsales)
VALUES (1, '2006-02-01', 100, 1)
INSERT INTO measurement(city_id, logdate, peaktemp, unitsales)
VALUES (2, '2006-03-01', 100, 1)
INSERT INTO measurement(city_id, logdate, peaktemp, unitsales)
VALUES (3, '2006-04-30', 100, 1)
INSERT INTO measurement(city_id, logdate, peaktemp, unitsales)
VALUES (4, '2006-05-15', 100, 1)
```

#### Drop partition <a href="#_pk4ruqzcrthu" id="_pk4ruqzcrthu"></a>

* Data will also be deleted from the parent table.

```sql
DROP TABLE measurement_y2006m02;
```

#### Detach partition <a href="#_v2y990ed1osa" id="_v2y990ed1osa"></a>

* Data still exists in a partition but no longer exists in a parent.

```sql
ALTER TABLE measurement DETACH PARTITION measurement_y2006m03;
```

#### Attach partition <a href="#_qttfhww6509p" id="_qttfhww6509p"></a>

* Attach the existing partition to a parent table.

```sql
ALTER TABLE measurement ATTACH PARTITION measurement_y2006m03
FOR VALUES FROM ('2006-03-01') TO ('2006-04-01')
```

### Inheritance <a href="#_l1r5ittxatp" id="_l1r5ittxatp"></a>

* Advantage
  * Partition tables can have columns more than a parent.
  * Not use EXCLUSIVE LOCK like declarative just use.
  * Flexible: can be used with a table that didn’t partition at a created time.
* Disadvantage
  * Must implement partition routing by self such as CHECK constraint for each partition, FUNCTION for the check which partition that should be inserted data.
  * Can no use with partitioned table, the table that is partitioned by declarative.

CREATE TABLE measurement\_y2006m02 () INHERITS (measurement);

### Further reading <a href="#_9morxzpniep4" id="_9morxzpniep4"></a>

* [https://www.postgresql.org/docs/10/ddl-partitioning.html](https://www.postgresql.org/docs/10/ddl-partitioning.html)
