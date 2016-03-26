#### pspsqldocdb
#####2
######1 json and jsonb
```
select pg_typeof('{"name":"ke"}'::json);
```
json can store duplicate key, preserve spaces  
jsonb does not allow duplicate key ,not preserve spaces    

convert text to json
```
select to_json('{"name":"ke"}'::text);
```

get value from json. single arrow and double arrow
```
select '{"name":"ke"}'::jsonb ->'name'
select '{"name":"ke"}'::jsonb ->>'name'
```


######sample
```
createdb newdb
psql newdb < newdb.sql
```

######json creation
```
select row_to_json(tbname) from tbname;
```

Or, using subquery
```
select row_to_json(x) from (select a,b from tbname) x;
```

using json_agg to return an array of results.
```
select *, (select json_agg(a) from (select...)a ) as alias from tbname;
```


text to json
```
select to_json (array[1,2]);
```

build array
```
select json_build_array (1,3,4);
