# Hamza Ahmed Abdi - 672522

Dataset Source: https://www.kaggle.com/datasets/datasnaek/chess/data

Rows: 20,058
Columns: 16

Goal: To analyze player engagement, game pacing, and opening popularity to improve the online chess platform's recommendation engine. This project focuses on advanced data preparation using Power Query and professional dashboard development in Power BI.

## Question 1: Advanced Power Query Data Preparation

### A. Basic Data Cleaning

1. I renamed the columns for :

   * white\_id → white\_player\_id
   * black\_id → black\_player\_id
   * increment\_code → time\_control
   * opening\_eco → opening\_code
   * Renamed created\_at to game\_start\_time.
   * Renamed last\_move\_at to Game End Time.
2. Since 'created\_at' and 'last\_move\_at' are Unix timestamps; I have to make 2 more custom columns to change them into date type using this formula:

\#datetime(1970, 1, 1, 0, 0, 0) + #duration(0, 0, 0, \[created\_at] / 1000) and #datetime(1970, 1, 1, 0, 0, 0) + #duration(0, 0, 0, \[last\_move\_at] / 1000)

then i deleted the two older columns

3. I selected the 'id' column and then removed the duplicates
4. From Home tab Remove Rows > Remove Blank Rows.
5. Trim and Clean Text Columns:

   * select 'opening\_name', 'winner' and 'victory\_status'.
   * go to Transform > Format > Trim.
   * go to Transform > Format > Clean.
6. All columns are consistent
7. Removed the unnecessary column 'opening\_code'since we already have 'opening\_name'

### B. Intermediate Transformations

1. I split the 'time\_control' column making ie 5+3(5 min + 2 sec increments) to minutes and increments by the delimiter '+' then renamed the new split columns
2. Merged(after reordering) the columns 'winner' and 'victory\_status' and named the merged column 'winner'
3. Extracted Year, Month, Quarter, and Day from 'game\_start\_time' Column
4. Created a custom column for the category i.e.;

   * If Base Time < 3 then "Bullet"
   * If Base Time < 10 then "Blitz"
   * If Base Time < 60 then "Rapid"
   * Else "Classical"
5. Created a conditional column 'player\_tier' where:

   * If white\_rating >= 2000 then "Master"
   * Else If white\_rating >= 1500 then "Intermediate"
   * Else "Beginner"
6. Filter time\_control\_min to be greater than 5 to remove ultra-bullet noise.
7. Sorted by 'game\_start\_time' Ascending to see the chronological flow.
8. Added an index column from 1

### C. Advanced Power Query Tasks

1. Created a date table by adding a black query and typing this code:

let

&#x20;   StartDate = #date(2017, 1, 1),

&#x20;   EndDate = #date(2026, 12, 31),

&#x20;   Source = List.Dates(StartDate, Duration.Days(EndDate - StartDate) + 1, #duration(1, 0, 0, 0)),

&#x20;   #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),

&#x20;   #"Renamed Columns" = Table.RenameColumns(#"Converted to Table",{{"Column1", "Date"}}),

&#x20;   #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns",{{"Date", type date}}),

&#x20;   #"Added Year" = Table.AddColumn(#"Changed Type", "Year", each Date.Year(\[Date]), Int64.Type),

&#x20;   #"Added Month" = Table.AddColumn(#"Added Year", "Month", each Date.MonthName(\[Date]), type text),

&#x20;   #"Added Quarter" = Table.AddColumn(#"Added Month", "Quarter", each "Q" \& Number.ToText(Date.QuarterOfYear(\[Date])), type text)

in

&#x20;   #"Added Quarter"



2. Used Group by with 2 aggregators to summarize player performance by their opening choice :

   * total\_games, Operation: Count Rows
   * avg\_white\_rating, Operation: Average
   * avg\_opening\_ply(Average Number of moves in the opening phase),Operation:Average
3. Created a business "match\_balance" category based on another category 'rating\_dif' (which is the rating difference between players) in order to understand which games have evenly matched players.

   * if \[rating\_dif] > 200 then "Mismatched (White Higher)"
   * else if \[rating\_dif] < -200 then "Mismatched (Black Higher)"
   * else if \[rating\_dif] > 50 then "Slightly Unbalanced"
   * else if \[rating\_dif] < -50 then "Slightly Unbalanced"
   * else "Evenly Matched"
4. Extracted text from the opening names to get just the openings and not the variations
5. Created a summarizered query for rated games 
