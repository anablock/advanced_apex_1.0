# Advanced Apex Programming 4th Edition

## Dev, Build and Test

Refer to the Salesforce documentation for Salesforce DX.
This is a Salesforce DX project.
Chapters are located in individual branches in the git repository. Steps within a chapter are represented by individual commits

## Resources

Refer to the license or license.txt file for copyright and license information.

## Description of Files and Directories

Refer to Salesforce DX and Salesforce metadata documetnation for a description of
all files.
For an in-depth explanation of this project, refer to the 4th edition of the book Advanced Apex Programming

## Issues


## =============================================
* triggers can receive up to 200 objects at once
* SOQL queries and DML statements should not be inside of loops
* Apex code should be designed to handle bulk operations

* Why test?
    * Does the code work
    * Code coverage
    * Handling invalid input

Unit tests are unique in that they have two sets of governor limits available: 
    * one set used for setting up data and verifying results, 
    * onse set for the test itself (the code between a call to the `Test.StartTest` and `Test.StopTest` methods.

One common design pattern is to create utility functions that can be shared by multiple test classes or methods.

## Example on how to use utility functions
in the following example, there are two utility functions.  The first one, `TestBulkPatterns.InitTestObjects`, sets up test scenarios based on some parameters.  The `newopportunities` parameter references a list that is initialized by this function with the new opportunities.  `NumberOfOpportunities` specifies the number of new opportunities to create.  `NumberOfOtherOpportunities` specifies the number of additional opps to create - opps that will be associated with the contacts, but will not be updated by the test code.  `ContactRolesPerOp` specifies the number of contacts to be associated with each opp.  `NumberOfContacts` specifies the number of contacts to distribute among the opps, and is required to be at larger or equal to `ContactRolesPerOp`.

```apex
function
    try 
        Test condition
        If fail return

        continue operation
    catch
        rethrow erro
    finally
        early return statement and any exceptions
        will all execute here
```

## Triggers
* non-deterministic - 

### Managing the Data Updates
Two rules to always keep in mind for insert and update triggers:
* Updating fields on an object is a before trigger - fields that are udpatable are updated.
* Updating fields on an object in an after trigger requires a DML operation and cannot be performed on the objects provided by the trigger.  Those objects, found in the new, old, newMap and oldMap trigger context variables, have all their fields set to read-only.

To facilitate combining DML operations from multiple classes, use centralized data handler.

#### Centralized Design Handler - simple option
`TriggerDMLSupport` class - class has two static variables
* `updatePendingObjects` variable is a Boolean that indicates to the system that an internal DML operation is currently in progress.  This can be used by the various triggers to modify their behavior - for example, to skip trigger handling that is either unnecassary, or might spawn off additional DML operations.  It is also used within the class to ensure that a DML operation does not invoke another one, even if the trigger framework tries to do so.
* `opsToUpdate` static variable is used by other classes to keep track of records that need to be updated.  In this example, the class only handles opportunity objects.  

```java
// Basic data management class
public virtual class TriggerDMLSupport {
    public static Boolean updatingPendingObjects = false;
    public static Map<ID, Opportunity> opsToUpdate = new Map<ID, Opportunity>();

    // Return a map of updatable opportunity records to use in after triggers
    public static Map<ID, Opportunity> getUpdatableOpportunities(Set<ID> opsIds) {
        Map<ID, Opportunity> ops = new Map<ID, Opportunity>();
        for(ID opid: opIDs) {
            ops.put(opid, new Opportunity(ID = opid));
        }
        return ops;
    }

    public static void updatePendingOpportunities() {
        if(!updatingPendingObjects){
            while(opsToUpdate.size()>0){
                List<Opportunity> updatingList = opsToUpdate.value();
                opsToUpdate = new Map<ID, Opportunity>();
                updatingPendingObjects = true;
                update updatingList;
                updatingPendingObjects = false;
            }
        }
    }
}
```
* `getUpdatableOpportunities` method creates a map containing empty opportunity objects for each ID.  The various trigger handler classes can then set field values for these objects for later update.

* `updatePendingOpportunities` function is called at the end of the trigger.   It does not fire if called while currently updating objects - this helps reduce the chance of reentrancy.  It continues to loop while there are any objects to update.  

### Implementing tightly controlled execution flow within a trigger
The code should be designed so that it is easy to add additional functionality to the trigger later - anywhere in the sequence of operations.

```java
trigger OnOpportunity on Opportunity (before insert, after insert, after update) {
    if(TriggerDMLSupport.updatingPendingObjects) return;
    Map<ID, Opportunity> updateableMap;
    if(trigger.isAfter) updateableMap = TriggerDMLSupport.getUpdatableOpportunities(trigger.newMap.keyset());

}

```
The first thing trigger does is check to see if the trigger was caused by an internal DML operation, in other words, are we currently in process of updating opps.

The trigger uses the TriggerDMLSupport class to create a map of updateable objects for use by the after triggers, passing it as a parameter to each class.  The two trigger handler classes are called, followed by the `TriggerDMLSupport.updatePendingOpportunities` method to perform the database operation on behalf of both classes.

## Maps
* Alwaya access map elements by key
* A map key can hold the `null` value
* Map keys of type String are case sensitive
* Apex is a statically typed language.
* Statically typed means that users must specify the data type for a variable before that variable can be used.
#


```java
Account[] aa = [SELECT Id, Name FROM Account WHERE Name ='Acme'];
Integer i = [SELECT COUNT() FROM Contact WHERE LastName ='Weissman']; List<List<SObject>> searchList = [FIND 'map*' IN ALL FIELDS RETURNING Account (Id, Name),
Contact, Opportunity, Lead];
```

A static or instance method invocation:
```java
System.assert(true)
myRenamingClass.replaceNames()
changePoint(new Point(x,y))
```

## Domain class template
`FinancialForce.com Apex Enterprise Patterns` library has base class `fflib_SObjectDomain`.
Domain class utilizing this base class is shown here:
```java
public class Races extends fflib_SObjectDomain 
{
    public Races(List<Race__c> races)
    {
        super(races);
    }
}
```