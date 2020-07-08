# Best practicies

## React Native

#### Debugging network requests in react-native
```javascript
// To see all the requests in the chrome Dev tools in the network tab.
XMLHttpRequest = GLOBAL.originalXMLHttpRequest ?
    GLOBAL.originalXMLHttpRequest :
    GLOBAL.XMLHttpRequest;
```
Don't forget to remove it before pushing!

#### Open link in a running simulator

```sh
xcrun simctl openurl booted link-goes-here
```


## Backend (Python)

#### Always use _id instead of id in search params
Aidbox accepts both id and _id search params. This is causing problems with access policies.
We should always use "_id" instead of "id" search params on frontend as well as backend part of an app.

### QA, Backend Tests

#### Mocking free functions
Use module path the function is looked up in (it might be different from the module that defines the function).
[Official unittest documentation has a detailed description](https://docs.python.org/3/library/unittest.mock.html#where-to-patch)



## Aidbox Migrations
We can use Aidbox REST Console to run SQL queries.

Reference: [Postgres JSON Functions and Operators](https://www.postgresql.org/docs/current/functions-json.html)

### Ex. 1: Deletion of an aidbox resource attribute

Updated resource example (the filter attribute to be deleted):
```yaml
id: d6fdcc6a-4513-45df-8f06-797a4727ce7f
resourceType: MedicalRule
sourceQuery:
  id: Source
  type: batch
  entry:
    - request:
        url: $snowstorm
        method: POST
      resource:
        ecl: '<< 372586001 |Hypotensive agent (substance)|'
        filter: '...'  // <- to be deleted
        resourceType: SnowstormRequest
```

Migration SQL query (for all resources of this type):
```sql
update medicalrule 
set resource = jsonb_set(
   resource, 
   '{sourceQuery,entry,0,resource}', 
   (resource #> '{sourceQuery,entry,0,resource}')- 'filter'
)
```
