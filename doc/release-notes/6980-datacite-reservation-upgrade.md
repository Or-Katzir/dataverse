## Dataverse installations using DataCite: upgrade action recommended.

For installations that are using DataCite, Dataverse v5.0 introduces a change in the process of registering the Persistent Identifier (DOI) for a dataset. Instead of registering it when the dataset is published for the first time, Dataverse will try to "reserve" the DOI when it's created (by registering it as a "draft", using DataCite terminology). When the user publishes the dataset, the DOI will be publicized as well (by switching the registration status to "findable"). This approach makes the process of publishing datasets simpler and less error-prone. 

New APIs have been provided for finding any unreserved DataCite-issued DOIs in your Dataverse, and for reserving them (see below). While not required - the user can still attempt to publish a dataset with an unreserved DOI - having all the identifiers reserved ahead of time is recommended. If you are upgrading an installation that uses DataCite, we specifically recommend that you reserve the DOIs for all your pre-existing unpublished drafts as soon as Dataverse v5.0 is deployed, since none of them were registered at create time. This can be done using the following API calls:  

`/api/pids/unreserved`  will report the ids of the datasets 

`/api/pids/:persistentId/reserve` reserves the assigned DOI with DataCite (will need to be run on every id reported by the the first API). 
(See the Native API guide for more information).

Scripted, the whole process would look as follows (adjust as needed):

```
   API_TOKEN='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
   
   curl -s -H "X-Dataverse-key:$API_TOKEN" http://localhost:8080/api/pids/unreserved |
   # the API outputs JSON; note the use of jq to parse it:
   jq '.data.count[].pid' | tr -d '"' | 
   while read doi
   do
      curl -s -H "X-Dataverse-key:$API_TOKEN" -X POST http://localhost:8080/api/pids/:persistentId/reserve?persistentId=$doi
   done
``` 

Going forward, once all the DOIs have been reserved for the legacy drafts, you may still get an occasional dataset with an unreserved identifier. DataCite service instability would be a potential cause. There is no reason to expect that to happen often, but it is not impossible. You may consider running the script above (perhaps with some extra diagnostics added) regularly, from a cron job or otherwise, to address this preemptively. 
