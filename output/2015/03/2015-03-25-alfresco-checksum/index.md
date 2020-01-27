# Generating Checksums for Alfresco Content

### Inspired from Chris Paul's [blog](http://blog.productivist.com/2011/11/21/generate-checksums-for-alfresco-content/) and [work](https://github.com/cmpaul/hashable)

I was reading about [Alfresco Global Virtual Hack-a-thon](https://wiki.alfresco.com/wiki/Projects_and_Teams_Global_Virtual_Hack-a-thon_2015) scheduled to be on 17th April 2015 and going through the list of Projects came across - *Document Checksuming and display in Share.* My bookmark's collection had Chris's blog and thought of bootstrapping myself for hackathon by rewriting the whole work done by Chris using Alfresco SDK 2.0 on 5.0.c.

Below are the brief details about my approach:

* Create an empty directory which will hold both the repository tier & share amp archetypes.

* Navigate into the above created folder and issue the below command: 

{{< highlight java >}}
    mvn archetype:generate -Dfilter=org.alfresco.maven.archetype:
{{< /highlight >}}

* You will be prompted to choose an archetype
    
 ![MyImage]({{ site.baseURL }}/img/img1.png)

* Enter **2** to create an AMP project for repository tier.
* For org.alfresco.maven.archetype:alfresco-amp-archetype version: choose value as **5** for selecting an archetype version of **2.0.0**.
* Enter value for property 'groupId' as **org.ootb**
* Enter value for property 'artifactId' as **chksum-repo**
* The next step would confirm with all the properties set by you and if you are satisfied with it enter **Y** to create the project.
* Repeat the above procedure for creating a share amp project.
* Once you have both the projects created import them into eclipse.
 ![Eclipse Project Structure after import]({{ site.baseurl }}/img/img2.png)

* On the **chksum-repo** (repository amp) side I haven't made any changes except that the below class where the name has been changed from HashableModel to ChecksumModel and the namespace url and prefix referring to [OrderOfTheBee](http://orderofthebee.org/)

{{< highlight java >}}
package org.ootb.model;
import org.alfresco.service.namespace.QName;

public interface ChecksumModel {
    static final String CHKSUM_MODEL_URI    = "http://orderofthebee.org/model/checksum/1.0";
    static final String CHKSUM_MODEL_PREFIX = "ootb";
    
    static final QName ASPECT_HASHABLE = QName.createQName(CHKSUM_MODEL_URI, "hashable");
    static final QName PROP_HASHTYPE   = QName.createQName(CHKSUM_MODEL_URI, "hashType");
    static final QName PROP_HASHVALUE  = QName.createQName(CHKSUM_MODEL_URI, "hashValue");
}
{{< /highlight >}}

* On the Share side to make the aspect visible to the user add the below part in share-config-custom.xml
{{< highlight java >}}
<config evaluator="string-compare" condition="DocumentLibrary" replace="true">
 <aspects>
     <!-- Aspects that a user can see -->
     <visible>
        <aspect name="ootb:hashable" />
     </visible>
 </aspects>
</config>
{{< /highlight >}}

This should make the Hashable aspect visible in the list of aspects available to add when the user clicks on "Manage Aspects" from document-actions.
![Manage Aspects]({{ site.baseurl }}/img/img3.png)

* By default in the above screen it would display the aspect name as ***aspect.ootb_hashable*** and to add a meaningful value to it add the following bean:
{{< highlight java >}}
<bean id="ootb.custom.resources" class="org.springframework.extensions.surf.util.ResourceBundleBootstrapComponent">
  <property name="resourceBundles">
     <list>
        <value>alfresco.module.${project.artifactId}.messages.chksum</value>
     </list>
  </property>
</bean>
{{< /highlight >}}

* In chksum.properties file add the below line:
{{< highlight java >}}
    aspect.ootb_hashable=Hashable
{{< /highlight >}}

* Once you add the Hashable aspect to a document the policy handler would calculate the checksum ***(ootb:hashValue)***  by default using ***(ootb:hashType)*** as **MD5** as shown below:
![Checksum value]({{ site.baseurl }}/img/img4.png)

* To display the checksum value on properties page you need to add the following in share-config-custom.xml:
{{< highlight java >}}
<!-- Display the Hashable properties -->
<config evaluator="node-type" condition="cm:content">
    <forms>
        <form>
            <field-visibility>
                <show id="ootb:hashType" />
                <show id="ootb:hashValue" />
            </field-visibility>
            <appearance>
                <field id="ootb:hashValue" read-only="true" />
            </appearance>
        </form>
    </forms>
</config>
{{< /highlight >}}

#### Status Indicator: 

* Defined within the **DocumentLibrary** config section, status indicators are small icons typically used to indicate the presence of an aspect.
* Indicator images are by default reference by id: ***/res/components/documentlibrary/indicators/{id}-16.png*** unless the name is overridden by the "icon" attribute.
* As shown below any content which has an aspect ***oot:hashable*** applied to it (has a checksum value calculated) then it shows the icon "H"
![Status indicator]({{ site.baseurl }}/images/img5.png)
* This is achieved by adding the following code snippet in share-config-custom.xml:
{{< highlight xml >}}
<indicators>
    <indicator id="hashable" index="10" icon="hashable-16.png">
        <evaluator>evaluator.doclib.indicator.hashable</evaluator>
    </indicator>
</indicators>
{{< /highlight >}}

* The evaluator ***evaluator.doclib.indicator.hashable*** helps to identify whether a node has an aspect ***oot:hashable*** and is defined in the bean
{{< highlight xml >}}
<bean id="evaluator.doclib.indicator.hashable" class="org.ootb.web.evaluator.doclib.indicator.ChecksumEvaluator">
</bean>
{{< /highlight >}}

##### The complete source code for this example is available on [GitHub](https://github.com/sujaypillai/alfchecksum).
