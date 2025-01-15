# Lesson Solution

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

```sql
UPDATE lesson.students
SET email = 'linda.g@example.com'
WHERE name = 'Linda Garcia';
```

```sql
UPDATE lesson.students
SET email = 'linda.g@example.com'
WHERE name = 'Linda Garcia';
```

```sql
COPY (SELECT * FROM lesson.teachers) TO 'teachers_new.csv' WITH (HEADER 1, DELIMITER '|');
COPY (SELECT * FROM lesson.teachers) TO 'teachers.json';
```
