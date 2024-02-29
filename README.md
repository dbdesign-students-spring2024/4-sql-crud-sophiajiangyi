# SQL CRUD
## Sophia Wu
## Part 1: Restaurant finder
### Table structure
```sql
-- create a new database
sqlite3 restaurants.db

-- create restaurants table
CREATE TABLE restaurants (
    id INTEGER PRIMARY KEY,
    restaurant_name TEXT,
    category TEXT,
    price_tier TEXT,
    neighborhood TEXT,
    open_hour TIME,
    close_hour TIME,
    avg_rating INTEGER,
    kids BOOLEAN,
)

-- create reviews table
CREATE TABLE reviews(
    id INTEGER PRIMARY KEY,
    restaurant_id INTEGER,
    review TEXT,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(id)
)
```

### Practice data
```sql
-- import data
.mode csv
.headers on
.import ./data/restaurants.csv restaurants --skip 1
```

### Queries
1. Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example).
```sql
SELECT *
FROM restaurants 
WHERE price_tier = 'Cheap' 
AND neighborhood = 'Soho';
```
2. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.
```sql
SELECT * 
FROM restaurants 
WHERE category = 'Fast Food' 
AND avg_rating >= 3 
ORDER BY avg_rating DESC;
```
3. Find all restaurants that are open now (see hint below).
```sql
SELECT *
FROM restaurants 
WHERE strftime('%H:%M', 'now', 'localtime') BETWEEN open_hour AND close_hour;
```
4. Leave a review for a restaurant (pick any restaurant as an example; note that leaving a review has no automatic effect on the average rating of the restaurant).
```sql
INSERT INTO reviews (restaurant_id, review) 
VALUES (10, 'I love the Onion Soup!');
```
5. Delete all restaurants that are not good for kids.
```sql
DELETE FROM restaurants 
WHERE kids = 'false';
```
6. Find the number of restaurants in each NYC neighborhood.
```sql
SELECT neighborhood, COUNT(id) AS num_restaurants 
FROM restaurants 
GROUP BY neighborhood 
ORDER BY num_restaurants;
```

## Part 2: Social media app
### Tables structure
```sql
/* User Table */
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT,
    passcode TEXT,
    username TEXT
)

/* Post Table*/
CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    sender_id INTEGER,
    receiver_id INTEGER,
    post_type TEXT,
    content TEXT DEFAULT '',
    visibility BOOLEAN DEFAULT True,
    time_posted TIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (sender_id) REFERENCES users(id),
    FOREIGN KEY (receiver_id) REFERENCES users(id)
)
```

### Practice data
```sql
-- import data
.mode csv
.headers on
.import ./data/users.csv users --skip 1 
.import ./data/posts.csv posts --skip 1 
```

### Queries
1. Register a new User.
```sql
INSERT INTO users (email, passcode, username) 
VALUES ('whoisthis@nyu.edu', 'whoisthis??', 'whoamI');
```
2. Create a new Message sent by a particular User to a particular User (pick any two Users for example).
```sql
INSERT INTO posts (sender_id, receiver_id, post_type, content, visibility, time_posted) 
VALUES (10, 20, 'Message', 'Hey', True, CURRENT_TIMESTAMP);
```
3. Create a new Story by a particular User (pick any User for example).
```sql
INSERT INTO posts (sender_id, post_type, content, visibility, time_posted) 
VALUES (30, 'Story', 'Hello', True, CURRENT_TIMESTAMP);
```
4. Show the 10 most recent visible Messages and Stories, in order of recency.
```sql
SELECT *
FROM posts 
WHERE visibility = 'True' 
ORDER BY time_posted DESC 
LIMIT 10;
```
5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.
```sql
SELECT *
FROM posts 
WHERE User_id = 100  
AND Recipient_id = 200 
AND post_type = 'Message' 
AND visibility = 'True' 
ORDER BY time_posted DESC
LIMIT 10;
```
6. Make all Stories that are more than 24 hours old invisible.
```sql
UPDATE posts 
SET visibility = False 
WHERE post_type = 'Story' 
AND ROUND((JULIANDAY('now') - JULIANDA(time_posted)) * 24) > 24;
```
7. Show all invisible Messages and Stories, in order of recency.
```sql
SELECT * 
FROM posts 
WHERE visibility = False 
ORDER BY time_posted DESC;
```
8. Show the number of posts by each User.
```sql
SELECT sender_id, COUNT(*) as total_posts
FROM posts
GROUP BY sender_id
ORDER BY sender_id;
```
9. Show the post text and email address of all posts and the User who made them within the last 24 hours.
```sql
SELECT p.content, u.email
FROM posts p
JOIN users u 
ON p.sender_id = u.id
WHERE ROUND((JULIANDAY('now') - JULIANDAY(p.time_posted)) * 24) <= 24
ORDER BY p.time_posted DESC;
```
10. Show the email addresses of all Users who have not posted anything yet.
```sql
SELECT u.email
FROM users u
LEFT JOIN posts p 
ON u.id = p.sender_id
WHERE u.id NOT IN (SELECT sender_id FROM posts);
```