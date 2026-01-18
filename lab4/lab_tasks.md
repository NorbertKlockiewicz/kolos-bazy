Exercise 1: Non-repeatable reads
In this exercise, please use PostgreSQL's default transaction isolation level – read committed.

In your table, develop a scenario in which two concurrent transactions cause the database to end up in an undesirable state because one of the transactions reads the same row more than once and receives different values in subsequent reads.

Put the scenario in your lab report.

Exercise 2: Explicit locking
Without increasing the isolation level (yet), use table-level locks on selected database objects to maintain database integrity in your selected scenario.

Put the scenario in your lab report.

Exercise 3: Repeatable read isolation level
Redo the scenario from Exercise 1 (without explicit locking) with the transaction isolation level set to repeatable read.

Was the problem resolved?

Put the scenario in your report, indicating any significant bits, e.g. when one transaction was waiting for another one to finish.

Exercise 4: Serialisation errors
Try to generate a serialisation error at the repeatable read isolation level.

In your report, include:

the scenario, along with comments,
the error that you received.
Exercise 5: Record-level locks
Try to generate a serialisation error, but first create a record-level lock in one of the transactions (SELECT FOR UPDATE/SELECT FOR SHARE) and notice how the other one behaves.

Put the scenario in your report.

