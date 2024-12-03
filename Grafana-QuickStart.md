# Grafana
Displaying CS data in Grafana is relatively straightforward. Codescene APIs return easy-to-parse JSON, which generally works well, as Grafana.
## Assumptions
Cloud Grafana and Cloud Codescene
Familiarity with Codescene APIs https://codescene.io/docs/integrations/rest-api.html
## Quick Setup
1. Install Infinity Plugin for Grafana   
2. Create a data connection between Grafana and Codescene  
3. Create a Grafana dashboard and Add visualisations  
4. Configure the Grafana panel to accept and parse JSON  
5. Customise visualisation look and feel
## Detailed Setup
Grafana data setups are standard across most connections. It requires a secure data connection and, separately, a call to an API. What is returned is parsed and displayed.
1. The easiest way to parse JSON in Grafana is to use a plugin expressly for that purpose. I have found Infinity to be a powerful and flexible tool to parse json, XML, and REST APIs. It also provides a powerful UQL editor to create customised queries. More information is available here: https://grafana.com/grafana/plugins/yesoreyeram-infinity-datasource/
2. Create a Bearer Token in Codescene. A REST API token can be created under the main Configuration menu item. Add a new token and copy the token to a secure location. Choose *Read privileges*. Make sure to copy it because you cannot see it again later. Note the CodeScene project number you intend to query (visible in the Codescene URL when you click on a Project Dashboard).
3. Create a new Infinity data source in Grafana. Someone with admin rights needs to set up both CodeScene and Grafana. Name it ***CodeScene.io \- Infinity***
    1. Test the query from a command line
        1. `curl \-X GET \--header 'Accept: application/json' \--header 'Authorization: Bearer \<bearer token from step 2\>' 'https://api.codescene.io/v2/projects/\<project\_id\>/analyses/latest' | jq \-r`
Hint: If you pipe the output into jq, a command-line JSON processor `jq -r` the returned JSON is formatted for easier reading.
2. From the Grafana GUI, set up a new Infinity data source using a Bearer Token as the authentication method
    1. Copy your previously created Bearer Token to **Auth details**
        1. **Base URL**: [https://api.codescene.io](https://api.codescene.io)
        1. **Allowed hosts**: [https://api.codescene.io](https://api.codescene.io)
        1. **Health check URL:** (*Optional*) [https://api.codescene.io/v2/projects](https://api.codescene.io/v2/projects) [https://api.codescene.io/v2/projects/](https://api.codescene.io/v2/projects/)\<project\_id\>/analyses/latest
        1. Save & Test  
4. Decide internally what kinds of data you want to visualise and from which project you want to extract data. As examples, we'll use
    1. Code health Hotspot
    1. Open branches (actively used in last X months.
    1. Highest risk branches
    1. Unplanned work over time
5. Build a new Grafana Dashboard
    1. Begin by creating a blank Grafana Dashboard then add a single Visualisation to it. \+ Add visualisation
    1. You should see your newly created Data source CodeScene.io \- Infinity
    1. For Code Health (Weighted Average), we’ll use the visualisation type, *Gauge*.
    1. In **Query A**, choose:
        1. **Type**: JSON
        2. **Parser**: Backend
        3. **Source**: URL
        4. **Format**: Table
        5. **Method**: GET
        6. **URL**: [https://api.codescene.io/v2/projects/](https://api.codescene.io/v2/projects/)\<project\_id\>/analyses/latest
    1. Refresh the Dashboard panel

If that worked, then under “Parsing options & Result fields Field types, alias and selectors”, you should be able to choose *Query Inspector → Query*
Refresh and then **\+ Expand All** to see the query results. It may look something like   
```json
Object
 traceId:undefined
  request:Object
   url:"api/ds/query?ds_type=yesoreyeram-infinity-datasource&requestId=SQR123"
   method:"POST"
    data:Object
     queries:Array[1]
      0:Object
       columns:Array[1]
        0:Object
         selector:"high_level_metrics.hotspots_code_health_now_weighted_average"
         text:"Now"
         type:"number"
       datasource:Object
        type:"yesoreyeram-infinity-datasource"
        uid:"ee3m6vsl9cqv4a"
       filters:Array[0]
       format:"table"
       global_query_id:""
       parser:"backend"
       refId:"A"
       root_selector:""
       source:"url"
       type:"json"
       url:"https://api.codescene.io/v2/projects/12345/analyses/latest"
```
7. Return to *Parsing options*  
      1. Choose *Add Columns*  
        1. **Selector**: high\_level\_metrics.hotspots\_code\_health\_now\_weighted\_average  
        1. **As**: Now  
        1. **Format as**: Number  
      1. Save  
8. : Customise the Gauge visualisation (*Optional*)  
      1. In Panel Options, add a **Title** and a **Description**  
      1. In Gauge, check **Show threshold markers**  
      1. In Thresholds, **Add threshold**  
         1. Examples:   
            1. Green 9  
            1. Light Green 8  
            1. Yellow 6  
            1. Orange 4  
            1. Red 0   
