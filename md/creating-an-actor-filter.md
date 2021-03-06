# Create an actor filter

An actor filter is implemented in Bonita BPM in two parts, the definition and the implementation. This enables you to change the
implementation without changing the definition. Several implementations can be created for a single definition. See [here ](actor-filtering.md)for a list of actor filters provided with the product.

An actor filter is a type of connector and is created in the same way, using the same schema files as for a connector. You can define a new actor filter definition or implementation in Bonita BPM Studio, using the wizards started from the **Development** menu, **Actor filters** sub menu. You can also create a new actor filter definition or implementation externally, and import it into Bonita BPM Studio.

This page explains how to create the actor filter definition and implementation and import it into Bonita BPM Studio. 

## Actor filter definition

An actor filter definition controls the external interfaces of the actor filter, both those visible to users (the actor filter configuration wizard) and those visible to the Bonita BPM Engine (the inputs and outputs). The actor filter definition consists of the following files:

* An XML file that defines the configuration wizard
* A file containing the image that is used as an icon to represent the actor filter
* A properties file for each language that the wizard supports

The following sections explain the elements of the definition. You can also use Bonita BPM Studio to create the definition.

### Actor filter configuration wizard definition

The XML file that defines the actor filter configuration wizard contains definitions for header information identifying the actor filter (including the icon), for inputs and outputs, and for the wizard pages. The definition follows the schema defined in http://documentation.bonitasoft.com/ns/connector/definition/6.0/connector-definition-descriptor.xsd. The table below lists the items in the definition:
| | Occurrence | Description|
|:-|:-|:-|
| id | 1 | The identifier of the actor filter definition. Used internally, not displayed in the wizard or in Bonita BPM Studio. Must be unique. |
| version | 1 | The version of the actor filter definition. Must be unique for the actor filter identifier. |
| icon | 0 or 1 | The name of a file containing the icon image. |
| category | 1 | The category to which the actor filter belongs. In Bonita BPM Studio, actor filters are grouped by category. Optionally, the category can have an icon image. |
| input | 0 or more | Input data passed to the actor filter. Each input has a unique name and a Java type. Optionally, it can be defined as mandatory. Optionally, a default value can be set. |
| output | 0 or more | Output data passed to the process from the actor filter.  Each output has a unique name and a Java type. |
| page | 0 or more | A page in the wizard. A page must have a unique id. A page consists of one or more widgets. A page is constructed by displaying the widgets in the order in which the definitions appear in the file. |
| widget | 1 or more per page | An element within a page, corresponding to an input. A widget must have a unique id, an input name, and a type. The following types of widget are available: text, password, text area, check box, radio button group, select (drop-down list), array, group, script editor, and list. |
| jarDependency | 0 or more | A dependency that must be satisfied for the wizard to run successfully. |

The page and widget definitions are required for an actor filter that is used or configured from within Bonita BPM Studio. 

The following example is the XML definition for the _initiator manager_ actor filter:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<definition:ConnectorDefinition xmlns:definition="http://www.bonitasoft.org/ns/connector/definition/6.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
     <id>initiator-manager</id>
     <version>1.0.0</version>
     <icon>initiator-manager.png</icon>

         <category id="process" icon="process.gif" ></category>

         <input name="autoAssign" type="java.lang.Boolean" mandatory="false" defaultValue="true" />

         <page id="config">
                <widget id="autoAssign" inputName="autoAssign" xsi:type="definition:Checkbox" ></widget>
        </page>
</definition:ConnectorDefinition>
```

In this example, there is one input, a Boolean that determines whether or not the task is automatically assigned to the user identified by the filter. There is one page in the configuration wizard, which contains one widget.

### Language properties file

The text displayed in a wizard is defined separately from the pages and widgets. This means that one actor filter definition can support multiple languages.

For each language, there is a properties file that contains the category name, the actor filter name, the actor filter definition, the name and description of each wizard page, and the name and description of each input widget. 

The default language properties file is called _actorfilter-id\_version_.properties, where _actorfilter-id_ is the actor filter identifier and _version_ is the actor filter version. You can provide additional properties files to support other languages. These files must be called _actorfilter-id\_lang\_version_.properties, where _lang_ identifies the language of the properties file. The language must be supported by Bonita BPM Studio. For example, the French properties file for the initiator manager shown above is called initiator-manager-1.0.0\_fr.properties. When a Bonita BPM Studio user launches an actor filter configuration wizard, the wizard is displayed in the language currently configured for Bonita BPM Studio. If there is no properties file for this language, the default file is used. 

The following example is the English language properties file for the _initiator manager_ actor filter:
```properties
process.category = Process
connectorDefinitionLabel = Initiator manager
connectorDefinitionDescription = Manager of the initiator of the process
config.pageTitle = Configuration
config.pageDescription = Configuration of the process initiator-manager filter
connexionConfigPage.pageDescription = Choose here if you want to automatically assign the task to this actor
autoAssign.label = Assign task automatically
autoAssign.description = The task will be claimed automatically by the resolved user 
```

## Actor filter implementation

An actor filter implementation consists of an XML resource file and a Java
class. You can create any number of implementations that correspond to a given definition. 
However, in a process there is a one-to-one relationship between the actor filter definition and the actor filter implementation.

### Actor filter implementation resource file

The resource file defines:

* the id and version of the definition that is implemented
* the id and version of the implementation
* the set of dependencies required by the implementation. 

The resource file follows the schema defined in http://documentation.bonitasoft.com/ns/connector/definition/6.0/connector-implementation-descriptor.xsd .

The following example is the resource file of an implementation of the
initiator manager actor filter:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<implementation:connectorImplementation xmlns:implementation="http://www.bonitasoft.org/ns/connector/implementation/6.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

         <definitionId>initiator-manager</definitionId>
         <definitionVersion>1.0.0</definitionVersion>
         <implementationClassname>org.bonitasoft.userfilter.initiator.manager.ProcessinitiatorManagerUserFilter</implementationClassname>
         <implementationId>initiator-manager-impl</implementationId>
         <implementationVersion>1.0.0</implementationVersion>

         <jarDependencies>
             <jarDependency>bonita-userfilter-initiator-manager-impl-1.0.0-SNAPSHOT.jar</jarDependency>
        </jarDependencies>

</implementation:connectorImplementation>
```

### Actor filter implementation Java class

The Java class must implement the org.bonitasoft.engine.filter.AbstractUserFilterclass and use the Engine ExecutionContext. The following methods must be implemented:
* validateInputParameters to check that the configuration of the actor filter is well defined
* filter to get a list of identifiers of all the users that correspond to a specified actor name
* shouldAutoAssignTaskIfSingleResult to assign the step to the user if filter returns one user

For details of the APIs, the methods and related objects, see the [Javadoc](http://documentation.bonitasoft.com/javadoc/api/${varVersion}/index.html).

### Actor filter example code

The following code is an example of the initiator manager actor filter. 
```groovy
public class ProcessinitiatorManagerUserFilter extends AbstractUserFilter {

    @Override
    public void validateInputParameters() throws ConnectorValidationException {
    }

    @Override
    public List<Long> filter(final String actorName) throws UserFilterException {
        try {
              final long processInstanceId = getExecutionContext().getParentProcessInstanceId();
              long processInitiator = getAPIAccessor().getProcessAPI().getProcessInstance(processInstanceId).getStartedBy();
              return Arrays.asList( getAPIAccessor().getIdentityAPI().getUser(processInitiator).getManagerUserId());
        } catch (final BonitaException e) {
            throw new UserFilterException(e);
        }
    }

    @Override
    public boolean shouldAutoAssignTaskIfSingleResult() {
        final Boolean autoAssignO = (Boolean) getInputParameter("autoAssign");
        return autoAssignO == null ? true : autoAssignO;
    }

} 
```

## Testing an actor filter

There are three stages to testing an actor filter:

1. Build the actor filter. If you are using Maven, create two projects, one
for the definition and one for the implementation. Build the artifacts for
import into Bonita BPM Studio, using the following command: 

mvn clean install

This creates a zip file.
2. Import the actor filter into Bonita BPM Studio. From the
**Development** menu, choose **Actor filters**,
then choose **Import...**. Select the zip file to be
imported.
3. Test the actor filter in a process. Create a minimal process and add the actor
filter to a step. Configure the process and run it from Bonita BPM Studio.
Check the Engine log (available through the **Help** menu) for
any error messages caused by the actor filter.

## Importing an actor filter into Bonita BPM Studio

1. Create a zip file that contains the files used by the definition and implementation.
2. In Bonita BPM Studio, go to the **Development** menu, **Actor filters**, **Import actor filter...**.
3. Upload the zip file.

The imported actor filter is now available in the dialog for adding an actor filter.

It is also possible to export an actor filter using options in the **Development** menu. The actor is exported as a .zip file, which you can import into another
instance of Bonita BPM Studio.

## Configuring and deploying a process with an actor filter 

When you configure a process that uses an actor filter in Bonita BPM Studio, you
specify the definition and implementation. You must also specify any
dependencies as process dependencies. After the actor filter has been specified
in the configuration, when you build the process for deployment referencing the
configuration, the actor filter code is included in the business archive.
