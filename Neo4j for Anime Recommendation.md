
//Neo4j for Anime Recommendation: Exploratory Analysis & System Building
//Here's how we can use Neo4j for anime & rating dataset



// 1. Data Import & Modeling:

/*
Nodes:
   User: Represents each user (user_id)
   Anime: Represents each anime (anime_id, name, genre, type, episodes, rating, members)
   
Relationships:
   RATED: Connects User to Anime with the rating property (rating).
*/

// Import 'anime.csv' as Anime nodes
// ----------------------------------------------------------------
// cypher 
LOAD CSV WITH HEADERS FROM 'file:///C:/Users/123/recommender/anime.csv' AS row
CREATE (a:Anime { 
    anime_id: toInteger(row.anime_id), 
    name: row.name, 
    genre: split(row.genre, ","), 
    type: row.type, 
    episodes: toInteger(row.episodes), 
    rating: toFloat(row.rating),
    members: toInteger(row.members)
})
// ----------------------------------------------------------------



// Import 'rating.csv' and create relationships
// ----------------------------------------------------------------
// cypher 
LOAD CSV WITH HEADERS FROM 'file:///C:/Users/123/recommender/rating.csv' AS row
MATCH (u:User {user_id: toInteger(row.user_id)}), (a:Anime {anime_id: toInteger(row.anime_id)})
CREATE (u)-[r:RATED {rating: toInteger(row.rating)}]->(a)
// ----------------------------------------------------------------




// 2. Exploratory Data Analysis:

// 2.1. Popular Genres: Find the most popular genres among all users:
// cypher 
// ----------------------------------------------------------------
MATCH (a:Anime)
UNWIND a.genre AS genre
RETURN genre, count(*) AS count 
ORDER BY count DESC
// ----------------------------------------------------------------


// 2.2. Average Rating by Genre: Calculate average rating for each genre:
// cypher 
// ----------------------------------------------------------------
MATCH (:User)-[r:RATED]->(a:Anime)
UNWIND a.genre AS genre
RETURN genre, avg(r.rating) AS average_rating 
ORDER BY average_rating DESC
// ----------------------------------------------------------------


// 2.3. User Preference Similarity: Find users with similar anime preferences:
// cypher 
// ----------------------------------------------------------------
MATCH (u1:User)-[r1:RATED]->(a:Anime)<-[r2:RATED]-(u2:User)
WHERE u1.user_id < u2.user_id  // Avoid duplicates
WITH u1, u2, count(a) AS common_anime, collect(abs(r1.rating - r2.rating)) AS rating_diffs
WHERE common_anime >= 5  // Set a threshold for common anime
RETURN u1.user_id, u2.user_id, common_anime, avg(rating_diffs) AS avg_rating_difference
ORDER BY avg_rating_difference ASC
// ----------------------------------------------------------------




// 3. Recommendation System:

// 3.1. Collaborative Filtering (User-Based):
// cypher 
// ----------------------------------------------------------------
MATCH (u1:User {user_id: 123})-[r1:RATED]->(a:Anime)<-[r2:RATED]-(u2:User)
WHERE u2 <> u1 AND r2.rating >= 8 // Find similar users who liked the same anime
WITH u2, count(*) AS commonAnimeCount, avg(r2.rating) AS avgRating
ORDER BY commonAnimeCount DESC, avgRating DESC
LIMIT 10
// ----------------------------------------------------------------



// 3.2. Recommend anime liked by these similar users that the target user hasn't seen
// cypher 
// ----------------------------------------------------------------
MATCH (u2)-[r:RATED]->(rec:Anime)
WHERE NOT (u1)-[:RATED]->(rec) AND r.rating >= 8
RETURN rec.name, rec.genre, avg(r.rating) AS recommendationScore
ORDER BY recommendationScore DESC
LIMIT 10 
// ----------------------------------------------------------------



// 3.3. Content-Based Filtering:
// cypher 
// ----------------------------------------------------------------
MATCH (u:User {user_id:123})-[r:RATED]->(a:Anime) 
WHERE r.rating >= 8
WITH u, collect(a.genre) AS user_genres 

// Find anime with similar genres
UNWIND user_genres AS user_genre
MATCH (rec:Anime) WHERE rec.genre CONTAINS user_genre AND NOT (u)-[:RATED]->(rec)
RETURN rec.name, rec.genre, rec.rating AS recommendationScore
ORDER BY recommendationScore DESC 
LIMIT 10
//




// 4. Further Analysis & Improvement:

/*
This example demonstrates how Neo4j can be used to build a simple anime recommendation system. The graph database enables efficient exploration of relationships and patterns, leading to insightful data analysis and targeted recommendations.
*/
