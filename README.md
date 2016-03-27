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
```

build object from array, must contain even numbers of elements
```
select json_build_object (1,3,4,5); 
```
will return
```
{"1" : 3, "4" : 5}
```

json_object func (syntax is strange)
```
select json_object ('{k1,k2}','{v1,v2}');
```
return
```
{"k1" : "v1", "k2" : "v2"}
```

or:
```
select json_object ('{k1,v1, k2,v2}');
```
return
```
{"k1" : "v1", "k2" : "v2"}
```

(from now, tbname contains two fields, id and body)  


Nested subquery
```
select row_to_json(x) from (select *, (select json_agg(a) from (select....)a) as alias from tbname)x;
```
######JSON process func
```
select *, jsonb_object_keys(body) from tbname
```
json.key, json.value
```
select json.key json.value from artist_docs, jsonb_each(tbname.body) json
select json.key json.value from artist_docs, json_each(tbname.body::json) json     //same as above
```

jsonb_to_record 
```
select j.* from tbname, jsonb_to_record(tbname.body) as j ( id int,name varchar(255))
```
######Existence operator
```
select * from tbname where body -> 'field'?'name'
```
######containment operator
```
select * from tbname where body @> '{"name":"value"}'
select * from tbname where body -> "name" @> '{"value"}'
```
######fuzzy
```
select * from tbname where body ->> 'name' LIKE '%x";
select * from tbname where (body ->> 'id')::int > 100;
```

######indexing

create index
```
create index idx_name on tbname using GIN(fname)
```

```
select pg_size_pretty(pg_relation_size('idx_name'::regclass)) as alias
```
prepare a index for jsonb
```
create index idx_name on tbname using GIN(fname jsonb_path_ops)
```
index for specific field
```
create index idx_name on tbname using GIN((fname -> 'name') jsonb_path_ops)
```

######plv8
```
create function json(out res varchar){
  as $$
   var cus = {"name":"ke"};
   var sql = "insert into cus(body) values($1) returning *;";
   var new = plv8.execute(sql,JSON.stringify(cus));
   return new[0].id
  $$ languare plv8;
-- call the function
select json()
```
review
```
row_to_json json_agg jsonb_to_record jsonb_array_elements
```

#####3 Document API
######1 Document table
```
create table doc{
  id serial primary key,
  body jsonb not null,
  search tsvector,
  created_at timestemptz default now() not null,
  updated_at timestemptz default now() not null
}

create idx on doc using GIN(body jsonb_path_ops)
create idx_s on doc using GIN(search)
```
