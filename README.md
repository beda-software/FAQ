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

#### Always user "token" type for SearchParameter instead of "string"
Mistakenly we use "string" as SearchParameter.type for some attributes with type "text", but it leads to serious issues, because FHIR search works as full-text search with substring partial matching. Example: SearchParameter User.email must have "token" type

#### Try to avoid subscription in favor of the custom operation
If you need to mutate the original resource before/after save - use the custom operation to do it. It has some benefits:
* Version id won't be increased twice
* Easier to test
* You can make transactional queries: all or nothing

#### Don't use the display in references
Using `display` in references is a bad idea because it is denormalization and the original title can be changed so that you will need to actualize it. Actualization is a very difficult process and has some drawbacks:
* It changes version ID --- our UI usually can't handle version mismatch
* It takes time - you need to go through the database and find all references
* It can't work with other subscriptions (because actualization is a best practice for subscriptions)

If you don't want to have a headache with these problems, consider using `_include` every time to get the actual value.

#### User resource role in the system
User resources must contain only credentials and links to appropriate Patient/Practitioner. 

Practitioners can be used for admin roles and other roles (see FHIR spec for details). 
Using references to `User` should be forbidden because makes a potential vulnerability (because of _include/_assoc that is not forbidden by default). Instead of this, always make references to appropriate Patient/Practitioner.

#### Access policy rules
They always should have `additionalProperties: false` for search params to forbid passing `_assoc/_include/_revinclude`.


#### Avoid using access_policy argument in the custom operation
Using `access_policy` argument in the customer operation makes it difficult to maintain policies for different roles


#### Generate future datetime ranges with DST/STD time shifting
In many countries there is a Daylight saving time shifting. While generating future time ranges it is necessary to consider such shiftings.

In Israel, DST->STD in 2020 is occuring on October 25.
For example, there is a task to generate a datetime range from October 20 for 7 days with the 1 day step.

This code will prevent to an error starting from October 25, because `datetime.timedelta` doesn't consider the DST->STD time shift:

```
while current_date < end_date:
  # datetime.timedelta doesn't consider time shifting
  current_date += datetime.timedelta(days=1)
```

To fix this use DST shift adding/subtraction:

```
next_date = current_date + datetime.timedelta(days=1)

# Normalize datetime to the local timezone considering DST/STD
next_date = next_date.tzinfo.normalize(next_date)

# Add/subtract DST (3600 seconds) /STD (0 seconds) time shift
next_date = next_date + (current.dst() - next_date.dst())
```

Do not forget to cover this logic with tests by checking that all time values (hours, minutes, etc) in your range are equal and the range parts quantity is correct

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

## Aidbox Resources

### How to add an object attribute with Any content to an Aidbox resource

```
"Condition.property": {
    "path": ["property"],
    "resource": {"resourceType": "Entity", "id": "Condition"},
    "extensionUrl": "http://beda.software/fhir-extensions/condition-property",
    "schema": {"type": "object"},
},
```
