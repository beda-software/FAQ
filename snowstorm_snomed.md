# Import SNOMED CT with NZ extensions to Snowstorm terminology server
## Download SNOMED
Just to look:
[www.health.govt.nz](https://www.health.govt.nz/nz-health-statistics/classification-and-terminology/new-zealand-snomed-ct-national-release-centre/snomed-ct-subsets-and-maps)
A site where we can find all files we need (authorization required):
[https://mlds.ihtsdotools.org/#/dashboard](https://mlds.ihtsdotools.org/#/dashboard?token=e609654e-847a-4f7b-a608-934ea44b1f5d)

List of files to download:
* SnomedCT_InternationalRF2_PRODUCTION_20200309T120000Z.zip
* SnomedCT_ManagedServiceNZ_PRODUCTION_NZ1000210_20200401T000000Z.zip
* SnomedCT_GPFP_PRODUCTION_20191030T120000Z.zip
* SnomedCT_GPFPICPC2_PRODUCTION_20190412T120000Z.zip

The GPFP Refset is represented in both archives listed above.

## Prepare archives
I’m not sure that it is necessary, but if separate imports don’t give a result, you can unpack all archives, collect all files from SNAPSHOT folders to one folder saving its directory structure and zip it.

## Deploy Snowstorm
Clone a repository
https://github.com/IHTSDO/snowstorm/

In docker-compose.yml change public port for snowstorm container to avoid conflicts:
```yaml
snowstorm:
image: snomedinternational/snowstorm:latest

container_name: snowstorm

depends_on:
elasticsearch:
condition: service_healthy

entrypoint: java -Xms2g -Xmx4g -jar snowstorm.jar --elasticsearch.urls=http://es:9200

networks:
- elastic

ports:
- 8085:8080
```

Assure that your docker app has enough resources (in other case elastic search won’t start and you will see an error)

Snowstorm creates all ES indexes after starting.

## Adjust Elasticsearch to avoid import errors
Possible import error:
`the number of terms [422770] used in the Terms Query request has exceeded the allowed maximum`

To avoid the error while importing files execute this request against elastic:
```
PUT http://localhost:9200/_settings


{
    "index": {
        "max_terms_count": 1000000
    }
}
```

List existing indexes by executing `http://localhost:9200/_cat/indices`
and check if the setting has been updated with `http://localhost:9200/_settings`

## Import SNOMED archive 
(according to [instructions](https://github.com/IHTSDO/snowstorm/blob/master/docs/loading-snomed.md))

1. Create an import job
```
{
  "branchPath": "MAIN",
  "createCodeSystemVersion": true,
  "type": "SNAPSHOT"
}
```
2. Copy import id
3. Upload the archive - The process can take more than one hour
4. Assure that import job is COMPLETED

5. Check the result using [http://localhost:8085/fhir/ValueSet/$expand?url=http://snomed.info/sct?fhir_vs=ecl/%5E450970008%20AND%20%3C%3C64572001](http://localhost:8085/fhir/ValueSet/$expand?url=http://snomed.info/sct?fhir_vs=ecl/%5E450970008%20AND%20%3C%3C64572001)
as example 
