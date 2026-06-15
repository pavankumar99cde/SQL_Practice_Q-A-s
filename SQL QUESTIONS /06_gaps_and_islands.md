# Gaps and Islands SQL Interview Questions

Gaps and Islands is one of the most frequently asked SQL topics in Data Engineer interviews.

---

## Sample Table

```sql
User_Logins
-----------
user_id     INT
login_date  DATE
```

---

## 1. Find users who logged in for 3 consecutive days

```sql
-- Solution 1: LAG()
WITH LoginHistory AS (
    SELECT user_id,
           login_date,
           LAG(login_date,1) OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) prev1,
           LAG(login_date,2) OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) prev2
    FROM User_Logins
)
SELECT DISTINCT user_id
FROM LoginHistory
WHERE login_date = prev1 + INTERVAL '1 day'
AND prev1 = prev2 + INTERVAL '1 day';
```

```sql
-- Solution 2: Self Join
SELECT DISTINCT l1.user_id
FROM User_Logins l1
JOIN User_Logins l2
ON l1.user_id = l2.user_id
AND l1.login_date = l2.login_date + INTERVAL '1 day'
JOIN User_Logins l3
ON l1.user_id = l3.user_id
AND l2.login_date = l3.login_date + INTERVAL '1 day';
```

---

## 2. Find the longest consecutive login streak

```sql
-- Solution 1: Row Number Trick
WITH Streaks AS (
    SELECT user_id,
           login_date,
           login_date -
           ROW_NUMBER() OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) * INTERVAL '1 day' grp
    FROM User_Logins
)
SELECT user_id,
       COUNT(*) streak_length
FROM Streaks
GROUP BY user_id, grp
ORDER BY streak_length DESC;
```

```sql
-- Solution 2: Dense Rank Method
WITH LoginGroups AS (
    SELECT *,
           DATE_SUB(
               login_date,
               INTERVAL DENSE_RANK() OVER(
                   PARTITION BY user_id
                   ORDER BY login_date
               ) DAY
           ) grp
    FROM User_Logins
)
SELECT user_id,
       COUNT(*) streak_length
FROM LoginGroups
GROUP BY user_id, grp;
```

---

## 3. Find missing dates in a sequence

```sql
-- Solution 1: Recursive CTE
WITH RECURSIVE DateSeries AS (
    SELECT MIN(login_date) dt
    FROM User_Logins

    UNION ALL

    SELECT dt + INTERVAL '1 day'
    FROM DateSeries
    WHERE dt <
    (
        SELECT MAX(login_date)
        FROM User_Logins
    )
)
SELECT dt
FROM DateSeries
WHERE dt NOT IN (
    SELECT login_date
    FROM User_Logins
);
```

```sql
-- Solution 2: Calendar Table
SELECT c.calendar_date
FROM Calendar c
LEFT JOIN User_Logins u
ON c.calendar_date = u.login_date
WHERE u.login_date IS NULL;
```

---

## 4. Find gaps between consecutive login dates

```sql
-- Solution 1: LAG()
SELECT user_id,
       login_date,
       login_date -
       LAG(login_date) OVER(
           PARTITION BY user_id
           ORDER BY login_date
       ) AS gap_days
FROM User_Logins;
```

```sql
-- Solution 2: Self Join
SELECT u1.user_id,
       u1.login_date,
       MAX(u2.login_date) previous_login,
       u1.login_date - MAX(u2.login_date) gap_days
FROM User_Logins u1
LEFT JOIN User_Logins u2
ON u1.user_id = u2.user_id
AND u2.login_date < u1.login_date
GROUP BY u1.user_id,
         u1.login_date;
```

---

## 5. Group consecutive records into islands

```sql
-- Solution 1
WITH Islands AS (
    SELECT *,
           login_date -
           ROW_NUMBER() OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) * INTERVAL '1 day' grp
    FROM User_Logins
)
SELECT user_id,
       MIN(login_date) start_date,
       MAX(login_date) end_date
FROM Islands
GROUP BY user_id, grp;
```

```sql
-- Solution 2
WITH Islands AS (
    SELECT *,
           SUM(is_new_group) OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) grp
    FROM (
        SELECT *,
               CASE
                   WHEN login_date -
                        LAG(login_date) OVER(
                            PARTITION BY user_id
                            ORDER BY login_date
                        ) > 1
                   THEN 1
                   ELSE 0
               END is_new_group
        FROM User_Logins
    ) x
)
SELECT *
FROM Islands;
```

---

## 6. Find users inactive for more than 30 days

```sql
-- Solution 1
WITH UserGap AS (
    SELECT user_id,
           login_date,
           login_date -
           LAG(login_date) OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) gap_days
    FROM User_Logins
)
SELECT *
FROM UserGap
WHERE gap_days > 30;
```

```sql
-- Solution 2
SELECT u1.user_id,
       u1.login_date
FROM User_Logins u1
JOIN User_Logins u2
ON u1.user_id = u2.user_id
WHERE DATEDIFF(
      day,
      u2.login_date,
      u1.login_date
      ) > 30;
```

---

## 7. Find first date of every streak

```sql
-- Solution 1
WITH Streaks AS (
    SELECT *,
           CASE
               WHEN LAG(login_date) OVER(
                    PARTITION BY user_id
                    ORDER BY login_date
               ) IS NULL
               OR login_date -
                  LAG(login_date) OVER(
                    PARTITION BY user_id
                    ORDER BY login_date
                  ) > 1
               THEN 1
               ELSE 0
           END streak_start
    FROM User_Logins
)
SELECT *
FROM Streaks
WHERE streak_start = 1;
```

```sql
-- Solution 2
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) rn
    FROM User_Logins
) x
WHERE rn = 1;
```

---

## 8. Find last date of every streak

```sql
-- Solution 1
WITH Streaks AS (
    SELECT *,
           CASE
               WHEN LEAD(login_date) OVER(
                    PARTITION BY user_id
                    ORDER BY login_date
               ) IS NULL
               OR LEAD(login_date) OVER(
                    PARTITION BY user_id
                    ORDER BY login_date
               ) - login_date > 1
               THEN 1
               ELSE 0
           END streak_end
    FROM User_Logins
)
SELECT *
FROM Streaks
WHERE streak_end = 1;
```

```sql
-- Solution 2
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY user_id
               ORDER BY login_date DESC
           ) rn
    FROM User_Logins
) x
WHERE rn = 1;
```

---

## 9. Find customers active for 5 consecutive days

```sql
-- Solution 1
WITH Activity AS (
    SELECT customer_id,
           activity_date,
           activity_date -
           ROW_NUMBER() OVER(
               PARTITION BY customer_id
               ORDER BY activity_date
           ) * INTERVAL '1 day' grp
    FROM Customer_Activity
)
SELECT customer_id
FROM Activity
GROUP BY customer_id, grp
HAVING COUNT(*) >= 5;
```

```sql
-- Solution 2
WITH ConsecutiveDays AS (
    SELECT customer_id,
           activity_date,
           DATEDIFF(
               day,
               LAG(activity_date) OVER(
                   PARTITION BY customer_id
                   ORDER BY activity_date
               ),
               activity_date
           ) diff
    FROM Customer_Activity
)
SELECT DISTINCT customer_id
FROM ConsecutiveDays;
```

---

## 10. Find longest streak per user

```sql
-- Solution 1
WITH Islands AS (
    SELECT user_id,
           login_date,
           login_date -
           ROW_NUMBER() OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) * INTERVAL '1 day' grp
    FROM User_Logins
)
SELECT user_id,
       MAX(streak_length)
FROM (
    SELECT user_id,
           grp,
           COUNT(*) streak_length
    FROM Islands
    GROUP BY user_id, grp
) x
GROUP BY user_id;
```

```sql
-- Solution 2
WITH StreakGroups AS (
    SELECT *,
           SUM(new_streak) OVER(
               PARTITION BY user_id
               ORDER BY login_date
           ) grp
    FROM (
        SELECT *,
               CASE
                   WHEN login_date -
                        LAG(login_date) OVER(
                            PARTITION BY user_id
                            ORDER BY login_date
                        ) > 1
                   THEN 1
                   ELSE 0
               END new_streak
        FROM User_Logins
    ) x
)
SELECT user_id,
       MAX(cnt)
FROM (
    SELECT user_id,
           grp,
           COUNT(*) cnt
    FROM StreakGroups
    GROUP BY user_id, grp
) y
GROUP BY user_id;
```

---
