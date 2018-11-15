check aftet_commit (only in rails 3) and  after_create 

Before
It's common to generate a background job to send emails, tweets or post to facebook wall, like

class Notification < ActiveRecord::Base
  after_create :asyns_send_notification

  def async_send_notification
    NotificationWorker.async_send_notification({:notification_id => id})
  end
end

class NotificationWorker < Workling::Base
  def send_notification(params)
    notification = Notification.find(params[:notification_id])
    user = notification.user
    # send notification to user's friends by email
  end
end
It looks fine, every time it creates a notification, generates an asynchronous worker, assigns notification_id to the worker, in the worker it finds the notification by id, then sends notification by email.

You won't see any issue in development, as local db can commit fast. But in production server, db traffic might be huge, worker probably finish faster than transaction commit. e.g.

main process	worker process
BEGIN	
INSERT INTO notifications(message, user_id) values('notification message', 1)	
# return id 10 for newly-created notification	

SELECT * FROM notifications WHERE id = 10
COMMIT	
In this case, the worker process query the newly-created notification before main process commits the transaction, it will raise NotFoundError, because transaction in worker process can't read uncommitted notification from transaction in main process.

Refactor
So we should tell activerecord to generate notification worker after notification insertion transaction committed.

class Notification < ActiveRecord::Base
  after_commit :asyns_send_notification, :on => :create

  def async_send_notification
    NotificationWorker.async_send_notification({:notification_id => id})
  end
end
Now the transactions order becomes

main process	worker process
BEGIN	
INSERT INTO notifications(message, user_id) values('notification message', 1)	
# return id 10 for newly-created notification	
COMMIT	

SELECT * FROM notifications WHERE id = 10
Worker process won't receive NotFoundErrors any more.


Active Record Transactions

Transactions are protective blocks where SQL statements are only permanent if they can all succeed as one atomic action. The classic example is a transfer between two accounts where you can only have a deposit if the withdrawal succeeded and vice versa. Transactions enforce the integrity of the database and guard the data against program errors or database break-downs. So basically you should use transaction blocks whenever you have a number of statements that must be executed together or not at all.

For example:

ActiveRecord::Base.transaction do
  david.withdrawal(100)
  mary.deposit(100)
end
This example will only take money from David and give it to Mary if neither withdrawal nor deposit raise an exception. Exceptions will force a ROLLBACK that returns the database to the state before the transaction began. Be aware, though, that the objects will not have their instance data returned to their pre-transactional state.

Different Active Record classes in a single transaction

Though the transaction class method is called on some Active Record class, the objects within the transaction block need not all be instances of that class. This is because transactions are per-database connection, not per-model.

In this example a balance record is transactionally saved even though transaction is called on the Account class:

Account.transaction do
  balance.save!
  account.save!
end
The transaction method is also available as a model instance method. For example, you can also do this:

balance.transaction do
  balance.save!
  account.save!
end
Transactions are not distributed across database connections

A transaction acts on a single database connection. If you have multiple class-specific databases, the transaction will not protect interaction among them. One workaround is to begin a transaction on each class whose models you alter:

Student.transaction do
  Course.transaction do
    course.enroll(student)
    student.units += course.units
  end
end
This is a poor solution, but fully distributed transactions are beyond the scope of Active Record.

save and destroy are automatically wrapped in a transaction

Both save and destroy come wrapped in a transaction that ensures that whatever you do in validations or callbacks will happen under its protected cover. So you can use validations to check for values that the transaction depends on or you can raise exceptions in the callbacks to rollback, including after_* callbacks.

As a consequence changes to the database are not seen outside your connection until the operation is complete. For example, if you try to update the index of a search engine in after_save the indexer won’t see the updated record. The after_commit callback is the only one that is triggered once the update is committed. See below.

Exception handling and rolling back

Also have in mind that exceptions thrown within a transaction block will be propagated (after triggering the ROLLBACK), so you should be ready to catch those in your application code.

One exception is the ActiveRecord::Rollback exception, which will trigger a ROLLBACK when raised, but not be re-raised by the transaction block.

Warning: one should not catch ActiveRecord::StatementInvalid exceptions inside a transaction block. ActiveRecord::StatementInvalid exceptions indicate that an error occurred at the database level, for example when a unique constraint is violated. On some database systems, such as PostgreSQL, database errors inside a transaction cause the entire transaction to become unusable until it's restarted from the beginning. Here is an example which demonstrates the problem:

# Suppose that we have a Number model with a unique column called 'i'.
Number.transaction do
  Number.create(:i => 0)
  begin
    # This will raise a unique constraint error...
    Number.create(:i => 0)
  rescue ActiveRecord::StatementInvalid
    # ...which we ignore.
  end

  # On PostgreSQL, the transaction is now unusable. The following
  # statement will cause a PostgreSQL error, even though the unique
  # constraint is no longer violated:
  Number.create(:i => 1)
  # => "PGError: ERROR:  current transaction is aborted, commands
  #     ignored until end of transaction block"
end
One should restart the entire transaction if an ActiveRecord::StatementInvalid occurred.

Nested transactions

transaction calls can be nested. By default, this makes all database statements in the nested transaction block become part of the parent transaction. For example, the following behavior may be surprising:

User.transaction do
  User.create(:username => 'Kotori')
  User.transaction do
    User.create(:username => 'Nemu')
    raise ActiveRecord::Rollback
  end
end
creates both “Kotori” and “Nemu”. Reason is the ActiveRecord::Rollback exception in the nested block does not issue a ROLLBACK. Since these exceptions are captured in transaction blocks, the parent block does not see it and the real transaction is committed.

In order to get a ROLLBACK for the nested transaction you may ask for a real sub-transaction by passing :requires_new => true. If anything goes wrong, the database rolls back to the beginning of the sub-transaction without rolling back the parent transaction. If we add it to the previous example:

User.transaction do
  User.create(:username => 'Kotori')
  User.transaction(:requires_new => true) do
    User.create(:username => 'Nemu')
    raise ActiveRecord::Rollback
  end
end
only “Kotori” is created. (This works on MySQL and PostgreSQL, but not on SQLite3.)

Most databases don’t support true nested transactions. At the time of writing, the only database that we’re aware of that supports true nested transactions, is MS-SQL. Because of this, Active Record emulates nested transactions by using savepoints on MySQL and PostgreSQL. See dev.mysql.com/doc/refman/5.0/en/savepoint.html for more information about savepoints.

Callbacks

There are two types of callbacks associated with committing and rolling back transactions: after_commit and after_rollback.

after_commit callbacks are called on every record saved or destroyed within a transaction immediately after the transaction is committed. after_rollback callbacks are called on every record saved or destroyed within a transaction immediately after the transaction or savepoint is rolled back.

These callbacks are useful for interacting with other systems since you will be guaranteed that the callback is only executed when the database is in a permanent state. For example, after_commit is a good spot to put in a hook to clearing a cache since clearing it from within a transaction could trigger the cache to be regenerated before the database is updated.

Caveats

If you’re on MySQL, then do not use DDL operations in nested transactions blocks that are emulated with savepoints. That is, do not execute statements like ‘CREATE TABLE’ inside such blocks. This is because MySQL automatically releases all savepoints upon executing a DDL operation. When transaction is finished and tries to release the savepoint it created earlier, a database error will occur because the savepoint has already been automatically released. The following example demonstrates the problem:

Model.connection.transaction do                           # BEGIN
  Model.connection.transaction(:requires_new => true) do  # CREATE SAVEPOINT active_record_1
    Model.connection.create_table(...)                    # active_record_1 now automatically released
  end                                                     # RELEASE savepoint active_record_1
                                                          # ^^^^ BOOM! database error!
end
Note that “TRUNCATE” is also a MySQL DDL statement!