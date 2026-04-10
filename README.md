# SQL_RealMadrid_Performance
A SQL-centered database project designed to track the performance and goal dynamics of Real Madrid players. It uses a relational structure to analyze match statistics, player efficiency, and goal details.
```sql
Create database RealMadrid_Performance_DB
```
```sql
/* Real Madrid Performance Analytics Database 
   Season-based Relational Model for Player Tracking
*/

USE RealMadrid_Performance_DB;
GO

-- 1. Players Table: Stores static personal information of the athletes
CREATE TABLE Players (
    player_id INT PRIMARY KEY IDENTITY(1,1),
    full_name NVARCHAR(100) NOT NULL,
    age TINYINT, -- Using TINYINT for memory optimization (1 byte)
    position VARCHAR(3), -- GK, DF, MF, FW (Standard football abbreviations)
    height_cm TINYINT,
    weight_kg TINYINT,
    nationality VARCHAR(30),
    preferred_foot CHAR(1) CHECK (preferred_foot IN ('R', 'L', 'B')) -- R: Right, L: Left, B: Both
);

-- 2. Matches Table: Keeps track of all fixtures in a season
CREATE TABLE Matches (
    match_id INT PRIMARY KEY IDENTITY(1,1),
    opponent VARCHAR(50) NOT NULL,
    match_date DATE NOT NULL,
    competition VARCHAR(10), -- e.g., 'LL' (La Liga), 'UCL' (Champions League)
    venue CHAR(1) CHECK (venue IN ('H', 'A')) -- H: Home, A: Away
);

-- 3. Match_Performances Table: Intersection table between Players and Matches
-- This table stores specific performance metrics for each player in every match
CREATE TABLE Match_Performances (
    performance_id INT PRIMARY KEY IDENTITY(1,1),
    player_id INT NOT NULL,
    match_id INT NOT NULL,
    minutes_played TINYINT,
    distance_km DECIMAL(4,2), -- High precision for physical tracking data
    pass_accuracy_pct TINYINT,
    sprints_count TINYINT,
    rating DECIMAL(3,1), -- Performance rating (e.g., 8.5)
    is_motm BIT DEFAULT 0, -- Boolean: 1 if 'Man of the Match', 0 otherwise

    -- Foreign Key Constraints with Cascade: 
    -- If a player/match is deleted, all related performance records are removed automatically.
    CONSTRAINT FK_Perf_Player FOREIGN KEY (player_id) 
        REFERENCES Players(player_id) ON DELETE CASCADE,
    CONSTRAINT FK_Perf_Match FOREIGN KEY (match_id) 
        REFERENCES Matches(match_id) ON DELETE CASCADE
);

-- 4. Goal_Events Table: Granular data for every goal scored
-- Linked via performance_id to identify the specific match/player context
CREATE TABLE Goal_Events (
    goal_id INT PRIMARY KEY IDENTITY(1,1),
    performance_id INT NOT NULL,
    goal_type VARCHAR(15), -- e.g., 'Header', 'Penalty', 'Long Shot'
    assist_provider_id INT NULL, -- FK to Players table (Can be NULL for solo goals)
    goal_minute TINYINT,

    CONSTRAINT FK_Goal_Performance FOREIGN KEY (performance_id) 
        REFERENCES Match_Performances(performance_id) ON DELETE CASCADE,
    
    -- Using NO ACTION here to prevent multiple cascade paths (SQL Server safety rule)
    CONSTRAINT FK_Goal_Assistant FOREIGN KEY (assist_provider_id) 
        REFERENCES Players(player_id) ON DELETE NO ACTION 
);
GO
```
