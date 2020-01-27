---
layout: post
title: Aikau Audit Report
date: 2015-09-18T16:22:42+08:00
lastmod: 2015-09-18T16:22:42+08:00
draft: false
tags: ['alfresco','aikau']
categories: ['Alfresco']
show_in_homepage: false
show_description: false
---


[EisenVault](https://eisenvault.com) takes away the hassle of managing documents by storing them in both electronic and physical forms, enabling smart-search and deploying quick retrieval measures. Our innovative cloud-based document management system can be used in multiple industry sectors, and caters to specific requirements of various job roles. Our vision - to be your preferred partner for electronic and physical document management.

This is [EisenVault's](https://eisenvault.com) first Aikau plugin and we would like to share our experience building it. While during the days when people used Alfresco Explorer there was an option to view the audit trail for a document using the show_audit.ftl by selecting ***"Preview in Template"***.
Below is a screenshot of how it looked - 

![Alfresco Explorer Audit Trail](/images/audit-example-explorer.jpg)

But there is no such option in Alfresco Share to view those audit data, so I thought of building it. Auditing is by default enabled in Alfresco but the data generation for the same is disabled and to enable this you would have to put in the below property in alfresco-global.properties.
{{< highlight properties >}}
audit.alfresco-access.enabled=true
{{< /highlight >}}
You can check the status for auditing by issuing the command:

{{< highlight console >}}
curl -u admin:password "http://localhost:8080/alfresco/service/api/audit/control"
{{< /highlight >}}

On issuing the above command you would get a JSON response something similar -
{{< highlight json >}}
    {
       "enabled" : true,
       "applications": 
       [
          {
             "name": "Alfresco Tagging Service",
             "path" : "/tagging",
             "enabled" : true
          }
             ,
          {
             "name": "alfresco-access",
             "path" : "/alfresco-access",
             "enabled" : true
          }
             
       ]
}
{{< /highlight >}}

You can also query to retrieve the audit events:
{{< highlight console >}}
curl -u admin:password "http://localhost:8080/alfresco/service/api/audit/query/alfresco-access?limit=2"
{{< /highlight >}}
{{< highlight json >}}
{
   "count":2,
   "entries": 
   [
      {
         "id":1,
         "application":"alfresco-access",
         "user":"admin",
         "time":"2015-09-16T13:48:08.796+08:00",
         "values": null
      },
      {
         "id":2,
         "application":"alfresco-access",
         "user":"admin",
         "time":"2015-09-16T13:48:09.560+08:00",
         "values": null
      }
   ]
}
{{< /highlight >}}
> ***If you looking for more information about Alfresco Audting you should definitely look [here](http://docs.alfresco.com/5.0/concepts/audit-intro.html)***


### Aikau Audit Report page
* The new Aikau Audit report page lets you view the audit trail for each document in the repository as shown below:

    ![Aikau Audit Report](/images/aikau-audit-report.png)

* You can access this page either from the document library action OR from the Document Actions section from the document details page:

    ![DL](/images/aar-dl.png) |   ![DA](/images/aar-da.png)


### How does it work?
* The [AlfList](http://dev.alfresco.com/resource/docs/aikau-jsdoc/AlfList.html) widget makes a call to webscript [NodeAuditTrailWebScript](https://github.com/vipulswarup/ev-aikau-auditreport/blob/master/ev-repo/src/main/java/com/eisenvault/webscript/NodeAuditTrailWebScript.java) with help of the attribute ***loadDataPublishTopic*** via the CrudService (using the "**ALF_CRUD_GET_ALL**" topic) which take the nodeRef as a parameter and this in turn returns the data in JSON format.
* Now to render the list what AlfList loads we need to create a view which is done with the help of [AlfListView](http://dev.alfresco.com/resource/docs/aikau-jsdoc/AlfListView.html).
* As the list contains five columns we would be adding same number of [HeaderCell](http://dev.alfresco.com/resource/docs/aikau-jsdoc/HeaderCell.html) for the attribute ***widgetsForHeader***.
* Each row in the data will be represented by [Row](http://dev.alfresco.com/resource/docs/aikau-jsdoc/Row.html) and each [Cell](http://dev.alfresco.com/resource/docs/aikau-jsdoc/Cell.html) will hold [Property](http://dev.alfresco.com/resource/docs/aikau-jsdoc/Property.html) to render the value for respective ***Cell***.
* As the last column "**Audit Entry Values**" holds multiple values the ***Property*** widget need to be overridden and you may find this in [Property.js](https://github.com/vipulswarup/ev-aikau-auditreport/blob/master/ev-share/src/main/amp/web/js/eisenvault/widgets/Property.js)


{{% admonition note %}}
You may find the complete source code for this addon on [Github](https://github.com/vipulswarup/ev-aikau-auditreport)
{{% /admonition %}}