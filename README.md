# Netflix-EDA-using-SQL
I used SQL Server to manipulate and analyze my Netflix viewing history data. I exported my data from my Netflix account and had an Excel file that contained two columns, one for Title and the second for viewing Date and 151 rows. I added a primary key to the dataset and imported the data into a database I created in SQL Server. From there I began to build out the dataset to include columns for Season, Episode Name, Category (movie/show), and Genre. After I cleaned and prepared the data, I explored my viewing history using different aggregate functions. I learned that my favorite genre is sci-f, my least favorite genre is crime, my most watched show is Space Force, my least watched show is Designated Survivor, I've watched 15 movies this year (2023), and since 11/2022 I've watched an average of 2 show/movies a day.


-- Getting an overall view of my viewing history
SELECT * FROM netflixviewinghistory


-- Checking data types
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'netflixviewinghistory'


-- Changing data type for date column from datetime to date
ALTER TABLE netflixviewinghistory
ALTER COLUMN Date DATE


--Updating the table with additional columns for season and episode name
ALTER TABLE netflixviewinghistory
ADD "Season" VARCHAR(255);

ALTER TABLE netflixviewinghistory
ADD "EpisodeName"  VARCHAR(255);



-- Splitting title into separate columns for season and episode name

UPDATE netflixviewinghistory
SET Title =
    CASE 
        WHEN CHARINDEX(':', title) > 0 then SUBSTRING(title, 1, CHARINDEX(':', title) - 1) 
        ELSE title 
    END, 
Season = 
    CASE 
        WHEN CHARINDEX(':', title, CHARINDEX(':', title) + 1) > 0 
            THEN SUBSTRING(title, CHARINDEX(':', title) + 1, CHARINDEX(':', title, CHARINDEX(':', title) + 1) - CHARINDEX(':', title) - 1) 
        ELSE '' 
    END, 
 EpisodeName =   
    CASE 
        WHEN CHARINDEX(':', title, CHARINDEX(':', title) + 1) > 0 
            THEN SUBSTRING(title, CHARINDEX(':', title, CHARINDEX(':', title) + 1) + 1, LEN(title)) 
        WHEN CHARINDEX(':', title) > 0 
            THEN SUBSTRING(title, CHARINDEX(':', title) + 1, LEN(title)) 
        ELSE '' 
    END;


-- Removing whitspaces from columns
UPDATE netflixviewinghistory
SET Title = TRIM(Title),
    Season = TRIM(Season),
	EpisodeName = TRIM(EpisodeName);



-- Updating values in episode name column to remove foreign characters
SELECT DISTINCT EpisodeName FROM netflixviewinghistory;

UPDATE netflixviewinghistory
SET EpisodeName = CASE WHEN EpisodeName = 'ITâ€™S GOOD TO BE BACK ON THE MOON' THEN 'IT''S GOOD TO BE BACK ON THE MOON'
                       WHEN EpisodeName = 'PiÃ±a Colada' THEN 'Pina Colada'
					   ELSE EpisodeName
					   END ;


-- Categorizing into show or movie
ALTER TABLE netflixviewinghistory
ADD "Category"  VARCHAR(255);

UPDATE netflixviewinghistory
SET
Category = 
    CASE 
	    WHEN Season like '%Season%' THEN 'Show'
	    WHEN Season like '%Part%' THEN 'Show'
        WHEN EpisodeName like '%Epsiode%' THEN 'Show'
	    WHEN Title in ('That ''90s Show',
	                'Living with Yourself',
					'exception',
					'PIECES OF HER',
					'Away',
					'1899',
					'I AM A STALKER',
					'Oats Studios') THEN 'Show'
	ELSE 'Movie'
		END;


-- Updating shows with null values in Season column to reflect 'Season 1'
UPDATE netflixviewinghistory
SET Season = 
       CASE 
	       WHEN Season = '' and Category = 'Show' THEN 'Season 1'
       ELSE Season
          END;


-- Updating the genre for all shows/movies
ALTER TABLE netflixviewinghistory
ADD "Genre"  VARCHAR(255);

UPDATE netflixviewinghistory
SET	Genre =
CASE WHEN Title = 'You' THEN 'Psychological Thriller'
     WHEN Title = '1899' THEN 'Mystery'
	 WHEN Title in ('21 Jump Street','Friends from College','Glass Onion','Living with Yourself','Space Force','That ''90s Show','The Recruit','This Is the End','Wednesday','You People') THEN 'Comedy'
	 WHEN Title in ('Another Life','Awake','Away','Code 8','Don''t Look Up','Elysium','Exception','I Am Mother','IO','JUNG_E','Oats Studios','Outside the Wire','Oxygen','Stowaway','The 100',
'The Cloverfield Paradox','The Colony','The Midnight Sky','The Mist','The OA','Travelers') THEN 'Sci-fi'
     WHEN Title in ('Bird Box','Bullet Head','PIECES OF HER','Prisoners','Shimmer Lake','Spiderhead') THEN 'Thriller'
	 WHEN Title in ('The Hurt Locker','Black Crab','War') THEN 'War'
	 WHEN Title = 'Blow' THEN 'Crime'
	 WHEN Title in ('Bullet Train','Extraction','How It Ends','Mile 22','No Escape','Resident Evil','Southpaw','Spectral','TAU','The Gray Man','The Guilty','Triple Frontier') THEN 'Action'
	 WHEN Title in ('Call Me by Your Name','Designated Survivor','Ozark','The King') THEN 'Drama'
     WHEN Title = 'Day Shift' THEN 'Supernatual'
     WHEN Title in ('I AM A STALKER','Our Universe','Stutz','The Hatchet Wielding Hitchhiker','The Volcano') THEN 'Documentary'
     WHEN Title in ('Slumberland','The Sandman') THEN 'Fantasy'
     WHEN Title = 'The Walking Dead' THEN 'Horror'
END; 



-- Most watched genre for shows and movies (Sci-Fi)
WITH GenreMost AS (
  SELECT Genre, COUNT(Genre) AS GenreTotal,ROW_NUMBER() OVER (ORDER BY COUNT(Genre) DESC) AS rn
  FROM netflixviewinghistory
  GROUP BY Genre
)
SELECT Genre, GenreTotal
FROM GenreMost
WHERE rn = 1


-- Most watched tv show (Space Force)
 WITH ShowMost AS (
  SELECT Title, Category,COUNT(Title) AS ShowTotal
  FROM netflixviewinghistory
  GROUP BY Title, Category
)
SELECT Title, Category, MAX(ShowTotal) AS TopShow
FROM ShowMost
WHERE ShowTotal = (SELECT MAX(ShowTotal) FROM ShowMost)
and Category = 'Show'
GROUP BY Title, Category;



-- Least watched genre for shows and movies (Crime)
WITH GenreLeast AS (
  SELECT Genre, COUNT(genre) AS GenreTotal,ROW_NUMBER() OVER (ORDER BY COUNT(Genre)) AS rn
  FROM netflixviewinghistory
  GROUP BY Genre
)
SELECT Genre, GenreTotal
FROM genreleast
WHERE rn = 1


-- Least watched tv show (Designated Survivor)
 WITH ShowLeast AS (
  SELECT Title, Category,COUNT(Title) AS ShowTotal
  FROM netflixviewinghistory
  GROUP BY Title, Category
)
SELECT Title, Category, MIN(Showtotal) AS TopShow
FROM ShowLeast
WHERE Showtotal = (SELECT MIN(Showtotal) FROM ShowLeast)
AND Category = 'Show'
GROUP BY Title, Category;



-- How many movies I've watched in 2023 (15)
SELECT Category,COUNT(Title) AS MovieTotal
FROM netflixviewinghistory
WHERE Category = 'Movie'
AND YEAR(Date) = '2023'
GROUP BY Category


-- Average number of shows and movies I watch a day (2)
WITH AvgView AS (
  SELECT Date AS Viewing_Date, COUNT(Title) as TitleTotal
  FROM netflixviewinghistory
  GROUP BY Date
  )
SELECT AVG(TitleTotal) AS Average
FROM AvgView;
