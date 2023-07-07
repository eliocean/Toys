# 文章： MYSQL VS MARIADB: TABLES AND DDL 区别

Although MySQL and MariaDB share a common ancestry, their functionality has diverged in subtle ways over the years. In this post, we’ll explore the differences in DDL and schema-related features of these two database servers, as well as operational concerns when performing schema changes.

If you’re currently planning a migration between these databases, you may find this list of differences to be surprisingly long and scary! But don’t worry, at the end of the post we’ll demonstrate an easy single-command solution for instantly testing cross-MySQL/MariaDB schema compatibility using Skeema and Docker.

## Table functionality

In this section, we’ll review the major functionality differences in table definitions. Please note we’re mostly just considering InnoDB tables in this post. Users of other storage engines will face much greater compatibility differences, and likewise for heavy users of stored procedures and functions.

#### JSON column type

MySQL provides a native [`JSON` column type](https://dev.mysql.com/doc/refman/8.0/en/json.html) which includes validation and binary storage. Internal fields in a JSON value can be indexed using virtual columns or functional indexes, and JSON arrays may be indexed using [multi-valued indexes](https://dev.mysql.com/doc/refman/8.0/en/create-index.html#create-index-multi-valued) (including support for *partial indexes*, i.e. empty arrays are omitted from the index). MySQL provides numerous [JSON functions](https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html), including shortcut arrow extraction operators.

In MariaDB, the type is actually an [alias for `LONGTEXT`](https://mariadb.com/kb/en/json-data-type/) with an automatic CHECK constraint calling to ensure validity. MariaDB provides a similar but slightly different list of [JSON functions](https://mariadb.com/kb/en/json-functions/), and lacks the shortcut arrow operators. MariaDB does not support multi-valued indexes yet.`JSON``JSON_VALID()`

#### inet4, inet6, uuid column types

MariaDB provides convenience types for storing [IPv4](https://mariadb.com/kb/en/inet4/), [IPv6](https://mariadb.com/kb/en/inet6/), and [UUID](https://mariadb.com/kb/en/uuid-data-type/) values in an efficient binary format, while still presenting human-readable textual versions to clients.

MySQL does not offer these types. The closest equivalent would be storing the values as appropriately-sized BINARY columns, and then adding an additional VIRTUAL column for the human-readable conversion, or perhaps using a view.

#### Numeric column types

Integer types in MySQL now omit/ignore **display widths**. For example, you’ll just see instead of . MariaDB still retains them. Display widths typically have [no effect whatsoever](https://www.skeema.io/docs/options/#lint-display-width) and are a common cause of developer confusion, so we applaud their removal.`int``int(11)`

MariaDB permits columns to store up to [38 digits after the decimal place](https://mariadb.com/kb/en/decimal/). In MySQL, the limit is 30 digits.`decimal`

#### Temporal column types

The type has various [deficiencies](https://www.skeema.io/docs/options/#lint-has-time) which are identical in both MySQL and MariaDB. The only major compatibility item to pay attention to is the **explicit_defaults_for_timestamp** server variable, which prevents some nonstandard legacy behaviors of the first column per table. This variable defaults to ON starting in [MySQL 8](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_explicit_defaults_for_timestamp) (released five years ago), but MariaDB only changed the default to ON in [MariaDB 10.10+](https://mariadb.com/kb/en/server-system-variables/#explicit_defaults_for_timestamp), released six months ago. If you migrate from MariaDB 10.9 (or older) to MySQL 8, or vice versa, be sure to carefully consider the effect of this variable.`timestamp``timestamp`

#### Collations

The list of available collations differs between MySQL and MariaDB. The default collation for each character set can differ as well, with one major example being the **default collation for utf8mb4**: in MySQL, it’s utf8mb4_0900_ai_ci ([UCA 9.0.0](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-sets.html), accent-insensitive, case-insensitive, no pad), whereas MariaDB currently still uses the [older, problematic](https://jira.mariadb.org/browse/MDEV-27009) utf8mb4_general_ci (non-UCA-compliant, accent-insensitive, case-insensitive, pad space) which considers all supplementary characters to be equal in comparisons.

#### Compression

Both MySQL and MariaDB support traditional InnoDB compression ([`ROW_FORMAT=COMPRESSED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-usage.html)) with no implementation differences. MariaDB briefly planned to deprecate this functionality, but reversed course after community feedback.

Both systems also support transparent page compression (“hole-punch”), albeit with different [syntax](https://dev.mysql.com/doc/refman/8.0/en/innodb-page-compression.html) and [compression algorithm configurability](https://mariadb.com/kb/en/innodb-page-compression/). However, top experts have been [highly critical](https://smalldatum.blogspot.com/2021/10/the-other-way-to-compress-innodb.html) of the production safety of this feature in general.

MariaDB also supports **[column-level compression](https://mariadb.com/kb/en/storage-engine-independent-column-compression/)**. Standard MySQL does not support this feature at all, but Percona Server provides a [more powerful implementation](https://docs.percona.com/percona-server/8.0/flexibility/compressed_columns.html) which optionally uses a pre-defined dictionary to achieve better compression ratios.

Although we’re largely just considering InnoDB for this post, we should mention that the MyRocks storage engine achieves better compression than any available solution in InnoDB. In theory, it can be used with any modern release of MySQL, MariaDB, or Percona Server. That said, you’ll have a much easier time using it with MariaDB or Percona Server. MyRocks is challenging to install in standard MySQL, and Oracle does not provide any technical support assistance for it.

Meanwhile, AWS Aurora users should note that [Aurora does not support compression](https://www.skeema.io/blog/2022/01/27/exploring-aurora-v3/) in any form, even in the most recent versions.

#### DEFAULT values

Both systems now support use of arbitrary expressions for column default values, rather than just requiring literal constants. However, the list of built-in functions [differs a bit](https://mariadb.com/kb/en/function-differences-between-mariadb-10-6-and-mysql-8-0/) between MySQL and MariaDB, potentially affecting portability of default value expressions.

Oddly, some column types in MySQL cannot have *literal* default values, but they *do* permit *expression* default values. This means you can just wrap a literal default value in parentheses to “expressionize” it:

```
mysql> CREATE TABLE foo (
    ->     comments text DEFAULT 'hello world'
    -> );
ERROR 1101 (42000): BLOB, TEXT, GEOMETRY or JSON column 'comments' can't have a default value

mysql> CREATE TABLE foo (
    ->     comments text DEFAULT ('hello world')
    -> );
Query OK, 0 rows affected (0.02 sec)
```

Mercifully, MariaDB doesn’t require you to jump through this hoop.

#### Generated columns

MySQL allows generated columns to optionally have the NOT NULL attribute, and properly enforces this by preventing writes which would yield a NULL value for the generation expression. MariaDB doesn’t permit this syntax, but you can use a CHECK constraint to get the same effect.

MySQL supports **[functional indexes](https://dev.mysql.com/doc/refman/8.0/en/create-index.html#create-index-functional-key-parts)**, allowing you to index arbitrary expression values directly. MariaDB doesn’t allow this, but you can index a virtual column instead, which indirectly achieves the same result.

As mentioned above, the list of built-in functions [differs](https://mariadb.com/kb/en/function-differences-between-mariadb-10-6-and-mysql-8-0/) between the two systems, which can be problematic during migrations for generated column definitions.

#### CHECK constraints

MySQL and MariaDB both support CHECK constraints, with only some minor differences in functionality.

In MySQL, the namespace for CHECK constraints is schema-wide, meaning each one must have a unique name within the schema. In MariaDB, they’re namespaced per table, so different tables may re-use the same constraint name.

In , MySQL offers dedicated syntax for dropping a CHECK constraint. MariaDB overloads the existing syntax for this purpose, which is also used for dropping FOREIGN KEY constraints. This is ill-conceived, since it’s possible for a FOREIGN KEY constraint and CHECK constraint to have the same name, and there’s no MariaDB syntax to indicate which one you want to drop!`ALTER TABLE``DROP CHECK``DROP CONSTRAINT`

MariaDB provides a funny-sounding server variable, [check_constraint_checks](https://mariadb.com/kb/en/server-system-variables/#check_constraint_checks), which can disable enforcement of *all* CHECK constraints, either globally or for the current session. MySQL does not provide this variable, but it does permit *individual* CHECK constraints to be disabled () or re-enabled () via syntax in , which MariaDB lacks.`NOT ENFORCED``ENFORCED``ALTER CHECK``ALTER TABLE`

As mentioned previously, the list of built-in functions differs between MySQL and MariaDB, which can affect portability of CHECK constraint expressions.

#### Miscellaneous features

MariaDB supports [**system-versioned tables**, application time periods, and bitemporal tables](https://mariadb.com/kb/en/system-versioned-tables/). MySQL does not offer any equivalent functionality.

In addition to traditional auto_increment tables, MariaDB allows you to use separate **[sequence](https://mariadb.com/kb/en/sequence-overview/)** objects for more fine-grained control. MySQL doesn’t provide this feature.

Partitioned tables using the LIST or LIST COLUMNS partitioning methods can have a DEFAULT (catchall) partition in MariaDB, but not in MySQL.

#### Reserved words

MySQL and MariaDB have different lists of [reserved words](https://dev.mysql.com/doc/refman/8.0/en/keywords.html). If any of your identifier names (tables, columns, stored procedures, etc) conflict with a reserved word, you will need to backtick-quote them in all queries. When migrating *from* MariaDB, some particularly problematic MySQL-only reserved words include , , , , , , and . If migrating in the other direction, the list for MariaDB includes some names like , , and .`groups``stored``empty``last_value``lead``rank``system``offset``position``general`

Skeema’s [lint-reserved-word](https://www.skeema.io/docs/options/#lint-reserved-word) option can help identify these conflicts automatically, as our codebase maintains a [mapping](https://github.com/skeema/skeema/blob/80635d9/internal/tengo/keyword.go#L95-L400) of reserved words by database flavor/version.

#### Portability comment syntax

MySQL and MariaDB each support special version-gated comment syntax for portability. SQL code wrapped in comments of the form will actually be [processed normally by MySQL 8.0+](https://dev.mysql.com/doc/refman/8.0/en/comments.html), but are [ignored by MariaDB](https://mariadb.com/kb/en/comment-syntax/#executable-comment-syntax); similarly, comments of the form are executed by MariaDB but ignored by MySQL.`/*!80000 ... */``/*M! ... */`

This syntax can be especially helpful if you need to maintain schema definitions that are compatible with both systems, even for functionality with syntactic differences. As an example, consider this table definition, which uses an column type in MariaDB, but falls back to for portability with MySQL:`inet6``binary(16)`

```sql
CREATE TABLE visits (
	`ip_address` /*!80000 binary(16) */ /*M! inet6 */
);
mysql> SHOW CREATE TABLE visits\G
*************************** 1. row ***************************
       Table: visits
Create Table: CREATE TABLE `visits` (
  `ip_address` binary(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)
MariaDB> SHOW CREATE TABLE visits\G
*************************** 1. row ***************************
       Table: visits
Create Table: CREATE TABLE `visits` (
  `ip_address` inet6 DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
1 row in set (0.000 sec)
```

## Operational differences

Several differing pieces of functionality affect operational concerns – that is, the safe *execution* of schema changes.

#### ALTER TABLE

Both systems support altering tables instantaneously using for some common operations, such as adding or dropping a column. However, many other types of alterations don’t support instant DDL. In such cases, the older method of non-blocking “online DDL” is often available (), but this tends to be slow for large tables, which then causes massive **replication lag** due to the asynchronous nature of the binlog-based logical replication stream. Beginning with MariaDB 10.8, the [binlog_alter_two_phase](https://mariadb.com/kb/en/replication-and-binary-log-system-variables/#binlog_alter_two_phase) variable offers a clever solution, potentially making slow online alters operationally feasible even if replicas are present. However, to date there’s a complete lack of community blog posts about production experience with this feature.`ALTER TABLE ... ALGORITHM=INSTANT``ALTER TABLE ... ALGORITHM=INPLACE, LOCK=NONE`

Standard MySQL doesn’t provide a way to make a slow replication-friendly, but AWS Aurora (which is based on MySQL, not MariaDB) potentially offers a solution. Aurora clusters utilize *physical* replication at the storage layer, which doesn’t suffer from traditional logical replication lag. This can make slow online alters usable directly, as long as there are no traditional binlog replicas tailing your cluster.`ALTER TABLE`

MariaDB’s allows you to specify [`ALGORITHM=NOCOPY`](https://mariadb.com/kb/en/innodb-online-ddl-overview/#nocopy-algorithm), which is stricter than concerning prevention of several expensive edge cases which rebuild the clustered index. MySQL doesn’t provide this level of granularity.`ALTER TABLE``ALGORITHM=INPLACE`

MariaDB also provides a mechanism for interactively [monitoring the progress of `ALTER TABLE`](https://mariadb.com/kb/en/progress-reporting/), which is a nice ease-of-use improvement.

#### Indexes

MySQL 8.0.27+ can **[build secondary indexes using multiple threads](https://blogs.oracle.com/mysql/post/mysql80-innodb-parallel-threads-ddl)**, improving performance quite substantially if your system has the spare resources to dedicate to this.`ALTER TABLE`

Both systems provide a way to mark an index as unusable for read queries, which is a good safety step prior to dropping a potentially-unused index. The syntax for this feature differs slightly though: MySQL calls this an **[INVISIBLE](https://dev.mysql.com/doc/refman/8.0/en/invisible-indexes.html)** index, while MariaDB calls it an **[IGNORED](https://mariadb.com/kb/en/ignored-indexes/)** index.

#### DROP TABLE

MySQL 8.0.23+ has fixed system stall issues that would occur when the InnoDB buffer pool is very large. It does not appear that MariaDB has an equivalent fix.`DROP TABLE`

#### Foreign key constraints

When running some types of DDL on a table, MySQL 8 obtains additional metadata locks on any other tables which have a parent or child foreign key constraint relationship with the table. This change is conceptually “correct”, but it can have severe operational consequences on heavy users of foreign key constraints. If there are long-running queries running among *any* of these parent/child tables, the DDL’s attempt to obtain metadata locks will be blocked – and since DDL is higher priority than other metadata locks, this in turn will block any new incoming queries on these tables, even simple reads. Typically this manifests as a major query pile-up which lasts until the long-running s all complete, followed by the DDL completion.`SELECT``SELECT`

#### Triggers, procs, funcs

MariaDB allows you to **atomically modify an existing trigger** using [CREATE OR REPLACE](https://mariadb.com/kb/en/create-trigger/). MySQL doesn’t provide this syntax, which means if you need to adjust an existing trigger, you must carefully lock the table, drop the old trigger, re-create it with the new definition, and then unlock the table. Otherwise, there’s a split second where the trigger does not exist, and data might be written during that time.

Similarly, MariaDB provides CREATE OR REPLACE syntax for stored procedures and functions as well. MySQL doesn’t offer this – and unlike with triggers, there’s no good workaround, since routines cannot be locked.

## Schema metadata

Database servers expose metadata in various ways, such as information_schema tables and SHOW queries. Minor divergences between MySQL and MariaDB can impact development tools, monitoring systems, and especially schema management systems. Skeema contains extremely detailed logic to account for these differences when introspecting your schemas and applying schema changes, as well as safety mechanisms to ensure correctness.

Many obvious differences can be found in the contents of **information_schema** – some of these tables only exist in MySQL or only in MariaDB. Some others exist in both, albeit with slightly different column lists, such as information_schema.check_constraints. But there are also much more subtle metadata discrepancies, especially since MySQL 8 completely replaced the internal data dictionary implementation used in prior releases.

MySQL 8 also introduced **caching logic for table statistics** in information_schema, controlled by the [information_schema_stats_expiry](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_information_schema_stats_expiry) variable. To avoid seeing stale data, sessions must override this variable, setting it to 0 to disable the cache entirely. (Skeema does this automatically for its MySQL 8 connections, in order to see up-to-date values for auto_increment counters.)

Finally, excessive emoji users should take note of metadata issues with **4-byte characters** in table definitions, such as in column default value expressions, generated column expressions, or check contraint clauses. All versions of MySQL and MariaDB fail to represent 4-byte characters properly in information_schema. Unfortunately, MariaDB mangles these characters in SHOW CREATE TABLE as well, so there’s no way to correctly dump definitions of tables using these characters. However, SHOW CREATE TABLE will return them properly in MySQL, providing a workaround used by Skeema.

## Compatibility check one-liner

If you’re considering a migration from MariaDB to MySQL (or vice versa), identifying your schema incompatibilities is a great first step in planning and estimating the effort. Using Skeema’s built-in Docker integration, we have an easy single-command solution to test your schema’s compatibility between the two systems. This can be used either as a one-off action, or perhaps as part of a continuous integration (CI) test suite on every commit, to prevent future regressions. The latter is especially useful if your project needs to maintain long-term compatibility with both MySQL and MariaDB at the same time.

The only system prerequisites here are:

- [`skeema`](https://www.skeema.io/docs/install/) (the Skeema CLI binary)
- A [schema repository](https://www.skeema.io/docs/examples/) managed by Skeema
- Docker daemon running locally

Assuming those are in place, let’s say you normally use MariaDB 10.6, and you want to test your schema’s compatibility with the latest MySQL 8. Simply to your schema repo and then run this command:`cd`

```mysql
skeema lint --workspace=docker --docker-cleanup=destroy --flavor=mysql:8.0
```

This will do the following:

- Download the Docker image for MySQL 8, if not already present on your system
- Create a Docker container running a MySQL 8 database server
- Run your schema repo’s CREATE statements in the containerized DB ([workspace=docker](https://www.skeema.io/docs/options/#workspace)), reporting any errors along the way
- Introspect and [lint](https://www.skeema.io/docs/commands/lint) all statements which succeeded, based on your configured linter options or Skeema’s defaults – for example [lint-reserved-word](https://www.skeema.io/docs/options/#lint-reserved-word) will indicate whether any of your table/column/etc names conflict with MySQL 8’s reserved words
- Halt the container and then delete it ([docker-cleanup=destroy](https://www.skeema.io/docs/options/#docker-cleanup)) to avoid taking up system resources

If you see no errors, your schema is in great shape compatibility-wise, for all CREATE statements [supported by your edition of Skeema](https://www.skeema.io/docs/requirements/#unsupported-features----community-edition)!

**Need advice or hands-on assistance with a particularly tricky migration project? In addition to developing Skeema, we provide expert consulting services for MySQL and MariaDB. [Reach out](https://www.skeema.io/contact/) to learn more.**
