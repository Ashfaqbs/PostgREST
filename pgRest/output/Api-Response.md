

## Get All:

http://localhost:3000/employee

```
[
{
"id": 1,
"emp_code": 101,
"name": "John Doe"
},
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith"
},
{
"id": 35,
"emp_code": 101,
"name": "Emily Davis"
}
]
```

## Get all but sorted by name


2. http://localhost:3000/employee?order=name.asc



```
[
{
"id": 35,
"emp_code": 101,
"name": "Emily Davis"
},
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith"
},
{
"id": 1,
"emp_code": 101,
"name": "John Doe"
}
]
```
## Get 1

3. http://localhost:3000/employee?limit=1


```
[
{
"id": 1,
"emp_code": 101,
"name": "John Doe"
}
]
```

## Create 


3. Create a new employee

http://localhost:3000/employee

```
{
  "emp_code": 103,
  "name": "Alice Wonderland"
}

```
OP 201


## Update Via Patch

http://localhost:3000/employee?id=eq.1

```
{
  "name": "Johnathan Doe"
}


OP 



[
    {
        "id": 1,
        "emp_code": 101,
        "name": "Johnathan Doe"
    }
]
```

## Delete
http://localhost:3000/employee?id=eq.1

OP 204




---------------------------------------------------------------------------

## Joins Via The embedded resource Not Custom
- a feature relies on defined foreign key relationships
one tables
```
SELECT id, emp_code, "name"
FROM public.employee;

2	102	Jane Smith
35	101	Emily Davis
4	103	Alice Wonderland


SELECT employee_id, skill
FROM public.employee_skills;

OP
2	PHP
2	Ruby
35	PHP
35	Ruby

```

URL 
http://localhost:3000/employee?select=id,emp_code,name,employee_skills(skill)


```
[
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith",
"employee_skills": [
{
"skill": "PHP"
},
{
"skill": "Ruby"
},
{
"skill": "PHP"
},
{
"skill": "Ruby"
},
{
"skill": "PHP"
},
{
"skill": "Ruby"
}
]
},
{
"id": 35,
"emp_code": 101,
"name": "Emily Davis",
"employee_skills": [
{
"skill": "PHP"
},
{
"skill": "Ruby"
}
]
},
{
"id": 4,
"emp_code": 103,
"name": "Alice Wonderland",
"employee_skills": []
}
]
```




http://localhost:3000/employee_skills?select=employee_id,skill,employee(id,emp_code,name)


```
[
{
"employee_id": 2,
"skill": "PHP",
"employee": {
"id": 2,
"name": "Jane Smith",
"emp_code": 102
}
},
{
"employee_id": 2,
"skill": "Ruby",
"employee": {
"id": 2,
"name": "Jane Smith",
"emp_code": 102
}
},
{
"employee_id": 2,
"skill": "PHP",
"employee": {
"id": 2,
"name": "Jane Smith",
"emp_code": 102
}
},
{
"employee_id": 2,
"skill": "Ruby",
"employee": {
"id": 2,
"name": "Jane Smith",
"emp_code": 102
}
},
{
"employee_id": 2,
"skill": "PHP",
"employee": {
"id": 2,
"name": "Jane Smith",
"emp_code": 102
}
},
{
"employee_id": 2,
"skill": "Ruby",
"employee": {
"id": 2,
"name": "Jane Smith",
"emp_code": 102
}
},
{
"employee_id": 35,
"skill": "PHP",
"employee": {
"id": 35,
"name": "Emily Davis",
"emp_code": 101
}
},
{
"employee_id": 35,
"skill": "Ruby",
"employee": {
"id": 35,
"name": "Emily Davis",
"emp_code": 101
}
}
]

```




## Running custom api using views

```
CREATE VIEW employee_with_skills AS
SELECT e.id, e.emp_code, e.name, es.skill
FROM public.employee e
LEFT JOIN public.employee_skills es ON e.id = es.employee_id;


select * from employee_with_skills;

OP
2	102	Jane Smith	PHP
2	102	Jane Smith	Ruby
2	102	Jane Smith	PHP
2	102	Jane Smith	Ruby
2	102	Jane Smith	PHP

```
API - http://localhost:3000/employee_with_skills


```
[
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith",
"skill": "PHP"
},
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith",
"skill": "Ruby"
},
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith",
"skill": "PHP"
},
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith",
"skill": "Ruby"
},
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith",
"skill": "PHP"
},
{
"id": 2,
"emp_code": 102,
"name": "Jane Smith",
"skill": "Ruby"
},
{
"id": 35,
"emp_code": 101,
"name": "Emily Davis",
"skill": "PHP"
},
{
"id": 35,
"emp_code": 101,
"name": "Emily Davis",
"skill": "Ruby"
},
{
"id": 4,
"emp_code": 103,
"name": "Alice Wonderland",
"skill": null
}
]
```

Note Custom can be implemented via - view or stored procedure (RPC)

- Dynamic SQL Function

```
CREATE OR REPLACE FUNCTION public.run_query(query text)
RETURNS json
LANGUAGE plpgsql
AS $$
DECLARE
  results json;
BEGIN
  EXECUTE 'SELECT json_agg(t) FROM (' || query || ') t' INTO results;
  RETURN results;
END;
$$;

```


Post  http://localhost:3000/rpc/run_query 

Request Body :
```
{
  "query": "SELECT \"transaction\", price, transfer_date, postcode, property_type, newly_built, duration, paon, saon, street, locality, city, district, county, ppd_category_type, record_status FROM public.land_registry_price_paid_uk LIMIT 1"
}

```

Response:
```

[
    {
        "transaction": "ffa361da-c01a-8a03-e053-4804a8c01f61",
        "price": 270000,
        "transfer_date": "2023-06-01",
        "postcode": "SS2 4AX",
        "property_type": "S",
        "newly_built": false,
        "duration": "F",
        "paon": "6",
        "saon": null,
        "street": "WEYBOURNE CLOSE",
        "locality": null,
        "city": "SOUTHEND-ON-SEA",
        "district": "SOUTHEND-ON-SEA",
        "county": "SOUTHEND-ON-SEA",
        "ppd_category_type": "A",
        "record_status": "A"
    }]
```