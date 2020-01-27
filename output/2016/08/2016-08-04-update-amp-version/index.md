# Downgrading AMP version in Alfresco



If you ever tried to downgrade an AMP version in Alfresco you should have seen an error something similar:

{{< highlight java >}}
Caused by: org.alfresco.error.AlfrescoRuntimeException: 02100002
  Downgrading of modules is not supported.
        Module 'acme-cms-poc-repo-amp' version 1.0.1 is currently installed and must be uninstalled before version 1.0.0 can be installed.
        at org.alfresco.error.AlfrescoRuntimeException.create(AlfrescoRuntimeException.java:51)
        at org.alfresco.repo.module.ModuleComponentHelper.startModule(ModuleComponentHelper.java:632)
        at org.alfresco.repo.module.ModuleComponentHelper.access$500(ModuleComponentHelper.java:61)
        at org.alfresco.repo.module.ModuleComponentHelper$1$1.execute(ModuleComponentHelper.java:259)
        at org.alfresco.repo.transaction.RetryingTransactionHelper.doInTransaction(RetryingTransactionHelper.java:454)
        at org.alfresco.repo.transaction.RetryingTransactionHelper.doInTransaction(RetryingTransactionHelper.java:342)
        at org.alfresco.repo.module.ModuleComponentHelper$1.doWork(ModuleComponentHelper.java:280)
        ... 45 more
{{< /highlight >}}


There are two ways to solve this issue:

1. Configuring a rule on a folder to execute script action.

2. Executing a SQL update statement directly on the DB.


### Configuring a rule on a folder to execute script action.

* Navigate to Admin Tools > Node Browser
* Change the "Select Store" drop down on the top right section to the option 'system://system'
* Change the search option from "fts-alfresco" to "storeroot" and press Search
    ![Node Browser](/images/img7.png)
* This would give the workspace reference 
    [`system://system/c7cad4eb-98a2-4904-be3c-f6c98f83ad09`]
* Navigate to **/sys:system-registry/module:modules**
* From the “Child Nodes” section find the module for which you need to update the versión and get the child reference in this case – 
     `system://system/31278e85-7fe5-49b2-a1b7-179707163da5`
     ![Node Browser](/images/img8.png)
* Copy the following script in to a text file and save it as changeAMPVersion.js and save it under ***Repository > Data Dictionary > Script***

{{< highlight javascript >}}
var node = search.findNode("<nodeRef >");
node.properties["module:currentVersion"]="1.0.0";
node.save();
{{< /highlight >}}

* Create a folder and apply a rule that executes the said script from the .js file uploaded in above step. Once executed the “module:currentVersion” of the module will reflect the required value.

{{% admonition note %}}
If you have the Javascript console installed then you may execute the above script directly on [JSConsole](https://github.com/share-extras/js-console)
{{% /admonition %}}

### Executing a SQL update statement directly on the DB.

* You may execute the below query directly on Alfresco DB to change the currentVersion value of AMP module
{{< highlight sql >}}
UPDATE alf_node_properties
LEFT JOIN (alf_qname, alf_namespace) ON (alf_node_properties.qname_id = alf_qname.id AND alf_qname.ns_id = alf_namespace.id)
SET alf_node_properties.string_value = "1.0.0"
WHERE alf_namespace.uri="http://www.alfresco.org/system/modules/1.0" AND alf_qname.local_name = "currentVersion" AND string_value = "1.0.1";
{{< /highlight >}}