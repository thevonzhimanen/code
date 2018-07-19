THIS IS A COLLECTION OF CODES FOR FUTURE USE
-Python
-SQL
-Processing
-Arduino 
-HTML
-CSS
-Javascript


/*

sql and sqlite in 5 min
-   " is not '
-  " for fields
-   ' for text
-   there's no indentations
-   not important capcase and lowercase
-   4 types in sqlite : text, real, integer and blob
-   rowid is a ghost row


spatialite 
http://www.gaia-gis.it/gaia-sins/spatialite-sql-4.2.0.html

sqlite
https://www.tutorialspoint.com/sqlite/
*/


--select

select * from dianping

  

SELECt "price", 'price' from dianping

--types (text, real, int, blob)

select typeof(name) from dianping

select typeof(price) as price_type , price from dianping

select typeof(cast(price as real)) as price_type, price from dianping

--limit

select price from dianping limit 100


-- order by

select price from dianping order by cast(price as real)  limit 100 

select price from dianping order by cast(price as real) asc  limit 100  

select price from dianping order by cast(price as real) desc limit 100 

-- where

select name, price from dianping where price>20

select name, price from dianping where cast(price as real)>20

select name, price from dianping where cast(price as real)>2000 order by cast(price as real) desc

-- update

update dianping
set price = cast(price as real)

-- distinct

select distinct type from dianping 

-- aggregate functions

select count()  from dianping 

select max(price) from dianping

select max(price), min(price), avg(price) from dianping

select max(price) as max_price, min(price) as min_price, avg(price) as avg_price from dianping

-- group by

select max(price) as max_price, min(price) as min_price, avg(price) as avg_price from dianping
group by type
order by avg(price)

select max(price) as max_price, min(price) as min_price, avg(price) as avg_price from dianping
group by type
order by max_price



--spatialite

//to select a database and turn it into a spatialite database;
select InitSpatialMetaData(1)

select AddGeometryColumn("dianping", 'geometry', 4326, 'POINT')

update dianping set geometry = makepoint(lon, lat, 4326)

select createspatialindex('dianping', 'geometry')

SELECT AddGeometryColumn('dianping', 'geom',  4326, 'POINT', 'XY')


https://www.arcgis.com/home/item.html?id=105f92bd1fe54d428bea35eade65691b


///// SPATIAL OPERATIONS
--http://www.gaia-gis.it/gaia-sins/spatialite-sql-4.3.0.html

--import table districts
--import table osm_amenities
-- attach database osm
--attach table dianping

SELECT AddGeometryColumn('dianping', 'geom',  4326, 'POINT', 'XY')

--01. intersect point with polygon

---count number of restaurants per district

create rest_x_district as 
select t1.Geometry, count(t2.geom)
from sh_districts as t1, a.dianping as t1
where st_intersects(t1.geometry, t2.geom)

--recover geometry columns

RecoverGeometryColumn( table String , column String , srid Integer , geom_type String , dimension Integer ) : Integer 

select RecoverGeometryColumn('rest_x_district', 'GEOMETRY',4326, 'MULTIPOLYGON', 'XY'); -- 1 MEANS THAT WORKED
----

--Create spatial index
--
CreateSpatialIndex( table String , column String ) : Integer

SELECT CreateSpatialIndex( 'rest_x_district', 'Geometry')  -- 1 MEANS THAT WORKED

--02. count number of fast food rest at distance

create table rest_x_school as 
select t1.*, count(t2.Geometry) as numrest
from sh_amenities as t1, b.osm_food as t2
where t1.type like 'school' and t2.amenity like 'fast_food' 
and st_intersects(st_buffer(t1.geometry,200), t2.Geometry) --change to decimal 0.01
group by t1.Geometry

select RecoverGeometryColumn('rest_x_school', 'Geometry',4326, 'POINT', 2); -- 1 MEANS THAT WORKED

SELECT CreateSpatialIndex( 'rest_x_school', 'Geometry')  -- 1 MEANS THAT WORKED


--03. distance to restaurants

select 
t1.*, 
st_distance(t1.geometry, t2.Geometry) as distance
from sh_amenities as t1, b.osm_food as t2
where t1.type like 'school' and t2.amenity like 'fast_food'
and distance <0.1

---advanced version

select
t3.*, avg(t3.distance)
from 
(select 
t1.*, 
st_distance(t1.geometry, t2.Geometry) as distance
from sh_amenities as t1, b.osm_food as t2
where t1.type like 'school' and t2.amenity like 'fast_food'
and distance <0.1
) as t3 
group by t3.osm_id

--Selecting columns from an attached CSV, and adding more columns

select Chinese, Area__km²__3_, Population__2015_census__4_, Density__km²_
from Shanghai_districts
union all
select Name,'Area', 'Population', 'Density'
from sh_district

--Adding columns

ALTER TABLE "sh_district"
ADD COLUMN 'area'

update sh_district set area= cast(areakm2 as real)

--Matching / "merging" data

create table #tablename# as 
select t2.Name, t1.Name, cast(t2.Area__km²__3_ as real) as areakm2, cast(t2.Population__2015_census__4_ as int) as population, t1.Shape_Leng, t1.Shape_Area, t1.geometry
from sh_district as t1, Shanghai_districts as t2
where t1.Name like t2.Chinese

--cast text as real

select cast (Areakm2 as real), cast (Population2015 as real) from sh_district2

--Creating a buffer along every street, counting number of points within buffer

c --change to decimal 0.01
group by t1.Geometry


create table fast_road2 as 
select t1.*, count(t2.Geometry) as numfastfood
from shanghai_china_osm_roads as t1, osm_food as t2
where t1.type like 'residential' and t2.amenity like 'fast_food' 
and st_intersects(st_buffer(st_transform(t1.geometry,3857),500), st_transform(t2.Geometry, 3857)) 
group by t1.Geometry

--updating CRS/SRID/EPSG

update #TABLE# set Geometry = st_transform(Geometry, 4490)

--add buffer geometry first

update osm_food set cafe_x_rest = 
Buffer(#Geometry#, 100)
where amenity like 'cafe'
--rememebr the buffesr are set to degrees by default!! to transform into meters:
st_buffer(st_transform(#Geometry#,3857),#meters#)

--create table with buffers 
--(first create separate tables with only the geometries you want)
create table cafe as
select * from osm_food where "amenity" like "cafe"

create table osm_rest as
select * from osm_food where "amenity" like "restaurant"

--then find their intersectitons
create table cafe_x_rest as
select t1.*, count(t2.geometry) as numrest
from osm_cafe as t1, osm_rest as t2
where st_intersects(t1.cafe_buffer, t2.geometry)
--don't forget to recover geometry columns and create spatial index!

create table cafe_x_rest as
select t1.*, count(t2.geometry) as numrest
from osm_cafe as t1, osm_rest as t2
where st_intersects(st_buffer(t1.geometry, 0.01), t2.geometry)
group by t1.geometry

--table of 'workplaces'

create table osm_workplaces as
select t1.*
from shanghai_china_osm_buildings as t1, shanghai_china_osm_amenities as t2
where t1.type like '%commercial%' and t1.type like '%industrial%' and t2.type like 'university' and t2.type like 'school'

--use workplaces table to calculate distances to cafes

create table cafe_x_work as
select t1.*, st_distance(t1.Geometry, st_centroid(t2.Geometry)) as distance
from osm_cafe as t1, osm_workplaces1 as t2
group by t1.Geometry

--2018.02.19 CLASS (from Alba)
--check it out:
---- atnight
---- barcelona dynamics
---- ambinnova.300000kms.net

-- regionalization == cluster of clusters
-- each color is defined by a set of properties (variables)


---------------------------------------------------------------------------
--- doing a cluster with spatialite (it is a fast operation). it does squares (no hexagons)
1. open osm with qgis
2. save as spatialite, crs 4326, srid 4326
3. query
	create table amenities as 
	select count() as c --this is an aggregate funct
	st_snaptogrid(st_transform("GEOMETRY", 3857),400) as geom --grid of 400m --3857 is pseudomercator. 4326 is degree
	from points
	where amenity is not null
	group by geom --this is the new column where geometry is converted

-- important: first explore what is the size of the grid that better explains our area
	
4. -- rclick on new table: recover geometry colum, srid 3857, dims xy, points
5. -- same query for the shops
6. -- on qgis you can copy paste styles ! (rc on layer, copy style, go to otyer layer, style paste style)

---------------------------------------------------------------------------
-- union == merges only different values
-- union all == merges all the values
-- join == adds more columns to a table taking those from other table, where they match in the id (e.g.)

---------------------------------------------------------------------------
---- how to merge 2 columns from differents tables
1. query
	create table commerce as
	select g.geom, a.c as ca, s.c as cs
	from ( --subquery
	select geom from amenities
	union
	select geom from shops
	) as g -- name of temporary query (ALWAYS put it)
	left join
	amenities as a on a.geom = g.geom
	left join
	shops as s on s.geom = g.geom 
	
-- now step by step
	select geom from amenities
	union all -- union and union all is different ! 
	select geom from shops
	
	table 1= all possible geometries (g.geom)
	table 2= ca of amenities (a.c)
	table 3= cs of shops (s.c)
	
-- to turn NULL into 0
	update commerce
	set cs = 0 where cs is null
	update commerce
	ca = 0 where ca is null

----- now in qgis
1. -- import commerce table
2. -- size according to proportion shops / commerce
2. -- coalesce("ca"/"cs", 0) since sometimes it could be 0/? or ?/0 and it would give us an error 

---------------------------------------------------------------------------
-----
1. -- query -- this takes long
	select
	sum(case when amenity like 'restaurant' then 1 else 0 end) as amenities,
	sum(case when shop like 'convenience' then 1 else 0 end) as convenience,	
	sum(case when amenity like 'marketplace' then 1, else 0, end) as marketplace
	sum(case when amenity in('marketplace', 'toilets', 'pharmacy') then 1 else 0, end) as mixeduses,
	st_snaptogrid(st_transform(geometry,3857),400) as geom
	from points
	group by geom
	
2. -- same query faster: change "like" by "is"
	select
	sum(case when amenity like 'restaurant' then 1 else 0 end) as amenities,
	sum(case when shop like 'convenience' then 1 else 0 end) as convenience,	
	sum(case when amenity like 'marketplace' then 1, else 0, end) as marketplace
	sum(case when amenity in('marketplace', 'toilets', 'pharmacy') then 1 else 0, end) as mixeduses,
	st_snaptogrid(st_transform(geometry,3857),400) as geom
	from points
	group by geom
	
3. -- same query with words % (now we need LIKE and cannot put IS)
	select
	sum(case when amenity like '%restaurant%' then 1 else 0 end) as amenities,
	sum(case when shop like '%convenience%' then 1 else 0 end) as convenience,	
	sum(case when amenity is 'marketplace' then 1, else 0, end) as marketplace
	sum(case when amenity in('marketplace', 'toilets', 'pharmacy') then 1 else 0, end) as mixeduses,
	st_snaptogrid(st_transform(geometry,3857),400) as geom
	from points
	group by geom
	
---------------------------------------------------------------------------
-- limit: sometimes it has to be put into the query, not at the end

---------------------------------------------------------------------------
functions
-- avg
-- max
-- sum
-- min
-- count

--my own successful snaptogrid query:

create table cafegrid as 
select 
PKUID, osm_id, amenity, tags, geometry,
count() as c,
st_snaptogrid(st_transform("geometry", 3857),200) as geom 
from osm_cafe
group by geom

--st_distance for ya:

create table XXXX
select 
t1.*, 
st_distance(st_transform(t1.Geometry, 3857), st_transform(t2.Geometry, 3857)) as distance
from transportnodes as t1, b.osm_workplaces as t2
where distance <1000
limit 10

--SUNBURST / SPIDERWEBS

create table metro_l_cafe as
select count() as count, makeline(t1.Geometry, t2.geometry) as lines
from metrostations as t1, a.osm_cafe as t2
where st_distance(st_transform(t1.Geometry, 3857), st_transform(t2.geometry, 3857)) < 750
group by t2.geometry
limit 500


select PKUID, osm_id, name, tags, numrest, geometry, count() as c, 
st_snaptogrid(st_transform("geometry", 3857),200) as geom , avg(numrest)
 from cafe_x_rest
group by distance

create table trans_x_cafe_grid as 
select 
PKUID, id, osm_id, name, type, ref, distance, avg_distance, geometry,
count() as c,
st_snaptogrid(st_transform("geometry", 3857),100) as geom ,
avg(avg_distance)
from trans_x_cafe_avg
group by geom

--trying to create a table of the district shapefiles, with names, population, area, and density
select 
a.Chinese as name_chinese, 
a.Hanyu_Pinyin as name_pinyin, 
a.Name as name_english,
cast(a.Area__km²__3_ as real) as areakm2, 
cast(a.Population__2015_census__4_ as int) as population, 
cast(a.Density___km²_ as real) as density,
b.geometry as geometry
from Shanghai_districts as a
left join 
a.shang_dis_merged as b
on a.Chinese like b.Name
//////
select 
a.Name as name_chinese, 
b.Hanyu_Pinyin as name_pinyin, 
b.Name as name_english,
cast(b.Area__km²__3_ as real) as areakm2, 
cast(b.Population__2015_census__4_ as int) as population, 
cast(b.Density___km²_ as real) as density,
a.geometry as geometry
from Shanghai_districts as b
left join 
a.shang_dis_merged as a
on a.Name like b.Chinese

--creating a table of all restaurants (dianping, bonapp, meituan, tripadvisor.....) 
--EXCEPT HOW TO TAKE THE '€€€€' COLUMN FROM TRIPADVISOR AND TURN IT INTO A REAL IN A QUERY?????

select name, (cast(dislikeVotes as int) + cast(likeVotes as int)) as comments, case when averagePrice like '< ¥100' then 75 when averagePrice like '< ¥200' then 150 when averagePrice like '< ¥300' then 250 when averagePrice like '> ¥300' then 350 end as price, geometry 
from a.bonapp
where geometry is not null
union all 
select name, cast(comments as int), cast(price as real), geometry 
from b.dianping
where geometry is not null
union all 
select title, cast(allCommentNum as int), cast(avgPrice as real), geometry 
from c.meituan
where geometry is not null
union all 
select name, cast(aggregateRating_reviewCount as int), case when d.priceRange like '€' then 75 when d.priceRange like '€€ - €€€' then 200 when d.priceRange like '€€€€' then 300 else null end, geometry 
from d.tripadvisor as d
where geometry is not null
union all
select wmPoi4Web__name, null, wmPoi4Web__wmCPoiLbs__min_price, geometry from e.waimay
where geometry is not null

--creating a square grid over everything
--you can't do it over EVERYTHING, but you can do it over the large district shapefiles

select #name#, st_squaregrid(st_collect(st_transform(Geometry, 3857)), 250) as geom
from shang_dis_merged

--table of districts, grouped by average restaurant price inside

create table sh_rest_x_distr2 as 
select t1.Geometry, count(t2.geometry), avg(t2.price)
from shanghai_districts_new as t1, a.all_restaurants as t2
where st_intersects(t1.geometry, t2.geometry)

--'snap' operation snaps geom1 to closest point on geom2

create table blocks_x_tweetJan as
select count() as count, snap(t1.GEOMETRY, t2.Geometry, 0.5)
from twitter_jan as t1, bcn_blocks as t2
group by t2.Geometry

create table streets_x_tweetJan as
select t1.*, st_snap(t1.geometry, st_segmentize(st_transform(t2.geometry, 3857), 100), 1) as snappedpoints
from twitter_jan as t1, a.bcn.streets as t2
group by snappedpoints
###maybe you should create the segmentized geometry first, then snap to them in a second query so that you can group by them

--the following counts tweets inside each sscc, and also aggregates the topic scores for each sscc
create table dist_x_tweetDec as 
select 
t1.Geometry, count(t2.GEOMETRY), sum(t2.ambig) as ambig, sum(t2.culture) as culture, sum(t2.politics)as politics, sum(t2.tech) as tech, sum(t2.travel) as travel, sum(t2.sports) as sports, sum(t2.lifestyle) as lifestyle, sum(t2.social) as social, sum(t2.food) as food
from a.BCN_SSCC as t1, twitter_dec as t2
where st_intersects(t1.Geometry, t2.GEOMETRY)
group by t1.Geometry


TEMP


create table sscc_x_hashtag as
select t1.PK_UID, t1.OBJECTID_1, t1.Area, t1.Geometry, sum (t2."retweet count"), st_intersects(t1, t2)
from BCN_SSCC as t1, twitter_bcn_all_geolocated as t2
where t2."hashtag text" like '%arcelona%'

ca, da, de, en, en-gb, es, fr, gl, hu, id, it, ja, nl, pl, pt, ru, sv, th, tr, uk

create table week3_lang as
select  
sum(case when "USER LANG" like 'ca' then 1 else 0 end) as catalan,
sum(case when "USER LANG" like 'da' then 1 else 0 end) as danish,
sum(case when "USER LANG" like 'de' then 1 else 0 end) as german,
sum(case when "USER LANG" like 'en' or "USER LANG" like 'en-gb' then 1 else 0 end) as english,
sum(case when "USER LANG" like 'es' then 1 else 0 end) as spanish,
sum(case when "USER LANG" like 'fr' then 1 else 0 end) as french,
sum(case when "USER LANG" like 'gl' then 1 else 0 end) as galician,
sum(case when "USER LANG" like 'hu' then 1 else 0 end) as hungarian,
sum(case when "USER LANG" like 'id' then 1 else 0 end) as hindi,
sum(case when "USER LANG" like 'it' then 1 else 0 end) as italian,
sum(case when "USER LANG" like 'ja' then 1 else 0 end) as japanese,
sum(case when "USER LANG" like 'nl' then 1 else 0 end) as dutch,
sum(case when "USER LANG" like 'pl' then 1 else 0 end) as polish,
sum(case when "USER LANG" like 'pt' then 1 else 0 end) as portugese,
sum(case when "USER LANG" like 'ru' then 1 else 0 end) as russian,
sum(case when "USER LANG" like 'sv' then 1 else 0 end) as swedish,
sum(case when "USER LANG" like 'th' then 1 else 0 end) as thai,
sum(case when "USER LANG" like 'tr' then 1 else 0 end) as turkish,
sum(case when "USER LANG" like 'uk' then 1 else 0 end) as ukranian
from week3_171121

create table week7_lang as
select  
sum(case when "USER LANG" like 'ca' then 1 else 0 end) as catalan,
sum(case when "USER LANG" like 'ar' then 1 else 0 end) as arabic,
sum(case when "USER LANG" like 'de' then 1 else 0 end) as german,
sum(case when "USER LANG" like 'en' or "USER LANG" like 'en-gb' then 1 else 0 end) as english,
sum(case when "USER LANG" like 'es' then 1 else 0 end) as spanish,
sum(case when "USER LANG" like 'fr' then 1 else 0 end) as french,
sum(case when "USER LANG" like 'gl' then 1 else 0 end) as galician,

sum(case when "USER LANG" like 'id' then 1 else 0 end) as hindi,
sum(case when "USER LANG" like 'it' then 1 else 0 end) as italian,
sum(case when "USER LANG" like 'ja' then 1 else 0 end) as japanese,
sum(case when "USER LANG" like 'nl' then 1 else 0 end) as dutch,

sum(case when "USER LANG" like 'pt' then 1 else 0 end) as portugese,
sum(case when "USER LANG" like 'ru' then 1 else 0 end) as russian,


sum(case when "USER LANG" like 'tr' then 1 else 0 end) as turkish,

sum(case when "USER LANG" like 'zh-cn' then 1 else 0 end) as chinese
from week7_171219

create table week17_lang as
select  
sum(case when "USER LANG" like 'ca' then 1 else 0 end) as catalan,

sum(case when "USER LANG" like 'de' then 1 else 0 end) as german,
sum(case when "USER LANG" like 'el' then 1 else 0 end) as greek,
sum(case when "USER LANG" like 'en' or "USER LANG" like 'en-gb' or "USER LANG" like 'en-GB' then 1 else 0 end) as english,
sum(case when "USER LANG" like 'es' then 1 else 0 end) as spanish,
sum(case when "USER LANG" like 'fr' then 1 else 0 end) as french,
sum(case when "USER LANG" like 'gl' then 1 else 0 end) as galician,


sum(case when "USER LANG" like 'it' then 1 else 0 end) as italian,
sum(case when "USER LANG" like 'ja' then 1 else 0 end) as japanese,
sum(case when "USER LANG" like 'nl' then 1 else 0 end) as dutch,
sum(case when "USER LANG" like 'pl' then 1 else 0 end) as polish,
sum(case when "USER LANG" like 'pt' then 1 else 0 end) as portugese,
sum(case when "USER LANG" like 'ru' then 1 else 0 end) as russian,
sum(case when "USER LANG" like 'sr' then 1 else 0 end) as serbian,


sum(case when "USER LANG" like 'tr' then 1 else 0 end) as turkish,

sum(case when "USER LANG" like 'zh-cn' then 1 else 0 end) as chinese
from week17_180227

--snaptogrid per week

create table week3_grid_200 as
select
st_snaptogrid("Geometry",200) as geom, count() as c, 
sum(AMBIG) as AMBIG,
sum(CULTURE) as CULTURE,
sum(POLITICS) as POLITICS,
sum(TECH) as TECH,
sum(TRAVEL) as TRAVEL,
sum(SPORTS) as SPORTS,
sum(LIFESTYLE) as LIFESTYLE,
sum(SOCIAL) as SOCIAL,
sum(FOOD) as FOOD
from week3 as a
group by geom

--create table around each metro station
create table metro_food as
select t1.id as id, t1.name as name, t1.Geometry as geometry, count(t2.geometry) as restaurants, avg(t2.price) as average_price, min(t2.price) as min_price, max(t2.price) as max_price, sum(t2.comments) as total_comments, avg(t2.comments) as avg_comments,
sum(case when
t3.cuisine like 'American' or 
t3.cuisine like 'Australian' or
t3.cuisine like 'Australian, Bar' or
t3.cuisine like 'Bakery & Pastries, French' or
t3.cuisine like 'Bar, American' or
t3.cuisine like 'Bar, European' or
t3.cuisine like 'Bar, French' or
t3.cuisine like 'Bar, Fusion' or
t3.cuisine like 'Bar, German' or
t3.cuisine like 'Bar, Irish' or
t3.cuisine like 'Bar, Italian' or
t3.cuisine like 'Bar, Mediterranean' or
t3.cuisine like 'Bar, North American' or
t3.cuisine like 'Bar, Pizza' or
t3.cuisine like 'All Western, Global Cuisine' or
t3.cuisine like 'Barbecue' or
t3.cuisine like 'British' or
t3.cuisine like 'Cafe, European' or
t3.cuisine like 'Cafe, Italian' or
t3.cuisine like 'Cafe, North American' or
t3.cuisine like 'Dessert, American' or
t3.cuisine like 'European' or
t3.cuisine like 'European, Bar' or
t3.cuisine like 'European, Cafe' or
t3.cuisine like 'European, Dessert' or
t3.cuisine like 'European, Fast Casual' or
t3.cuisine like 'European, Mediterranean' or
t3.cuisine like 'European, Refined Homestyle' or
t3.cuisine like 'European, Seafood' or
t3.cuisine like 'French' or
t3.cuisine like 'French, Bakery & Pastries' or
t3.cuisine like 'French, Bar' or
t3.cuisine like 'French, Cafe' or
t3.cuisine like 'French, European' or
t3.cuisine like 'French, Wine Bar' or
t3.cuisine like 'German' or
t3.cuisine like 'Italian' or
t3.cuisine like 'Italian, Bar' or
t3.cuisine like 'Italian, Bar/Lounge/Pub' or
t3.cuisine like 'Italian, Pizza' or
t3.cuisine like 'Italian, Seafood' or
t3.cuisine like 'Italian, Steakhouse' or
t3.cuisine like 'Mexican / Tex-Mex' or
t3.cuisine like 'Mexican/Tex-Mex' or
t3.cuisine like 'Mexican/Tex-Mex, Italian' or
t3.cuisine like 'Mexican/Tex-Mex, North American' or
t3.cuisine like 'Middle Eastern' or
t3.cuisine like 'Middle Eastern, Mediterranean' or
t3.cuisine like 'Middle Eastern, Sandwiches & Delis' or
t3.cuisine like 'Middle Eastern, Turkish' or
t3.cuisine like 'North American' or
t3.cuisine like 'North American, Bar' or
t3.cuisine like 'North American, Cafe' or
t3.cuisine like 'North American, Dessert' or
t3.cuisine like 'North American, Fast Casual' or
t3.cuisine like 'North American, French' or
t3.cuisine like 'North American, Fusion' or
t3.cuisine like 'North American, Global Cuisine' or
t3.cuisine like 'North American, Grocery' or
t3.cuisine like 'North American, Juice & Beverages' or
t3.cuisine like 'North American, Sandwiches & Delis' or
t3.cuisine like 'North American, Steakhouse' or
t3.cuisine like 'North American, Vegetarian' or
t3.cuisine like 'North American, Wine Bar' or
t3.cuisine like 'Pizza' or
t3.cuisine like 'Pizza, Bar/Lounge/Pub' or
t3.cuisine like 'Pizza, Italian' or
t3.cuisine like 'Pizza, North American' or
t3.cuisine like 'Portuguese' or
t3.cuisine like 'Sandwiches & Delis, Mediterranean' or
t3.cuisine like 'Seafood, North American' or
t3.cuisine like 'Spanish' or
t3.cuisine like 'South American' or
t3.cuisine like 'South American, Spanish' or
t3.cuisine like 'South American, Steakhouse' or
t3.cuisine like 'Steakhouse' or
t3.cuisine like 'Steakhouse, American' or
t3.cuisine like 'Turkish' or
t3.cuisine like 'Turkish, Mediterranean' or
t3.cuisine like 'Wine Bar, French' then 1 else 0 end) as western,
sum(case when
t3.cuisine like 'Cantonese' or
t3.cuisine like 'Cantonese, Dim Sum' or
t3.cuisine like 'Cantonese, Fast Casual' or
t3.cuisine like 'Cantonese, Fusion' or
t3.cuisine like 'Chinese' or
t3.cuisine like 'Chinese, Cantonese' or
t3.cuisine like 'Chinese, Shanghainese' or
t3.cuisine like 'Dim Sum, Bar' or
t3.cuisine like 'Dim Sum, Chinese' or
t3.cuisine like 'Dongbei, Chinese' or
t3.cuisine like 'Hot Pot' or
t3.cuisine like 'Hot Pot, Chinese' or
t3.cuisine like 'Hot Pot, Taiwanese' or
t3.cuisine like 'Hunan, Chinese' or
t3.cuisine like 'Indonesian, Southeast Asian' or
t3.cuisine like 'Japanese' or
t3.cuisine like 'Japanese, Sushi' or
t3.cuisine like 'Jiangsu/Zhejiang' or
t3.cuisine like 'Korean' or
t3.cuisine like 'Seafood, Singaporean' or
t3.cuisine like 'Shanghainese, Chinese' or
t3.cuisine like 'Shanghainese, Fast Casual' or
t3.cuisine like 'Sichuan, Chinese' or
t3.cuisine like 'Sichuan, Noodle Shop' or
t3.cuisine like 'Singaporean' or
t3.cuisine like 'Sushi' or
t3.cuisine like 'Sushi, Japanese' or
t3.cuisine like 'Taiwanese' or
t3.cuisine like 'Thai' or
t3.cuisine like 'Thai, Barbecue' or
t3.cuisine like 'Thai, Southeast Asian' or
t3.cuisine like 'Xibei/Xinjiang' or
t3.cuisine like 'Yunnan' or
t3.cuisine like 'Yunnan, Bar' or
t3.cuisine like 'Yunnan, Chinese' or
t3.cuisine like 'Vietnamese' or
t3.cuisine like 'Vietnamese, Noodle Shop' or
t3.cuisine like 'Southeast Asian' or
t3.cuisine like 'Southeast Asian, Bar/Lounge/Pub' or
t3.cuisine like 'Southeast Asian, Thai' then 1 else 0 end) as eastern,
sum(case when
t3.cuisine like 'Sandwiches & Delis' or
t3.cuisine like 'Sandwiches & Delis, Bar' or
t3.cuisine like 'Seafood' or
t3.cuisine like 'Seafood, Fast Casual' or
t3.cuisine like 'Seafood, Sandwiches & Delis' or
t3.cuisine like 'Skewers, Barbecue' or
t3.cuisine like 'Vegetarian' or
t3.cuisine like 'Vegetarian, Global Cuisine' or
t3.cuisine like 'Wine Bar' then 1 else 0 end) as other
from shanghai_L2Metro_stations_3857 as t1, all_food as t2, a.bonapp as t3
where st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t2.geometry, 3857))
and st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t3.geometry, 3857))
group by t1.id

--leave average price calculation for later:
update metro_food set average_price =
(select avg(t2.price) 
from shanghai_L2Metro_stations_3857 as t1, all_food as t2
where t2.price is not null 
and t2.price is not 0
and st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t2.geometry, 3857))
group by t1.id)

--alternately, use the above query to make a new table called average_price, then join it with metro_food
SELECT * 
from metro_food, average_price
WHERE  average_price.ROWID = metro_food.id

--add median price later (this one works!)
create table median_price as
SELECT t1.id as id, AVG(t2.price) as median_price
FROM (SELECT t2.price
FROM all_food as t2
ORDER BY t2.price
LIMIT 2
OFFSET (SELECT (COUNT(t2.price) - 1) / 2
FROM all_food as t2)),
shanghai_l2Metro_stations_3857 as t1, all_food as t2
where st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t2.geometry, 3857))
group by t1.id

--

select (t1.id as id), (t2.price as median_price
from shanghai_L2Metro_stations_3857 as t1, all_food as t2
where st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t2.geometry, 3857))
order by price
limit 1
offset (select count(t2.price) / 2)
group by t1.id)


--distance calculations were rejected... in case you need them later..
select
avg(st_distance(st_transform(t1.Geometry, 3857), st_transform(t2.geometry, 3857))) as avg_distance_food, avg(st_distance(st_transform(t1.Geometry, 3857), st_transform(t4.Geometry, 3857))) as avg_distance_amenity
from shanghai_L2Metro_stations as t1, all_restaurants as t2, a.bonapp as t3, b.shanghai_china_osm_amenities as t4
where st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t2.geometry, 3857))
and (st_distance(st_transform(t1.Geometry, 3857), st_transform(t2.geometry, 3857))) < 750

--add eastern/western ratio later
update MetroL2_x_food set eastern_western_ratio =
cast(eastern as real)
/
cast(western as real)

--add amenities from osm
update t1 set amenities =
count(t2.geometry)
from MetroL2_x_food as t1, shanghai_osm_points as t2
where st_intersects(t1.Geometry, t2.geometry)
group by MetroL2_x_food."id"

--what other calcs could you add to metro_food?
-total road length 
-number of distinct amenities
(all of these within the buffer)
-comparison with metro buffers adjacent
-IDEALLY, the buffers would not be simple circular buffers, but voronoi catchment areas.
-food density within metro catchment area
-amenity density within metro catchment area
-population per catchment area
-each person consumes about 2kg of food per day, and produces 2kg of waste

--separate cuisine query:
create table cuisine2 as
select t1.id as id, sum(case when
t3.cuisine like 'Cantonese' or
t3.cuisine like 'Cantonese, Dim Sum' or
t3.cuisine like 'Cantonese, Fast Casual' or
t3.cuisine like 'Cantonese, Fusion' or
t3.cuisine like 'Chinese' or
t3.cuisine like 'Chinese, Cantonese' or
t3.cuisine like 'Chinese, Shanghainese' or
t3.cuisine like 'Dim Sum, Bar' or
t3.cuisine like 'Dim Sum, Chinese' or
t3.cuisine like 'Dongbei, Chinese' or
t3.cuisine like 'Hot Pot' or
t3.cuisine like 'Hot Pot, Chinese' or
t3.cuisine like 'Hot Pot, Taiwanese' or
t3.cuisine like 'Hunan, Chinese' or
t3.cuisine like 'Indonesian, Southeast Asian' or
t3.cuisine like 'Japanese' or
t3.cuisine like 'Japanese, Sushi' or
t3.cuisine like 'Jiangsu/Zhejiang' or
t3.cuisine like 'Korean' or
t3.cuisine like 'Seafood, Singaporean' or
t3.cuisine like 'Shanghainese, Chinese' or
t3.cuisine like 'Shanghainese, Fast Casual' or
t3.cuisine like 'Sichuan, Chinese' or
t3.cuisine like 'Sichuan, Noodle Shop' or
t3.cuisine like 'Singaporean' or
t3.cuisine like 'Sushi' or
t3.cuisine like 'Sushi, Japanese' or
t3.cuisine like 'Taiwanese' or
t3.cuisine like 'Thai' or
t3.cuisine like 'Thai, Barbecue' or
t3.cuisine like 'Thai, Southeast Asian' or
t3.cuisine like 'Xibei/Xinjiang' or
t3.cuisine like 'Yunnan' or
t3.cuisine like 'Yunnan, Bar' or
t3.cuisine like 'Yunnan, Chinese' or
t3.cuisine like 'Vietnamese' or
t3.cuisine like 'Vietnamese, Noodle Shop' or
t3.cuisine like 'Southeast Asian' or
t3.cuisine like 'Southeast Asian, Bar/Lounge/Pub' or
t3.cuisine like 'Southeast Asian, Thai' then 1 else 0 end) as eastern, 
sum(case when
t3.cuisine like 'American' or 
t3.cuisine like 'Australian' or
t3.cuisine like 'Australian, Bar' or
t3.cuisine like 'Bakery & Pastries, French' or
t3.cuisine like 'Bar, American' or
t3.cuisine like 'Bar, European' or
t3.cuisine like 'Bar, French' or
t3.cuisine like 'Bar, Fusion' or
t3.cuisine like 'Bar, German' or
t3.cuisine like 'Bar, Irish' or
t3.cuisine like 'Bar, Italian' or
t3.cuisine like 'Bar, Mediterranean' or
t3.cuisine like 'Bar, North American' or
t3.cuisine like 'Bar, Pizza' or
t3.cuisine like 'All Western, Global Cuisine' or
t3.cuisine like 'Barbecue' or
t3.cuisine like 'British' or
t3.cuisine like 'Cafe, European' or
t3.cuisine like 'Cafe, Italian' or
t3.cuisine like 'Cafe, North American' or
t3.cuisine like 'Dessert, American' or
t3.cuisine like 'European' or
t3.cuisine like 'European, Bar' or
t3.cuisine like 'European, Cafe' or
t3.cuisine like 'European, Dessert' or
t3.cuisine like 'European, Fast Casual' or
t3.cuisine like 'European, Mediterranean' or
t3.cuisine like 'European, Refined Homestyle' or
t3.cuisine like 'European, Seafood' or
t3.cuisine like 'French' or
t3.cuisine like 'French, Bakery & Pastries' or
t3.cuisine like 'French, Bar' or
t3.cuisine like 'French, Cafe' or
t3.cuisine like 'French, European' or
t3.cuisine like 'French, Wine Bar' or
t3.cuisine like 'German' or
t3.cuisine like 'Italian' or
t3.cuisine like 'Italian, Bar' or
t3.cuisine like 'Italian, Bar/Lounge/Pub' or
t3.cuisine like 'Italian, Pizza' or
t3.cuisine like 'Italian, Seafood' or
t3.cuisine like 'Italian, Steakhouse' or
t3.cuisine like 'Mexican / Tex-Mex' or
t3.cuisine like 'Mexican/Tex-Mex' or
t3.cuisine like 'Mexican/Tex-Mex, Italian' or
t3.cuisine like 'Mexican/Tex-Mex, North American' or
t3.cuisine like 'Middle Eastern' or
t3.cuisine like 'Middle Eastern, Mediterranean' or
t3.cuisine like 'Middle Eastern, Sandwiches & Delis' or
t3.cuisine like 'Middle Eastern, Turkish' or
t3.cuisine like 'North American' or
t3.cuisine like 'North American, Bar' or
t3.cuisine like 'North American, Cafe' or
t3.cuisine like 'North American, Dessert' or
t3.cuisine like 'North American, Fast Casual' or
t3.cuisine like 'North American, French' or
t3.cuisine like 'North American, Fusion' or
t3.cuisine like 'North American, Global Cuisine' or
t3.cuisine like 'North American, Grocery' or
t3.cuisine like 'North American, Juice & Beverages' or
t3.cuisine like 'North American, Sandwiches & Delis' or
t3.cuisine like 'North American, Steakhouse' or
t3.cuisine like 'North American, Vegetarian' or
t3.cuisine like 'North American, Wine Bar' or
t3.cuisine like 'Pizza' or
t3.cuisine like 'Pizza, Bar/Lounge/Pub' or
t3.cuisine like 'Pizza, Italian' or
t3.cuisine like 'Pizza, North American' or
t3.cuisine like 'Portuguese' or
t3.cuisine like 'Sandwiches & Delis, Mediterranean' or
t3.cuisine like 'Seafood, North American' or
t3.cuisine like 'Spanish' or
t3.cuisine like 'South American' or
t3.cuisine like 'South American, Spanish' or
t3.cuisine like 'South American, Steakhouse' or
t3.cuisine like 'Steakhouse' or
t3.cuisine like 'Steakhouse, American' or
t3.cuisine like 'Turkish' or
t3.cuisine like 'Turkish, Mediterranean' or
t3.cuisine like 'Wine Bar, French' then 1 else 0 end) as western,
sum(case when
t3.cuisine like 'Sandwiches & Delis' or
t3.cuisine like 'Sandwiches & Delis, Bar' or
t3.cuisine like 'Seafood' or
t3.cuisine like 'Seafood, Fast Casual' or
t3.cuisine like 'Seafood, Sandwiches & Delis' or
t3.cuisine like 'Skewers, Barbecue' or
t3.cuisine like 'Vegetarian' or
t3.cuisine like 'Vegetarian, Global Cuisine' or
t3.cuisine like 'Wine Bar' then 1 else 0 end) as other
from shanghai_L2Metro_stations_3857 as t1, a.bonapp as t3, all_food as t2
where st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t2.geometry, 3857))
and st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t3.geometry, 3857))
group by t1.id

--joining multiple tables:
create table metro_food_NEW as
SELECT metro_food.*, average_price.*, cuisine.*, median_price.*, minimum_price.*
FROM metro_food
JOIN 
average_price ON average_price.ID = metro_food.ID
JOIN
cuisine ON cuisine.ID = metro_food.ID
JOIN 
median_price ON median_price.ID = metro_food.ID
JOIN 
minimum_price ON minimum_price.ID = metro_food.ID
WHERE metro_food.id like average_price.id

--create table for box plot
create table boxplot_1 as
select 
t2.price from all_food as t2, shanghai_L2Metro_stations as t1 where st_intersects(st_buffer(st_transform(t1.Geometry, 3857), 750), st_transform(t2.geometry, 3857)) and t1.id is 1
.....
price from all_food where st_intersects(st_buffer(st_transform(shanghai_L2Metro_stations.Geometry, 3857), 750), st_transform(all_food.geometry, 3857)) and shanghai_L2Metro_stations.id is 2,
price from all_food where st_intersects(st_buffer(st_transform(shanghai_L2Metro_stations.Geometry, 3857), 750), st_transform(all_food.geometry, 3857)) and shanghai_L2Metro_stations.id is 3,
price from all_food where st_intersects(st_buffer(st_transform(shanghai_L2Metro_stations.Geometry, 3857), 750), st_transform(all_food.geometry, 3857)) and shanghai_L2Metro_stations.id is 4,
price from all_food where st_intersects(st_buffer(st_transform(shanghai_L2Metro_stations.Geometry, 3857), 750), st_transform(all_food.geometry, 3857)) and shanghai_L2Metro_stations.id is 5

--TIPS FROM PABLO:
--this is a subquery:
(select min(t2.price) where t2.price is not null)
--try indexing columns. check sqlite library. this will be more work in the beginning, and make the database maybe bigger, but ultimately you will just be able to filter by values which is faster down the line.

--Calculating road length:
SELECT t1.*, st_length(st_transform(t2.Geometry, 3857)) as road_length
from osm_subway_stations_L2_voronoi as t1, shanghai_china_osm_roads as t2
where st_intersects(st_transform(t1.Geometry, 3857), st_transform(t2.Geometry, 3857))
group by t1.Geometry
