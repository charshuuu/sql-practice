### sql基础知识测试

题目来源自sqlzoo，选取一些比较有代表性的题目进行作答并给出一些知识点注解。

#### 一、world表练习，主要涉及select&from&where

表名为world，详细表内容如下：

| name        | continent | area    | population | gdp          |
| :---------- | :-------- | :------ | :--------- | :----------- |
| Afghanistan | Asia      | 652230  | 25500100   | 20343000000  |
| Albania     | Europe    | 28748   | 2831741    | 12960000000  |
| Algeria     | Africa    | 2381741 | 37100000   | 188681000000 |
| Andorra     | Europe    | 468     | 78115      | 3712000000   |
| Angola      | Africa    | 1246700 | 20609294   | 100990000000 |
| ...         |           |         |            |              |

通过使用select&from&where语句来查询并得到我们想要的结果，由于题库自带文本编辑器没有换行自动排版功能，以下sql语句格式部分不规范。

1. 找出有至少200百萬(2億)人口的國家名稱，及人均國內生產總值。

   `select name, (gdp/population) from world`
   `where population>200000000`

   //此处也可以写成 (gdp/population) as xx from world，xx将会作为列名出现在结果中

2. 顯示法國，德國，意大利(France, Germany, Italy)的國家名稱和人口。

   `select name, population from world`
   `where name in ("France", "Germany", "Italy")`

   //当同列里要查询的项有多个时用in方法，如果是不包含则用not in

3. 顯示包含單詞“United”為名稱的國家。

   `select name from world`
   `where name like "%united%"`

   //包含某个词用like '%xx%'，如果是以xx开头则'%xx'，以xx结尾则'xx%'，不包含用not like

4. 美國、印度和中國(USA, China)是人口又大，同時面積又大的國家。排除這些國家。顯示以人口或面積為大國的國家，但不能同時兩者。顯示國家名稱，人口和面積。

   `select name, population, area from world`
   `where (area>3000000 or population>250000000) and name not in ("United States", "China")`

   //在审要求时要注意“和”与“或”这样的信息，and表示连接的两个条件必须都满足，or表示连接的两个条件只需要满足一个

5. 除以為1000000（6個零）是以百萬計。除以1000000000（9個零）是以十億計。使用 ROUND函數來顯示的數值到小數點後兩位。對於南美顯示以百萬計人口，以十億計2位小數GDP。

   `select name, round(population/1000000, 2), round(gdp/1000000000, 2) from world`
   `where continent="south america"`

   //在ROUND函数中，第二个参数表示要四舍五入到的小数位数。正数表示要保留的小数位数，而负数表示要四舍五入的整数位数（看下一题示例）。

6. 顯示國家有至少一個萬億元國內生產總值（萬億，也就是12個零）的人均國內生產總值。四捨五入這個值到最接近1000。顯示萬億元國家的人均國內生產總值，四捨五入到最近的$ 1000。

   `select name, round(gdp/population, -3) from world`
   `where gdp>1000000000000`

7. 显示以字母N开头的国家的名称，但用Australasia替代Oceania。

   `SELECT name,`
          `CASE WHEN continent='Oceania' THEN 'Australasia'`
               `ELSE continent END`
    `FROM world`
    `WHERE name LIKE 'N%'`

   //case语句用于在查询中根据条件执行不同的操作，一般形式有如

   `CASE`
     `WHEN condition1 THEN result1`
     `WHEN condition2 THEN result2`
     `...`
     `ELSE default_result`
   `END`

   每个条件都是一个布尔表达式，当某个条件为真时，对应的result将被返回。如果所有条件都不满足，将返回default_result。

8. Put the continents right...

   - Oceania becomes Australasia
   - Countries in Eurasia and Turkey go to **Europe/Asia**
   - Caribbean islands starting with 'B' go to **North America**, other Caribbean islands go to **South America**

   Show the name, the original continent and the new continent of all countries.

   `SELECT name, continent AS original_continent,`
     `CASE`
       `WHEN continent = 'Oceania' THEN 'Australasia'`
       `WHEN continent IN ('Eurasia', 'Turkey') THEN 'Europe/Asia'`
       `WHEN continent = 'Caribbean' AND name LIKE 'B%' THEN 'North America'`
       `WHEN continent = 'Caribbean' THEN 'South America'`
       `ELSE continent END AS new_continent`
   `FROM world`

   //case中每个语句的条件可以是多重条件。

9. 显示名称和首都，其中名称和首都具有相同数量的字符。

   `SELECT name, capital FROM world`
   `WHERE LENGTH(name)=LENGTH(capital)`

   //length函数的用法

10. 瑞典的首都是斯德哥尔摩。两个词都以字母'S'开头。显示名称和首都，其中每个词的第一个字母相匹配。不包括名称和首都相同的国家。 您可以使用LEFT函数来分离第一个字符。 您可以使用<>作为不等于运算符。

    `SELECT name, capital FROM world`
    `WHERE LEFT(name,1) = LEFT(capital,1) and name != capital`

    //left的用法，不等于用!=或者<>

#### 二、nobel表练习，主要涉及order by的应用

表名为nobel，详细表内容如下：

| yr   | subject    | winner                      |
| :--- | :--------- | :-------------------------- |
| 1960 | Chemistry  | Willard F. Libby            |
| 1960 | Literature | Saint-John Perse            |
| 1960 | Medicine   | Sir Frank Macfarlane Burnet |
| 1960 | Medicine   | Peter Madawar               |
| ...  |            |                             |

1. 列出爵士的獲獎者、年份、獎頁(爵士的名字以Sir開始)。先顯示最新獲獎者，然後同年再按名稱順序排列。

   `select winner, yr, subject from nobel`
   `where winner like 'sir%'`
   `order by yr desc, winner`

   //order by语句可以有多个排序标准，按想要的排序顺序依次放置，用desc可以倒序排列。

2. 表达式 subject IN ('Chemistry', 'Physics')可以被用作一个值，它将返回 0 或 1。展示1984年的获奖者和获奖主题，按照主题和获奖者姓名排序；但将化学（Chemistry）和物理（Physics）列在最后。

   `SELECT winner, subject FROM nobel`
   `WHERE yr=1984`
   `ORDER BY subject IN ('Physics','Chemistry'),subject,winner`

   //subject IN ('Physics', 'Chemistry')的结果将会被首要考虑，如果主题是物理学或化学得到1就会排在后面。

3. 查找尤金•奧尼爾EUGENE O'NEILL得獎的所有細節（注意名字中有单引号）

   `select * from nobel`
   `where winner = 'EUGENE O''NEILL'`

   //你不能把一個單引號直接的放在字符串中。但您可連續使用兩個單引號在字符串中當作一個單引號。

4. 

   

#### 三、world表练习，主要涉及select嵌套语句

表名为world，详细表内容如下：

| name        | continent | area    | population | gdp          |
| :---------- | :-------- | :------ | :--------- | :----------- |
| Afghanistan | Asia      | 652230  | 25500100   | 20343000000  |
| Albania     | Europe    | 28748   | 2831741    | 12960000000  |
| Algeria     | Africa    | 2381741 | 37100000   | 188681000000 |
| Andorra     | Europe    | 468     | 78115      | 3712000000   |
| Angola      | Africa    | 1246700 | 20609294   | 100990000000 |
| ...         |           |         |            |              |

1. 列出歐州每國家的人均GDP，當中人均GDP要高於英國'United Kingdom'的數值。

   `select name from world`
   `where gdp/population > (select gdp/population from world`
   `where name = 'United Kingdom')` 
   `and continent = 'europe'`

   //注意“欧洲”、“人均”这几个限制条件

2. Germany德國（人口8000萬），在Europe歐洲國家的人口最多。Austria奧地利（人口850萬）擁有德國總人口的11％。顯示歐洲的國家名稱name和每個國家的人口population。以德國的人口的百分比作人口顯示。

   `select name, concat(round((population/(select population from world where name = 'germany'))*100,0), '%')`
   `from world`
   `where continent = 'europe'`

   //使用round函数将第二个参数设置为0删除小数，再使用concat函数连接上%符号

3. 哪些國家的GDP比Europe歐洲的全部國家都要高呢?

   `select name from world`
   `where gdp > all(select gdp from world where continent = 'europe')`

   //all函数可以保证gdp要大于筛选出来的欧洲所有国家的gdp

4. 在每一個州中找出最大面積的國家，列出洲份continent, 國家名字name及面積area。

   `SELECT continent, name, area FROM world x`
   `WHERE area >= ALL`
   `(SELECT area FROM world y`
   `WHERE y.continent=x.continent`
   `AND area>0)`

   //因为有些记录会为null所有增加一个判断area>0，同时注意其中给嵌套查询的两张表分别给了别名确保在同一个大洲中进行比较。

5. 列出洲份名稱，和每個洲份中國家名字按子母順序是排首位的國家名。(即每洲只有列一國)

   `select continent, name from world x`
   `where name <=`
   `all(select name from world y`
   `where x.continent = y.continent`
   `order by name)`

   //在后面的知识里可以用窗口函数来帮助解答，在这里我们可以用学过的all来帮助解答，小于等于得到的所有结果就只会返回第一个结果（即字母排首位的国家名），还要注意的是select语句不能放在比较的前面

6. 有些國家的人口是同洲份的所有其他國的3倍或以上。列出 國家名字name 和 洲份 continent。

   `select name, continent from world x`
   `where population >` 
   `all(select population*3 from world y where x.continent = y.continent and x.name <> y.name)`

   //要注意加上判断比较的两个国家名字不相同的这个限定条件

7. 

#### 四、world表练习，主要涉及count、sum等函数和group by、having的知识

表名为world，详细表内容如下：

| name        | continent | area    | population | gdp          |
| :---------- | :-------- | :------ | :--------- | :----------- |
| Afghanistan | Asia      | 652230  | 25500100   | 20343000000  |
| Albania     | Europe    | 28748   | 2831741    | 12960000000  |
| Algeria     | Africa    | 2381741 | 37100000   | 188681000000 |
| Andorra     | Europe    | 468     | 78115      | 3712000000   |
| Angola      | Africa    | 1246700 | 20609294   | 100990000000 |
| ...         |           |         |            |              |

1. 對於每一個洲份，顯示洲份和國家的數量。

   `select continent, count(name) from world`
   `group by continent`

   //这边的group by 能将表中数据按照continent列的值来进行分组，这样我们就能对每个不同的continent值进行改组中的计数。

2. 列出有至少100百萬(1億)(100,000,000)人口的洲份。

   `select continent from world`
   `group by continent`
   `having sum(population) > 100000000`

   //在分组后，HAVING子句用于筛选分组，只有那些满足条件的分组才会包含在结果中。在这里它保留了那些总人口超过1亿的大洲。

3. 

#### 五、基于多个表格的JOIN练习

表格有如下三个：

game（赛事）

| id(編號) | mdate(日期)  | stadium(場館)             | team1(隊伍1) | team2(隊伍2) |
| :------- | :----------- | :------------------------ | :----------- | :----------- |
| 1001     | 8 June 2012  | National Stadium, Warsaw  | POL          | GRE          |
| 1002     | 8 June 2012  | Stadion Miejski (Wroclaw) | RUS          | CZE          |
| 1003     | 12 June 2012 | Stadion Miejski (Wroclaw) | GRE          | CZE          |
| 1004     | 12 June 2012 | National Stadium, Warsaw  | POL          | RUS          |
| ...      |              |                           |              |              |

goal（进球）

| matchid(賽事編號) | teamid(隊伍編號) | player(入球球員)     | gtime(入球時間) |
| :---------------- | :--------------- | :------------------- | :-------------- |
| 1001              | POL              | Robert Lewandowski   | 17              |
| 1001              | GRE              | Dimitris Salpingidis | 51              |
| 1002              | RUS              | Alan Dzagoev         | 15              |
| 1001              | RUS              | Roman Pavlyuchenko   | 82              |
| ...               |                  |                      |                 |

eteam（欧洲队伍）

| id(編號) | teamname(隊名) | coach(教練)      |
| :------- | :------------- | :--------------- |
| POL      | Poland         | Franciszek Smuda |
| RUS      | Russia         | Dick Advocaat    |
| CZE      | Czech Republic | Michal Bilek     |
| GRE      | Greece         | Fernando Santos  |
| ...      |                |                  |

1. 列出球員名字叫Mario (player LIKE 'Mario%)有入球的 隊伍1 team1, 隊伍2 team2 和 球員名 player

   `SELECT team1, team2, player`
   `FROM game JOIN goal ON (id=matchid)`
   `where player LIKE 'Mario%'`

   //from xx1 join xx2表示将xx2的数据合并到xx1，on则是配对的条件。 

2. 列出'Fernando Santos'作為隊伍1 team1 的教練的賽事日期，和隊伍名。

   `select mdate, teamname from game`
   `join eteam on (team1=eteam.id)`
   `where coach = 'Fernando Santos'`

   //注意因为连接的两个表格中都有id信息所以要用eteam.id来区分

3. 列出全部賽事，射入德國龍門的球員名字。

   `SELECT distinct(player)`
   `FROM game JOIN goal ON matchid = id` 
   `WHERE (team1='GER' or team2='GER') and teamid <> 'ger'`

   //审题后可以知道筛选条件为参与的任意一方为德国队且射门的球员不是德国队的

4. 每一場德國'GER'有參與的賽事中，列出賽事編號 matchid, 日期date 和德國的入球數字。

   `SELECT matchid,mdate, count(teamid)`
   `FROM game JOIN goal ON matchid = id` 
   `WHERE (team1 = 'ger' OR team2 = 'ger') and teamid = 'ger'`
   `group by matchid`

   //要求每一场有德国参与的赛事中德国进球数，就得先按照赛事号来分组然后筛选计数

5. List every match with the goals scored by each team as shown. This will use [CASE WHEN] which has not been explained in any previous exercises.If it was a team1 goal then a 1 appears in score1, otherwise there is a 0. You could SUM this column to get a count of the goals scored by team1. Sort your result by mdate, matchid, team1 and team2.

   | mdate        | team1 | score1 | team2 | score2 |
   | ------------ | ----- | ------ | ----- | ------ |
   | 1 July 2012  | ESP   | 4      | ITA   | 0      |
   | 10 June 2012 | ESP   | 1      | ITA   | 1      |
   | 10 June 2012 | IRL   | 1      | CRO   | 3      |
   | ...          |       |        |       |        |

   `SELECT mdate, team1,`
   `sum(CASE WHEN teamid=team1 THEN 1 ELSE 0 END) score1, team2,`
   `sum(CASE WHEN teamid=team2 THEN 1 ELSE 0 END)  score2`
   `FROM game left JOIN goal ON matchid = id`
   `group by mdate, matchid`

   //goal表是进球记录，要注意的是为了防止那些双方都没有进球的比赛不显示出来，得采用left join，这样如果数据为null就会计数为0。

   

