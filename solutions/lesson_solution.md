### 3.1 Create tables with constraints

```sql
CREATE TABLE lesson.students (
  id INTEGER PRIMARY KEY, -- primary key
  name VARCHAR NOT NULL, -- not null
  address VARCHAR,
  phone VARCHAR,
  email VARCHAR CHECK(CONTAINS(email, '@')), -- check
  class_id INTEGER REFERENCES lesson.classes(id) -- foreign key
);
```

### 5.2 Updating data

```sql
UPDATE lesson.students
SET email = 'linda.g@example.com'
WHERE name = 'Linda Garcia';
```
