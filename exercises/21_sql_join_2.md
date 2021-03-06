# SQL JOINS

> Download  [this file](school.sql) to your computer (Right click and select "Save link as"). You import it by creating a new database in MAMP and name it whatever, selecting the database in MAMP and go to the tab **Import**. You then select the the downloaded file with **Choose file**. When you have added the file, press **Go** at the bottom and the new tables will be created.

> Use [W3schools.com | SQL Tutorial](https://www.w3schools.com/sql/default.asp) for quick reference

## Övningar

1. Get and join `students` och `courses`
2. Join  `students` and `courses` by comparing `course_id` on the student with the `id` on the `courses` table
3. Choose columns from the previous query so we only get the students name, the course id and the course name.
4. Transform `course_name` to be named `course` and that the students name should just be `name`, use the same query as exercise 3 but modify it.
5. With an SQL-query, count how many students every course has. Only show courses that have students.
6. Count how many students each course has but also show courses that does not have any students.
7. Show every student and which course they are taken. If they are not enrolled on a course the value of the course should be `null`
8. Modify the previous SQL-query so that instead of `null` the value says `"Unenrolled"`
9. Show every course and how many students that are enrolled in every course like below:

| enrolled_students | course     |
| ----------------- | ---------- |
| 1                 | JavaScript |
| 2                 | PHP        |
| 1                 | Unenrolled |
| 0                 | Agile      |
| 0                 | HTML       |



10. Use the table `enrollments` and make a three-way-join. Show a list of which students are enrolled in which courses like below:

| name   | course     |
| ------ | ---------- |
| Ursula | HTML       |
| Steffe | HTML       |
| Steffe | Agile      |
| Waba   | JavaScript |




## Lösningsförslag

1. Everything originates from the `CROSS JOIN`. This `JOIN` puts two tables together in every possible combination. For our example we have 4 courses and 4 students = 16 combinations. This is usually not what we want, we want to JOIN on certain criteries, we want only `INNER JOIN` or sometimes just called `JOIN`. But the same thing will happen if we don't specify the `ON`-statement

```sql
SELECT * FROM students
JOIN courses
```

---

2. We need to have an if-statement that returns the matching columns, where the `id` for the course is the same in both tables. This will only return colums that match and will not return `NULL`-values or empty cells.

```sql
SELECT * FROM students
JOIN courses
ON courses.course_id = students.course_id
```

---

3. We might also want to be more specific and not select everything with the wildcard (**`*`**). This will also solve a problem we have, we now have two columns with `course_id` when we only need one or none. To specificy which columns to get we need to tell SQL from which tables to pick from with dot-notation. We can reference `students` and `courses` "before" we have used the `JOIN`. The `JOIN` happens before the `SELECT`.

```sql
SELECT students.first_name, courses.course_id, courses.course_name
FROM students
JOIN courses
ON courses.course_id = students.course_id
```

---
4. For easier handling of the column names we should rename the columns so it gets properly formatted in _PHP_ and also so we have less to write. We do this with aliasing: `AS`. This means we use `$data["name"]` instead of `$data["first_name"]` when we use it in _PHP_.

```sql
SELECT students.first_name AS name,
courses.course_id,
courses.course_name AS course
FROM students
JOIN courses
ON courses.course_id = students.course_id
```

---

5. Sometimes we want to use [*aggregate*](https://www.w3schools.com/sql/sql_count_avg_sum.asp)-functions such as `SUM()`, `COUNT()`, `MAX()`, `MIN()`. In these cases the column will be named after the aggregate-function which is usually not what we want. We need to use aliasing: `AS` to solve this. Instead of getting `$data["COUNT(students.student_id)"]` we get `$data["enrolled_students"]` which is much more readable and easier to write.

```sql
SELECT COUNT(students.student_id) as enrolled_students,
courses.course_name AS course 
FROM students
JOIN courses
ON courses.course_id = students.course_id
```

But when we use the aggregate-functions we are usually left with _one_ value, it is either the max value in the whole column or the total count of every value in the column. In the example above we get the count of all students, not enrolled students in every course. As we are counting the number of student for each course we must group the result. Otherwise the query will count all the students independent of which course the belong to. We do this by using `GROUP BY` to tell _SQL_ how to count the rows.

```sql
SELECT COUNT(students.student_id) as enrolled_students,
courses.course_name AS course 
FROM students
JOIN courses
ON courses.course_id = students.course_id
GROUP BY course
-- We can reference the alias 'course' instead of 'courses.course_name'
```

---

6. The query above will return the number of enrolled students in every course **if** there are enrolled students in the course. But we have two more courses with 0 students. Maybe we want all courses, even if there are no students. This means that we want ALL the values from one of the tables, this is a good case for `LEFT JOIN`/`RIGHT JOIN`. Returns the whole right table even if there isn't a matching column in the left. We will get a list of all courses and 0 in the left column.

```sql
SELECT COUNT(students.student_id) as enrolled_students,
courses.course_name AS course
FROM students
RIGHT JOIN courses
ON courses.course_id = students.course_id
GROUP BY course
```

---

7. If we instead do a `LEFT JOIN`, we will get the number of unenrolled students. We have a `NULL`-value, which means that the course have no name, but we have students with that `NULL`-value.

```sql
SELECT COUNT(students.student_id) as enrolled_students,
courses.course_name AS course
FROM students
LEFT JOIN courses
ON courses.course_id = students.course_id
GROUP BY course
```

---

8. If we want to rename the empty columns left to us by the JOIN we can use the helper function `IFNULL`. It will return the original value if not `null`, and a value set by us if it is `null`.

```sql
SELECT COUNT(students.student_id) as enrolled, 
IFNULL(courses.course_name, "Unenrolled") as course 
FROM students
LEFT JOIN courses
ON courses.course_id = students.course_id
GROUP BY course
```

---

9. maybe want the both these results above, both the `LEFT JOIN` and the `RIGHT JOIN`. We then have to run both a `LEFT JOIN` AND a `RIGHT JOIN` and mush 'em together with [`UNION`](https://www.w3schools.com/sql/sql_union.asp)

```sql
SELECT COUNT(students.student_id) as enrolled_students,
IFNULL(courses.course_name, "Unenrolled") as course 
FROM `students`
LEFT JOIN `courses` ON `students`.`course_id` = `courses`.`course_id`
GROUP BY course

UNION

SELECT COUNT(students.student_id) as enrolled_students,
IFNULL(courses.course_name, "Unenrolled") as course 
FROM `students`
RIGHT JOIN `courses` ON `students`.`course_id` = `courses`.`course_id`
GROUP BY course
```

---



10. We need to join multiple columns. Here we can join the three tables first, then we can do our `ON`-statement, our control statement (if-statement basically). The `course_id` must be the same AND the `student_id` must be the same in all three tables.

```sql
SELECT students.first_name as name,
courses.course_name as course 
FROM students
JOIN enrollments
JOIN courses
ON enrollments.course_id = courses.course_id 
AND enrollments.student_id = students.student_id
```

---

This syntax will also produce the same result. Instead on doing a double join and having the `AND`.

```sql
SELECT students.first_name as name,
courses.course_name as course 
FROM students
JOIN courses
ON students.course_id = courses.course_id 
JOIN enrollments
ON enrollments.student_id = students.student_id
```


