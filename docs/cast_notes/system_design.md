# System Design from Semicolon & Son

Notes on system design from [Semicolon&Son](https://www.semicolonandsons.com/series/learn-system-design-for-web).

Data is more important than code, thus, we have to keep its integrity in check.

## NOT NULL constraint on booleans columns

Make boolean columns not nullable with some default value set. Otherwise, you will have issues to remember to consider the `nullability` when querying every time.

## Data constraints at DB level

Always filter and verify the user supplied parameters. However, add database level constraints as well.

Examples:

```SQL
ALTER TABLE users ADD CONSTRAINT check_valid_role_state CHECK ( role IN ('admin', 'superadmin', 'regular'))

ALTER TABLE invoices ADD CONSTRAINT check_valid_amount CHECK (amount >= 0 AND amount <= 1000000 )
```

### Add uniqueness constraints at DB level

Application level uniqueness constraints are usually rubbish, which performs a read and then decides to write or not.

This results in two queries and is suspectible for race conditions on multi threaded environments.

### Add ON DELETE CASCADE to foreign key constraints where children should be removed and performance is crucial

ActiveRecord's dependent: :destroy could be slow. When you care about performance use ON DELETE CASCADE on foreign key references.

```sql
ALTER TABLE users ADD CONSTRAINT fk_user_id FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE CASCADE;
```
