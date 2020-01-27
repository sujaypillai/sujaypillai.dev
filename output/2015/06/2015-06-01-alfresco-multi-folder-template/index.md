# Alfresco Multi Folder Template Selection

If you look into the Email Form while setting an "Send Email" Action on a folder you would be wondering how we can add new email templates into the list of drop down "Use Template" as shown below -

![MyImage](/images/img6.png)

The only option without any customization is to add your templates into the folder : **Repository > Data Dictionary > Email Templates > Notify Email Templates**

#### *How does this work?*
Look for the bean 'ac-email-templates' in alfresco/WEB-INF/classes/alfresco/action-services-context.xml which has the definition as below -

{{< gist sujaypillai 657379f60bc621e12ba9 >}}

The property *searchPath* is what restricts it from fetching templates from a single folder.

#### *What we need to change?*
Let's bring in our changes so that we can have multiple folders to select the templates from. For this we will be overriding the bean as below -

{{< gist sujaypillai b49b2d53f10647dedfec >}}

The first change is converting the property *searchPath* from a String literal to a List.
{{< highlight java >}}
private List<String> searchPath = Collections.emptyList();
{{< /highlight >}}

Then update the *getAllowableValuesImpl()* method so that it parses each folder path to search for what files to be included in the dropdown.

{{< highlight java >}}
protected Map<String, String> getAllowableValuesImpl() {
    Map<String, String> result = new HashMap<String, String>(23);
    for(String path : searchPath){
        List<NodeRef> nodeRefs = searchService.selectNodes(repository.getRootHome(),path,null,this.namespaceService,false);
        NodeRef rootFolder = null;
        if (nodeRefs.size() == 0)
        {
            throw new AlfrescoRuntimeException("The path '" + searchPath + "' did not return any results.");
        }
        else
        {
            rootFolder = nodeRefs.get(0);
        }
        buildMap(result, rootFolder);
    }
    Map<String,String> sortedResult = sortByName(result);
    return sortedResult;
}
{{< /highlight >}}

I have also added an extra method *sortByName* so that the values in the dropdown are displayed in sorted order by their name.

Checkout the code for [FolderContentsParameterConstraint](https://github.com/sujaypillai/alf-tutorials/blob/master/alftutorial-repo/src/main/java/org/ootb/repo/action/constraint/FolderContentsParameterConstraint.java) for a better understanding of how it works.

I made this whole project as 2 step -

1. First is bootstrapping a template under a specified path. [v1.0](https://github.com/sujaypillai/alf-tutorials/releases/tag/v1.0)
2. Make the templates available for selection under Email Action. [v2.0](https://github.com/sujaypillai/alf-tutorials/releases/tag/v2.0)

{{% admonition note %}}
The source code for this project lives [here](https://github.com/sujaypillai/alf-tutorials)
{{% /admonition %}}