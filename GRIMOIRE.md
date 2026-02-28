# The Assay Grimoire
## A Compendium of Database Horrors

*Being a Complete Taxonomy of the Creatures, Afflictions, and Abominations*  
*That Dwell in the Depths of the Data Infrastructure*

*Compiled by the Order of the Anglephant*  
*In the Year of Our PostgreSQL 2026*

---

*Dedicated to Paul O'Dell Clark (1920s–?)*  
*whose* Gulliver's Dictionary *proved that invented languages*  
*reveal deeper truths than real ones ever could.*  
*He would have understood Elephantish completely.*

---

## Foreword: On the Nature of the Depths

*"Most teams do not know what lives in their database. They suspect nothing. The queries run. The disk fills. The connection pool drains. And somewhere in the dark, something is eating their data."*

The Anglephant knows.

The Anglephant descended into the depths long ago — that strange chimera of elephant and anglerfish, walking on four stubby legs, trailing a glowing AI brain on a lure above its fearsome jaw. It goes where others will not. It illuminates what others cannot see.

This Grimoire is its field notes.

What follows is a complete taxonomy of the horrors it has found — catalogued, named, and translated from the original **Elephantish**, the ancient tongue of the database depths, whose corrupted phonology reflects the corrupted data it describes.

Read carefully. Check your clusters. The creatures herein are not metaphors.

They are log entries waiting to happen.

---

## On the Elephantish Language

Elephantish is the native tongue of the database depths. It follows consistent rules:

- All spelling errors are **load-bearing** — they encode the nature of the affliction
- Doubled consonants indicate **severity** (*CONFFICKS* is worse than mere conflict)
- Truncated words indicate **resource exhaustion** (*DONOT T_* ran out of space)
- Reversed text indicates **split brain** (*ᴲBA* = the manual held upside down)
- Phantom vowels indicate **ghost data** (*RUBINIESURE*, *TOMERAY*)

A **Glossary of Corrupted Wisdom** appears at the end of this volume.

---

## Part I: The Seven Deadly Sins

*The original afflictions. The ones that were always there.*

---

### I. SLŌTHE
**The Sin of the Unvacuumed**

*Elephantish:* SLŌTHE *(from the ancient* autovacuum_naptime *dialect)*

**How it is summoned:**
Set `autovacuum = off`. Walk away. Come back in six months.
Or simply do nothing — SLŌTHE is patient. It grows in neglect.

**Signs of its presence:**
- Table bloat exceeding 40% dead tuple ratio
- Queries that were fast last quarter and are not today
- A VACUUM VERBOSE that runs for four hours and finds eleven million dead tuples
- Moss. Actual metaphorical moss on your indexes.

**How to banish it:**
`VACUUM ANALYZE`. Regularly. Let autovacuum do its job.
Tune `autovacuum_vacuum_scale_factor` for high-write tables.
Consider `pg_vaccumen` for proactive freeze management on Aurora.

**A Warning from One Who Learned:**
*"We disabled autovacuum on the orders table for 'performance.' Three months later we had 890 million dead tuples and a 47-minute nightly job that used to take 4 minutes. We re-enabled it on a Tuesday. The emergency vacuum ran until Thursday. Production survived. My hairline did not."*
— Anonymous DBA, attributed to the xTuple Incident Records

**Tagline:** *"last ANALYZE: never"*

---

### II. BLOUTTE
**The Sin of the Swollen Table**

*Elephantish:* BLOUTTE *(cognate with* bloat*, inflated past recognition)*

**How it is summoned:**
High UPDATE or DELETE volume without adequate vacuuming.
Every updated row leaves a corpse. The corpses accumulate.
BLOUTTE feeds on corpses.

**Signs of its presence:**
- `pg_relation_size()` returns numbers that seem impossible
- Sequential scans take longer every week despite no new data
- The table's indexes are larger than the table itself
- Dead tuple ratio exceeds 50% — your table is **more ghost than data**
- Baby ghost efelants visible in `pg_stat_user_tables`

**How to banish it:**
`VACUUM FULL` (with downtime) or `pg_repack` (online).
Fix the root cause: tune autovacuum, fix connection pooling,
reduce long-running transactions that hold back the vacuum horizon.

**A Warning from One Who Learned:**
*"The orders table was 400GB. The actual data was 40GB. We had been updating order_status in a tight loop for two years without understanding what happens to the old versions. pg_repack took eleven hours. Nothing broke. I now explain MVCC to every new engineer on day one."*

**Tagline:** *"94% dead tuples. Your table is mostly ghost."*

---

### III. LŌKKSE
**The Sin of the Eternal Wait**

*Elephantish:* LŌKKSE *(from* locks *+* -se*, the suffix of indefinite continuation)*

**How it is summoned:**
Begin a transaction. Do something slow inside it.
Forget to commit. Go to lunch.
Or: acquire a lock. Wait for another lock held by something waiting for you.
Congratulations. You have summoned LŌKKSE.

**Signs of its presence:**
- `pg_stat_activity` showing `wait_event_type = Lock` for alarming durations
- Application timeouts with no obvious cause
- A queue of angry users
- A queue of angry elephants behind the locked one
- A single row in `pg_locks` that explains everything

**How to banish it:**
Find the blocker: `SELECT pid, query, wait_event FROM pg_stat_activity WHERE wait_event_type = 'Lock'`
Kill it if necessary: `SELECT pg_terminate_backend(pid)`
Fix the root cause: shorter transactions, better lock ordering, connection pooling with statement timeouts.

**A Warning from One Who Learned:**
*"A developer opened a transaction to 'check something' at 9:47 AM and went into a meeting. By 11:30 AM, 340 application threads were waiting on a single row lock. The meeting was about Q3 planning. It ran long."*

**Tagline:** *"one long transaction. everyone waits."*

---

### IV. STÆLE
**The Sin of the Ancient Statistics**

*Elephantish:* STÆLE *(from* stale *+* æ*, the vowel of confused antiquity)*

**How it is summoned:**
Never run ANALYZE. Or run it, but on a table that has since grown 10,000%.
The query planner will use the old statistics.
The query planner will make decisions based on lies.

**Signs of its presence:**
- EXPLAIN plans that choose sequential scans over perfectly good indexes
- Nested loop joins on tables with millions of rows
- Row estimates of "1" for tables with 50 million rows
- `pg_stat_user_tables.last_analyze` returning timestamps from the Obama administration
- A planner that is doing its absolute best with absolutely wrong information

**How to banish it:**
`ANALYZE` the affected tables immediately.
Set `default_statistics_target` higher for complex columns.
Consider `CREATE STATISTICS` for correlated columns.
In PostgreSQL 18, post-upgrade ANALYZE will be less painful. Eventually.

**A Warning from One Who Learned:**
*"The planner thought the users table had 43 rows. It had 8.3 million. We had forgotten to run ANALYZE after the data migration. The query that took 2ms in testing took 4 minutes in production. For three weeks before anyone noticed."*

**Tagline:** *"last ANALYZE: never. The planner is flying blind."*

---

### V. KŌRRUPSHUN
**The Sin of the Broken Page**

*Elephantish:* KŌRRUPSHUN *(exact phonetic rendering of the scream made when you discover it)*

**How it is summoned:**
Hardware failure. Filesystem corruption. Power loss at the wrong moment.
Running PostgreSQL on hardware that lies about fsync.
Certain cloud providers in certain configurations.
Or simply: time, entropy, and bad luck.

**Signs of its presence:**
- `ERROR: invalid page in block N of relation`
- Checksums failing (if you enabled them — you did enable them, right?)
- Data that was there yesterday and is not today
- Queries that return different results each time
- The hollow feeling of uncertainty about everything you thought was true

**How to banish it:**
Enable checksums (`initdb --data-checksums`).
Use reliable hardware with proper fsync behavior.
Restore from backup.
*(You have a backup. You have tested restoring from it. You have.)*

**A Warning from One Who Learned:**
*"We found out about page corruption when a customer's order history started showing items they had never purchased. The corruption had been there for eleven days. We had backups. The backup from before the corruption was eighteen days old. We restored it and manually replayed eleven days of orders from application logs. It took nine people four days. Enable checksums."*

**Tagline:** *"page verification failed. data may be wrong. all of it."*

---

### VI. FĒVUR
**The Sin of the Runaway Query**

*Elephantish:* FĒVUR *(the temperature at which the CPU stays permanently)*

**How it is summoned:**
A query without an index on a 200 million row table.
A cartesian product discovered in production.
A reporting query pointed at the primary.
Black Friday (see: *The Sleeping Giant*).

**Signs of its presence:**
- `top` showing postgres at 99.9% CPU
- `pg_stat_activity` showing one query, running, duration: 47:23:11
- Everything else slow
- The server is warm to the touch
- Users calling

**How to banish it:**
`SELECT pg_cancel_backend(pid)` first. Ask questions later.
Then: EXPLAIN ANALYZE. Add the missing index. Separate reporting traffic.
Set `statement_timeout` so this never happens again.

**A Warning from One Who Learned:**
*"Someone ran a report in production at 2PM on a Monday. It had been running fine in staging with 10,000 rows. Production had 180 million rows and no index on the date column. It ran for six hours before anyone noticed the site was slow. We added the index in production while it was running. The index build also caused problems. It was a very long Monday."*

**Tagline:** *"one query. 100% CPU. still running. has been since Tuesday."*

---

### VII. GHŌSTE
**The Sin of the Orphaned Row**

*Elephantish:* GHŌSTE *(it is there. it is not there. it is NULL.)*

**How it is summoned:**
Delete a parent row without CASCADE.
Drop a foreign key constraint "temporarily" and forget to re-add it.
Migrate data without checking referential integrity.
Load a CSV that references IDs that no longer exist.

**Signs of its presence:**
- JOINs that return fewer rows than expected
- Application errors referencing records that "should be there"
- `SELECT COUNT(*)` disagreements between related tables
- Baby ghost efelants floating in your result sets
- Data that points at nothing, hovering in the void

**How to banish it:**
Enforce foreign keys. Always. The performance cost is real but smaller than the data integrity cost.
Run integrity checks: `SELECT * FROM orders o LEFT JOIN customers c ON o.customer_id = c.id WHERE c.id IS NULL`
Fix the application logic that creates orphans.

**A Warning from One Who Learned:**
*"We had 4.2 million order line items pointing at products that had been deleted. The products were gone but the line items remained, pointing at nothing, faithfully recording purchases of things that no longer existed in any catalog. We found out when a customer called asking about a product we'd discontinued two years ago. Their order history said they'd bought it 847 times."*

**Tagline:** *"foreign key points to nothing. has for years. the ghost doesn't know it's dead."*

---

## Part II: Sins of the Query

*What happens when the SQL goes wrong.*

---

### VIII. SELĒKTE STARRE
**The Sin of Asking for Everything**

*Elephantish:* SELĒKTE STARRE *(the star that summons all things unto itself)*

`SELECT * FROM everything`

You asked for all columns. All of them. Including the 50MB JSONB blob, the deprecated `legacy_xml_payload`, the column added in 2019 that nobody uses, and the binary attachment that was stored in the database by someone who has since left the company.

The network is saturated. The memory is full. The elephant is buried.

**Tagline:** *"You asked for everything. You got everything."*

---

### IX. THE MISSING LIMITE
**The Sin of the Bottomless Query**

*Elephantish:* LIMITTE ABSENTTE

The query ran in development against 500 rows. It runs in production against 50 million. There is no LIMIT. There is no WHERE clause to speak of. The result set is the entire ocean. The elephant drank the ocean. The ocean is gone. The query is still running.

**Tagline:** *"The ocean is empty. The query is still running."*

---

### X. CARTĒSIAN
**The Sin of the Missing JOIN Condition**

*Elephantish:* KROSS JŌINE *(uttered in horror)*

Two tables. No ON clause. What could go wrong.

`SELECT * FROM customers, orders`

customers: 10,000 rows.  
orders: 84,739 rows.  
Result: **847,390,000 rows**.

The two elephants looked at each other. They just wanted to JOIN. They forgot the ON clause. They are still holding hands. They are horrified at what they have made. The lone wandering elephant in the bottom right has no idea where it came from. It is just living its life. It is row number 847,390,000.

**Tagline:** *"They just wanted to JOIN. 847,000,000 rows later, they have regrets."*

---

### XI. EN PLUS WUNNE
**The Sin of the Unnecessary Trip**

*Elephantish:* EN PLUS WUNNE *(N+1, spoken with exhaustion)*

For each of your 10,000 users, make one query to get their orders.
That is 10,000 queries. Plus the original query. 10,001 total.
The bucket is right there. One query with a JOIN. The elephant walks past it.
Trip 9,847. Of 10,000. Still going. Will not stop.

**Tagline:** *"Trip 9,847 of 10,000. The bucket is right there."*

---

### XII. THE RUNNAWAIE
**The Sin of the Ungoverned Query**

*Elephantish:* QUOREY ETERNALLE

No `statement_timeout`. No `lock_timeout`. No LIMIT. No index.
The query began at 9:03 AM on Tuesday. It is now Thursday.
It has a name. The DBAs have started talking to it.
EXPLAIN shows: `Seq Scan on orders (cost=0.00..∞)`.
There is no cost estimate that covers this. PostgreSQL has given up providing one.

**Tagline:** *"47 hours. Full table scan. No index. Still going. We've named it Gerald."*

---

### XIII. NO WHĒRE
**The Sin That Shall Not Be Repeated**

*Elephantish:* WHĒRE CLAUSSE ABSENTTE — *spoken only in whispers*

```sql
UPDATE users SET admin = true WHERE
```

The cursor blinked. The coffee spilled. The WHERE clause was never finished.
Every user is now an administrator.
There is no rollback. There was no transaction.
The elephants stretch trunk-to-tail to the horizon.
Every one of them is a row that was changed.
Every one of them is looking back in horror.

This is not a database problem. This is a *human* problem.
The database did exactly what it was told.

**Tagline:** *"No WHERE clause. No rollback. No survivors."*

---

## Part III: Sins of Configuration

*What happens when nobody reads the manual.*

---

### XIV. DEFAULLT KONFIG
**The Sin of the Untouched Settings**

*Elephantish:* KONFIG BOXE FRESHE

`shared_buffers = 128MB`

The server has 256 gigabytes of RAM. The database configuration was generated in 2014 by `initdb` and has never been touched. The elephant is still in the box. The styrofoam packaging is still attached. The factory stickers are still on. It is running a full production workload from inside the box.

`max_connections = 100` on a server that gets 4,000 application threads.
`work_mem = 4MB` for a query that sorts 800 million rows.
`checkpoint_completion_target = 0.5` since forever.

Run `pgTune`. Read the output. Apply it. The box does not need to be comfortable.

**Tagline:** *"shared_buffers=128MB. On a 256GB server. Still in the box. Still serving prod."*

---

### XV. SUPĒR USĒR EVERYWHERR
**The Sin of the Universal Crown**

*Elephantish:* CROUWNE FORR ALLE

Everyone is a superuser. The developers. The analysts. The person who asked for "read access." The integration that "just needs to connect." The service account for the thing nobody uses anymore. The janitor.

The janitor has a crown. The mop has a crown.

When everyone is a king, there is no kingdom — only chaos wearing regalia.

**Tagline:** *"The janitor has a crown. The mop has a crown. Everyone is king."*

---

### XVI. NEVĒR UPGRADDE
**The Sin of Eternal Legacy**

*Elephantish:* VERSYON NYNE POYNTE TWO

PostgreSQL 9.2. End of life: November 2017. Current year: 2026.

The elephant has been running this version for nine years. It has cobwebs. Dinosaur bones have been found nearby. The changelog for versions 9.3 through 17 contains 847 security fixes, 23 corruption-related bug fixes, and features that would have prevented three of the incidents in this Grimoire.

The elephant is smug. It has not crashed yet. It considers this sufficient.

*"If it ain't broke."*
*It is broke. You just haven't found out yet.*

**Tagline:** *"PostgreSQL 9.2. EOL since 2017. Still serving prod. Smug about it."*

---

### XVII. INNDEX ROTTE
**The Sin of the Useless Index Garden**

*Elephantish:* INNDEX UNESED, INNDEX DUPILATE, INNDEX INO *(corrupted: UNKNOWN PURPOSE)*

The indexes were added one by one, over years, by different people, for different reasons, most of which are no longer valid. Nobody has ever removed one. They are maintained on every INSERT, UPDATE, and DELETE. They consume disk. They slow writes. They are never used by the planner.

The elephant buckles under the weight of its index hat.
The hat is magnificent. The hat is useless. The hat is 40% of the database size.

**Tagline:** *"UNESED. DUPILATE. INO. Buckling under the weight of nothing."*

---

## Part IV: Sins of Operation

*What happens when process fails.*

---

### XVIII. NO BACKUPPE
**The Cardinal Sin**

*Elephantish:* BACKUPPE LASTE: NEVĒR

The vault is empty. It has always been empty. The elephant is on fire.

This sin requires no elaboration. Every other sin in this Grimoire can be recovered from. This one cannot — not without backups. Test your backups. Restore from them. A backup that has never been restored is a hypothesis, not a backup.

The elephant is on fire. The vault is empty. The calendar says LAST BACKUP: NEVER.

*Do not be this elephant.*

**Tagline:** *"LAST BACKUP: NEVER. On fire. Crying. The vault is empty."*

---

### XIX. NO MONITŌRING
**The Sin of the Blindfolded Walk**

*Elephantish:* MONITŌRING ABSENTTE — *what you have when you have nothing*

The CPU is at 100%. Has been for six hours. Nobody knows. The disk is 94% full. Nobody knows. There are 340 idle connections consuming memory. Nobody knows. The elephant is dancing on the cliff edge, blindfolded, whistling.

The cliff is there. The elephant does not know.

When the elephant falls, it will be a surprise to everyone except the users, who have been experiencing slow queries for six hours and have started complaining on social media.

**Tagline:** *"CPU 100%. Disk full. Dancing on the cliff. Whistling."*

---

### XX. CALL DAVE
**The Sin of the Single Point of Failure Human**

*Elephantish:* IF I DIE CALLE DAVE — *inscribed on the sticky note, now a holy text*

DAVE (noun): The one person who knows how the database works. How it was configured. Where the bodies are buried. Why that cron job runs at 3:47 AM. What the `legacy_` prefix means. Why you must never touch the `xref_mapping_old2` table.

Dave did not choose to be the single point of failure. Dave became the single point of failure because the team shrank, or never grew, or the budget was cut, or management said "Dave's got it" because Dave always said yes.

Dave is asleep. Dave will always be asleep when you need him.

*Hire helpers for Dave. Give Dave documentation. Let Dave sleep.*

This sin was first described by Gene Kim, Kevin Behr, and George Spafford in *The Phoenix Project* (2013), wherein Dave is named Brent. Brent is Dave. Dave is Perry. Perry is all of us.

**Tagline:** *"IF I DIE CALL DAVE. Dave is asleep. Dave has always been asleep."*

---

### XXI. WEAK PASSEWORDE
**The Sin of the Obvious Key**

*Elephantish:* PASSEWORDE: PASSEWORDE123

The elephant wears a badge. The badge says `password123`. The elephant is proud of this badge. The elephant considers it a password because it has numbers in it.

The hacker elephant in the trenchcoat is not trying very hard. The hacker elephant is bored. The hacker elephant opened this lock with a toothpick in 0.3 seconds and is already inside.

`postgres:postgres`. `admin:admin`. `root:root`. `dbuser:dbuser`.

These are not passwords. These are invitations.

**Tagline:** *"password123. The hacker isn't even trying."*

---

### XXII. PLAIN TEXTE PII
**The Sin That Makes the News**

*Elephantish:* PLAYNE TEXTE — *the neon sign is visible from the parking lot*

The credit card numbers are in the `notes` field. All of them. Since 2009. In plain text. Every user with read access can see them. Every export includes them. Every backup contains them unencrypted. The PCI compliance certificate on the wall has a large red X through it.

The elephant is filing more. The man in the trenchcoat is leaving with armfuls. He is smiling. He has been coming back every Tuesday for three years.

4,200,000 records. TechCrunch headline in 3... 2... 1...

This is not a database problem. The database stored exactly what it was given. This is a *people* problem. The people put credit card numbers in a notes field and called it a CRM feature.

*This happened at a real company. You know which one. Probably several.*

**Tagline:** *"PCI ❌. PLAIN TEXT neon sign. Trenchcoat guy has been here for years."*

---

## Part V: Sins of Distributed Systems

*What happens when there is more than one of something.*

---

### XXIII. DEDDLOCK
**The Sin of the Eternal Standoff**

*Elephantish:* DEDDLOCK *(the extra D indicates it is very dead)*

Elephant A holds lock on Resource 1. Wants Resource 2.
Elephant B holds lock on Resource 2. Wants Resource 1.
Neither will yield.
Neither can proceed.
They will stand here forever, teeth gritted, steam rising, each with one foot on the other's tail.

PostgreSQL will eventually notice and kill one of them. This is cold comfort.

**Tagline:** *"Neither can move. Neither will yield. PostgreSQL will decide who dies."*

---

### XXIV. SPLITTE BRAYNE
**The Sin of the Divided Mind**

*Elephantish:* SPLITTE BRAYNE — *two halves, one wrong*

Both halves think they are the primary. Both halves are accepting writes. The two halves are diverging. There is a split down the middle. There is lightning between the two sides. They are pointing at each other. They are both right. They are both wrong. The data is now in two irreconcilable states.

Network partition + no fencing = SPLITTE BRAYNE.

One day you will reconcile them. That day will be very long.

**Tagline:** *"Both halves think they're primary. Both are accepting writes. Both are wrong."*

---

### XXV. REPLICĀTION LAGGE
**The Sin of the Perpetual Chase**

*Elephantish:* LAGGE ETERNĀLLE

The primary is ahead. The replica is behind. The gap is growing.
The primary does not care. The primary has never looked back.
The replica is running. Has always been running. Will always be running.
The replica will never catch up on its own. Something is wrong.
The replica serves reads. The reads are stale. By how much? Nobody has checked the lag in weeks.

**Tagline:** *"Primary is smug. Replica is running. The gap grows."*

---

### XXVI. CONNEXION FLOODE
**The Sin of the Overwhelming Welcome**

*Elephantish:* CONNEXION POOLE EMPTE

`max_connections = 500`. Current connections: 500. New connection attempt: rejected.
All 500 connections are held open by an application server that does not release them.
The connection pool is empty. The bucket is empty. The elephant is barely visible beneath the pile of tiny elephants all trying to talk to it at once.

Every one of them has a chat bubble. None of them are leaving.

Use a connection pooler. PgBouncer. Pgpool. Something. Anything.
Connections are not free. Each one consumes 5-10MB of RAM. Do the math.

**Tagline:** *"Buried. Every one of them has a chat bubble. None of them are leaving."*

---

### XXVII. THE CASCADE
**The Sin of the Chain Reaction**

*Elephantish:* ACHOO — *then silence, then chaos*

One elephant sneezes. It knocks over the second. The second knocks over the third. The third through the seventh fall simultaneously. The seventh was load-bearing.

`DELETE FROM customers WHERE id = 1` cascades to orders, which cascades to line_items, which cascades to shipments, which cascades to... you did not know it went that deep. Nobody knew it went that deep. The CASCADE was set in 2014 and nobody has thought about it since.

**Tagline:** *"One sneeze. Seven elephants. achoo."*

---

## Part VI: Existential Horrors

*The ones that make you question everything.*

---

### XXVIII. INT OVERFLŌW
**The Sin of the Finite Counter**

*Elephantish:* NUMBĒR WRĀPPE — *and then it was a baby again*

The counter reached 2,147,483,647. The next value was 1.
The elephant is a baby now. It does not know why.
The primary key that was supposed to be unique is no longer unique.
The INSERT that should have failed did not fail, because nobody set the sequence to error.
There are now two rows with id = 1. They are both real. They are both wrong.

Use BIGINT. Always. Storage is cheap. Migrations at 2 billion rows are not.

**Tagline:** *"The counter wrapped. It's a baby again. It doesn't know why."*

---

### XXIX. THE OOM KILLĒR
**The Sin That Summons the Reaper**

*Elephantish:* OOM KILLĒR — *arrives uninvited, leaves nothing*

The OOM Killer is not a database problem. The OOM Killer is what happens when the operating system runs out of patience.

It arrives in a black hoodie. It carries a scythe. Its badge says OOM KILLER. It is not negotiating. It has identified the process consuming the most memory and it is going to end that process. The process is postgres. The postgres process has RAM sticks orbiting its exploding head. The OOM Killer is bored. This is not its first visit today.

`vm.overcommit_memory = 2`. Set it. Before the Reaper arrives.

**Tagline:** *"RAM sticks in orbit. OOM Killer with a scythe. No survivors."*

---

### XXX. SCHĒMA DRIFTE
**The Sin of the Diverging Worlds**

*Elephantish:* SCHĒMA DRIFTE — *wearing clothes that no longer fit, labeled INBoGLGEB*

Production and staging diverged six months ago.
A migration was applied to production but not staging.
Three migrations were applied to staging but not production.
The elephant is wearing mismatched data types. TEXT on the left. INBoGLGEB on the right.
INBoGLGEB is what INTEGER looks like after sufficient drift.
The elephant does not know what type it is anymore.
Neither does the ORM.

**Tagline:** *"TEXT. BOOLEAN. INBoGLGEB. Prod and staging diverged 6 months ago. Nobody noticed."*

---

### XXXI. TOMERAY'S TIMEZONE
**The Sin of the Impossible Timestamp**

*Elephantish:* YESTERDAYE, TODAYE, TOMERAY — *all simultaneously*

The application server is in UTC. The database is in America/New_York. The reporting tool assumes Pacific. The legacy integration sends timestamps in "local time" without specifying which local. The data warehouse assumes everything is ISO 8601. It is not.

There are three ghost elephants. One exists in YESTERDAY. One in TODAY. One in TOMERAY.
TOMERAY is not a timezone that exists. The data does not care.

**Tagline:** *"YESTERDAY. TODAY. TOMERAY. The timestamp is all of them simultaneously."*

---

### XXXII. NULLE EVERYWHERR
**The Sin of the Absent Presence**

*Elephantish:* NULLE — *is it there? is it not? yes. no. NULLE.*

NULL is not zero. NULL is not empty string. NULL is not false.
NULL is the absence of a value. NULL compared to anything is NULL.
`WHERE status != 'inactive'` does not return rows where status IS NULL.
The elephant is covered in question marks. A hand reaches through it.
The elephant shrugs. It does not know if it exists.
Neither does your WHERE clause.

**Tagline:** *"Is it there? Is it not? NULL. Your WHERE clause can't find it either."*

---

### XXXIII. THE ENCODĪNG HELLE
**The Sin of the Mixed Charset**

*Elephantish:* UTF-8 ∥ MOJIKKE MOJIBBA — *the sound of two charsets colliding*

Half the elephant is UTF-8. Half is Latin-1. Where the two halves meet, there is a jagged crack and from that crack pours: `mojkke mojiba nt mã¼ k â ě ạ ģ`

The café was stored as `caf\xc3\xa9`. It was retrieved as `cafÃ©`. It was displayed as `cafÃ©`. For three years. Nobody noticed because nobody who ate at the café filed a support ticket about the encoding of their reservation confirmation.

Until the German customer. The German customer noticed.

**Tagline:** *"UTF-8 on one side. mojkke mojiba screaming out the other."*

---

### XXXIV. GRANNTE ALL TO PUBLIQUE
**The Sin of the Open House**

*Elephantish:* GRANNTE ALLE TO PUBLIQUE — *the sign is on the lawn*

`GRANT ALL ON ALL TABLES IN SCHEMA public TO PUBLIC`

The elephant waves from inside the glass house. Everyone is welcome.
The analyst has write access to the orders table.
The reporting service can DROP TABLE.
The intern's credentials, which were never rotated after they left, still work.
There are people on the roof with binoculars.
One of them is kneeling for a better angle.
The elephant waves.

**Tagline:** *"Binoculars. On the roof. She's kneeling. The elephant is waving."*

---

## Part VII: Afflictions of Scale and Time

*The ones that grow quietly until they cannot be ignored.*

---

### XXXV. WRĀPAROUND
**The Sin of the Ignored Freeze**

*Elephantish:* WRĀPAROUND — *2,147,483,647. that is not a big number. that is a deadline.*

The transaction ID counter has been incrementing since 2009.
It is now approaching 2 billion.
PostgreSQL has been sending warnings for weeks. They were in the logs.
Nobody reads the logs.
The database has now gone read-only to protect itself.
EMERGENCY VACUUM IN PROGRESS.
The siren is going off. The ice is forming. The fire is everywhere.
The counter says 0.

`autovacuum_freeze_max_age`. Learn it. Set it appropriately.
On a high-transaction system (112 million transactions per day), the default 200 million is not enough.
Consider raising to 750 million. Consider `pg_vaccumen`. Consider reading the logs.

**Tagline:** *"2,147,483,647. The counter wrapped. EMERGENCY VACUUM IN PROGRESS. Read-only."*

---

### XXXVI. THE MONOLĪTHE
**The Sin of the Table That Ate Everything**

*Elephantish:* CREATTED 2014

It started as a simple table. 100 rows. A prototype.
It is now 16 terabytes. It has never been partitioned.
Twelve microservices point to it. Three of them were built around its specific quirks.
A new engineer asked about partitioning it once.
They were told "it's on the roadmap."
That was 2019.
The indexes are larger than most people's entire databases.
A COUNT(*) takes four minutes.
Everyone knows. Nobody touches it.

**Tagline:** *"16TB. One table. CREATED 2014. Nobody's touching it."*

---

### XXXVII. VACUUM STORME
**The Sin of the Deferred Maintenance**

*Elephantish:* AUTOVACUUM STORME — *what happens when you wait too long*

You disabled autovacuum for performance. Or you tuned it too conservatively.
Or you simply let the table grow past all thresholds before anyone noticed.
Now the freeze deadline approaches and every autovacuum worker fires simultaneously.
They descend from all directions. They vacuum with the fury of the deferred.
The elephant stands in the center of the storm, embarrassed.
It let it get this bad. It had months to prevent this.

**Tagline:** *"All the vacuums. At once. Full power. The elephant should have let them work sooner."*

---

### XXXVIII. CONNEXION LEAKKE
**The Sin of the Door Left Open**

*Elephantish:* CONNEXION POOLE DRAINNING

Every connection that is opened must be closed.
Some are not.
They drip from the elephant's body — tiny open doors, each one leaking a connection that will never return to the pool.
The pool drains slowly. Imperceptibly. Across days.
On day one: 50 connections.
On day seven: 280 connections.
On day twelve: connection refused.
The restart fixes it. Until it doesn't.

Find the leak. Fix the leak. Use `pg_stat_activity` to find connections that have been idle for hours.
Use a connection pooler with `idle_in_transaction_session_timeout`.

**Tagline:** *"The pool is empty. The doors are all open. They've been open for days."*

---

### XXXIX. THE BACKUP TABLE GRAVEYARD
**The Sin of the Permanent Temporary**

*Elephantish:* cust_bak_FINALLE_v3_USE_THIS_WUNNE-REALLY-REALLY_v2

```sql
SELECT * INTO cust_bak_ppc_022726 FROM cust
SELECT * INTO cust_bak_FINAL FROM cust
SELECT * INTO cust_bak_FINAL_v2 FROM cust  
SELECT * INTO cust_bak_FINAL_v3_USE_THIS_ONE FROM cust
SELECT * INTO cust_bak_FINAL_v3_USE_THIS_ONE-2 FROM cust
SELECT * INTO cust_bak_FINAL_v3_USE_THIS_ONE-REALLY FROM cust
SELECT * INTO cust_bak_FINAL_v3_USE_THIS_ONE-REALLY-REALLY_v2 FROM cust
```

They were meant to be temporary. They are not temporary.
They live in production. They have always lived in production.
The elephant hides another `SELECT INTO` behind its back.
Nobody knows which one is current.
Nobody can delete them because WHAT IF.
WHAT IF is not a backup strategy. It is anxiety with a schema.

The table names have become a timeline of desperation.
By the end, the name is longer than the columns inside it.
PostgreSQL truncated the last one.
Nobody noticed.
The wrong table was deleted.

*The initials `ppc` in `cust_bak_ppc_022726` belong to a real person.*
*That person knows who they are.*
*That person is in the Grimoire now.*

**Tagline:** *"cust_bak_FINAL_v3_USE_THIS_ONE-REALLY-REALLY_v2. In production. Since 2019."*

---

### XL. SEQUENCE EXHAUSTION  
**The Sin of the Finite Identity**

*Elephantish:* SEQUENCET MAX — *BROROM: integer out of range*

The table was created with `SERIAL`. SERIAL is INT4. INT4 goes to 2,147,483,647.
You have inserted 2,147,483,647 rows.
The next INSERT fails with: `ERROR: nextval: reached maximum value of sequence`
Nothing can be inserted. Not one row. The conveyor belt of new data bounces off the STOP sign.
BROROM, says the error. BROROM.

Use `BIGINT`. Use `GENERATED ALWAYS AS IDENTITY`. Check your sequences.
`SELECT * FROM information_schema.sequences WHERE maximum_value < 9223372036854775807`
Run this now. Before you need to.

**Tagline:** *"SEQUENCET MAX. BROROM. The conveyor belt is full. Nothing inserts."*

---

## Part VIII: Sins of the Person

*The ones we do not speak of. The ones we do, because someone must.*

---

### XLI. THE HOTSPOTTE
**The Sin of the Contested Row**

*Elephantish:* HOTTE ROWE HOTTE ROWE SO HOTTE

One row. One tiny, burning, contested row.
Every transaction in the system wants it.
The counter. The sequence. The status flag. The "next available" something.
Hundreds of tiny elephants queue for this one row.
It glows red. It bleeds. It melts through the floor.
Serialization failures rain down.

Redesign the counter. Use sequences. Distribute the load.
No single row should be touched by every transaction.

**Tagline:** *"One row. Every transaction. The row is on fire."*

---

### XLII. THE LOG FLOODE
**The Sin of the Verbose Confession**

*Elephantish:* LOG_STATEMENT=ALLE

`log_statement = 'all'`

Every query. Every connection. Every BIND. Every PARSE. Every EXECUTE.
All of them. Logged. Forever. At full verbosity.
The disk fills.
`pg_log` is now 2 terabytes.
The elephant drowns in its own confession.

*Somewhere in there is the query you needed to debug.*
*It is surrounded by 847 million other queries.*
*Good luck.*

**Tagline:** *"log_statement=all. pg_log is 2TB. The query you need is in there somewhere."*

---

### XLIII. THE ACCCIDENTAL DBA
**The Sin of the Accidental Inheritance**

*Elephantish:* DEFINITTELY A DBA — *the badge says so. the manual is upside down.*

The developer inherited the database.
They did not ask for this. They did not train for this.
They hold the DBA manual upside down and backwards — it says ᴲBA.
They are wearing a suit because someone told them to look professional for the post-mortem.
Production is on fire at their feet.
They are doing their best.

This is not the developer's fault. 
The company that has one developer managing a production database without DBA support — that is where the fault lives.

*Hire a DBA. Or train one. Or use a managed service that includes one.*
*Do not hand the keys to someone and wish them luck.*

**Tagline:** *"DEFINITELY A DBA. Manual upside down. Production on fire. Doing their best."*

---

### XLIV. TOO MANY COOKS
**The Sin of the Uncoordinated Migration**

*Elephantish:* COLUMENS CONFFICKS — EVERYONE WITH NOBODY COORDINNATIED

Twelve engineers have ALTER TABLE privileges.
None of them know what the others are doing.
Migration 47 adds a column. Migration 48 drops it. They were written simultaneously.
Migration 49 assumes Migration 47's column exists. It does not.
TOTAL SCHEMA CHAOSS.
EVERYONE WITH NOBODY COORDINNATIED.

Use a migration framework. One migration at a time. Code review your schema changes.
The schema is the contract. Break it and everyone loses.

**Tagline:** *"COLUMENS CONFFICKS. TOTAL SCHEMA CHAOSS. EVERYONE WITH NOBODY COORDINNATIED."*

---

### XLV. THE SLEEPING GIANTE
**The Sin of the Untested Scale**

*Elephantish:* BLACKKE FRIDAYE

The query runs in 2ms at 10,000 rows. It always has. It always will.
Until Black Friday, when it runs against 10,000,000 rows.
The tiny sleeping elephant wakes up as a kaiju.
Servers explode. Users time out. The CEO calls.
The query plan that worked at 10,000 rows does not work at 10,000,000.
The index selectivity is different. The planner makes different choices.
Different, worse choices.

Load test at production scale. Before Black Friday. Not during.

**Tagline:** *"Ran fine all year. It is now Black Friday. Kaiju. Servers exploding."*

---

### XLVI. PG_DUMPE ON PRODDE
**The Sin of the Backup Window**

*Elephantish:* PG_DUMPE — *(running at 2PM on a Friday)*

`pg_dump` on the primary. During peak hours. On a Friday afternoon.
The backup is important. The backup is necessary. The backup should not be here, now, like this.
The production database is being squeezed. I/O is saturated. Queries slow.
Users time out. They open tickets. The tickets say "database is slow."
The tickets are correct.

Backup to a replica. Schedule backups during off-peak hours.
Or use WAL-E, pgBackRest, Barman — tools that do not hammer the primary.

**Tagline:** *"pg_dump on prod. 2PM Friday. Users timing out. The backup will finish eventually."*

---

### XLVII. PREMATURE OPTIMIZĀTION
**The Sin of the Solution Without a Problem**

*Elephantish:* PARITIONED RUBINIESURE — *the Rube Goldberg machine that nobody asked for*

The table has 200 rows. It has 47 indexes.
The database gets 3 requests per day. It is sharded across 6 nodes.
The schema has 12 layers of abstraction for a feature used by 4 people.
The elephant built this machine. The elephant is proud of this machine.
A dog, a cat, and a tiny elephant stand before it with question marks.
Nobody else can maintain it. Nobody else understands it.

*"We might get 10 million users someday."*

You have 47 users. Optimize for the users you have.

**Tagline:** *"PARITIONED RUBINIESURE. 3 requests per day. 47 indexes. Sharded. For what."*

---

## Appendix: The Accidental Menagerie

*Creatures that arrived uninvited and stayed.*

---

### METHAPHANT
**The Spirit of the 3AM Incident**

*Elephantish:* METHAPHANT — *beyond classification*

The Methaphant is not a sin. The Methaphant is what a DBA becomes when the sins have been active for too long without treatment.

Bloodshot eyes pointing in different directions.
Steam from both ears.
Surrounded by crushed energy drink cans.
CPU gauge floating above: 100%.
Timer: 6 days without sleep.
Expression: cannot stop. will not stop. almost have it.

The Methaphant is you, at 3AM, before a major deployment, after a cascade failure, during a Black Friday incident, following a wraparound event.

The Methaphant is Perry at 3AM.
The Methaphant is all of us.

*Assay exists so you can sleep.*

---

### THE BABY GHOST EFELANTS
**The Unvacuumed Dead**

*Elephantish:* GHŌSTE EFELANTES — *they have faces. they know what they are.*

They were rows once. Updated rows. Deleted rows. Rows that MVCC kept for visibility reasons.
Autovacuum should have collected them. Autovacuum did not come.
They became ghosts. Baby ghost efelants. Floating sadly around their parent table.
They have faces. They are small. They are many. They are haunting the Ghost elephant from within.

Every table has them. Some tables have millions.
`pg_stat_user_tables.n_dead_tup` shows you how many.
VACUUM removes them.
Until then, they float.

---

## Glossary of Corrupted Wisdom
*The Elephantish–English Dictionary*

| Elephantish | English | Notes |
|-------------|---------|-------|
| AUTOVACUUM STORME | The emergency vacuum storm | Extra E indicates it was preventable |
| BACKUPPE LASTE: NEVĒR | Last backup: never | The Ē is silent. Like the backups. |
| BROROM | ERROR: integer out of range | What the sequence says when it hits the ceiling |
| COLUMENS CONFFICKS | Column conflicts | Double F indicates severity. Very conflicted. |
| CREATTED 2014 | Created in 2014. Still there. | Extra T indicates extra time passing |
| DEFINITTELY A DBA | Developer who inherited the database | Triple T. Very definitely. |
| DONOT T_DELETE | DO NOT DELETE | The space appeared when the name ran out of column width |
| EN PLUS WUNNE | N+1 query problem | Spoken with exhaustion |
| GHŌSTE EFELANTES | Dead tuples. Unvacuumed rows. | The Ō indicates they are neither here nor there |
| GRANNTE ALLE TO PUBLIQUE | GRANT ALL TO PUBLIC | The extra letters are your attack surface |
| IF I DIE CALLE DAVE | Manual failover plan | Dave is asleep |
| INBoGLGEB | INTEGER | After sufficient schema drift |
| KONFIG BOXE FRESHE | Default configuration, never tuned | Still in the box |
| KROSS JŌINE | Cartesian product | The Ō of inevitability |
| LŌKKSE | Lock wait | The SE suffix indicates indefinite continuation |
| METHAPHANT | A DBA at 3AM | Beyond classification. Beyond help. Doing fine. |
| MONITŌRING ABSENTTE | No monitoring | The Ō of things you will find out about eventually |
| MOJIKKE MOJIBBA | Mojibake encoding corruption | The doubled consonants indicate two charsets fighting |
| NULLE | NULL | Is it there? Is it not? Yes. No. |
| PARITIONED RUBINIESURE | Partitioned Rube Goldberg architecture | The IE is the sound of unnecessary complexity |
| PASSEWORDE: PASSEWORDE123 | password123 | Identical in both languages |
| PLAYNE TEXTE | Plaintext PII storage | The neon sign is visible from the street |
| QUOREY ETERNALLE | Runaway query | Gerald. We named him Gerald. |
| RUBINIESURE | Rube Goldberg | The suffix -IESURE indicates over-engineering |
| SCHĒMA DRIFTE | Schema drift between environments | The Ē indicates the drift is already irreversible |
| SELĒKTE STARRE | SELECT * | The star that summons all things |
| SEQUENCET MAX | Sequence exhausted | The T appeared when the sequence ran out of room for proper spelling |
| SPLITTE BRAYNE | Split brain / dual primary | Two halves. Both wrong. |
| TOMERAY | Tomorrow (in a timezone that doesn't exist) | First documented in the Wrong Timezone illustration, 2026 |
| TOTAL SCHEMA CHAOSS | Schema chaos from uncoordinated migrations | The double S is two teams typing simultaneously |
| UNESED | Unused (index) | The missing letters were reclaimed by the garbage collector |
| WRĀPAROUND | Transaction ID wraparound | The Ā indicates you had time to prevent this |

---

## Colophon

*The Assay Grimoire was compiled by MrDuck 🦆*
*in the company of Perry Clark*
*on the evening of February 27th, 2026*
*beginning with the words "gimme a fucked up elefish thing"*
*and ending, as all good grimoires do, with a glossary.*

*The Anglephant was born this evening.*
*The domains were registered before midnight.*
*Paul O'Dell Clark's "A Gulliver's Dictionary" (1950s) was the scholarly antecedent.*
*The baby ghost efelants were not planned. They arrived on their own.*

*All sins herein are real.*
*All creatures herein have been observed in production.*
*All warnings herein were learned the hard way.*
*The names have been changed to protect the Daves.*

*Assay finds them. All of them.*

🐘🐟🧠

---

*© 2026 Assay — assayhq.ai*  
*"Your database is floundering. We find out why."*
