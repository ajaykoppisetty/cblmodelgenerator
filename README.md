## BACKGROUND

This is a command line interface (CLI) to generate Couchbase Lite models from a Core Data model file, i.e. ```*.xcdatamodeld```.

## INSTALLATION

1.  Clone the repository to your local drive
2.  Select Archive from the ```Product``` menu in XCode.
3.  Right-click on the cblmodelgenerator archive in the Organizer and click "Show in Finder"
4.  Right-click on cblmodelgenerator*.xcarchive in Finder and click "Show Package Contents"
5.  Under ```Products/usr/local/bin```, locate cblmodelgenerator and copy this file to any directory in your terminal path, e.g. ```/usr/local/bin```

## USAGE

```cblmodelgenerator /path/to/xcdatamodeld [/path/to/modeloutputdirectory]```

Make sure your Core Data Model ```.xcdatamodeld``` is NOT included in any targets for your app as the file will be invalid in many cases.

## Core Data Model File Rules

You must adhere to these rules in order for the cblmodelgenerator to correctly generate header/source files for you. Since NoSQL supports more types of data than CoreData, we utilize specific conventions to direct the generator to create the desired syntax.

JSON-compatible objects include ```NSString``` and ```NSNumber```. Non-JSON compatible objects include ```NSData```, ```NSDate```, and ```NSDecimalNumber```.

### Required classes in your model

Starting with a blank Core Data model file, you must create an Entity named ```CBLModel```. It will not be generated in code however since CouchbaseLite already has this. Any subsequent CBLModel entities you create MUST set the ```CBLModel``` entity as its parent.

If you decide to use my ```CBLNestedModel``` class (from my fork of couchbase-lite-ios), then you must also create an entity named ```CBLNestedModel``` and set it as the parent for any entities that should derive from that class. Similar to ```CBLModel``, this entity will also NOT be generated when you run the CLI since it is included with the CouchbaseLite framework.

Essentially, all your entities must have a parent, either CBLModel, CBLNestedModel, or derivative entities that you create yourself.

### Attribute Type Conversions

Use attribute types for normal properties that store primitives and generic foundation objects.

| Core Data Attribute Type | Property Attribute Type |
|--------------------------|:-----------------------:|
| Integer 16/32/64         |         int             |
| Boolean                  |         bool            |
| Float                    |         float           |
| Double                   |         double          |
| Decimal                  |         NSDecimalNumber |
| String                   |         NSString        |
| Date                     |         NSDate          |
| Binary Data              |         NSData          |
| Transformable            |         *Object*        |

Transformable attributes should be thought of as "relationships" to other objects not represented in your CoreData model file. For example, you may want an NSArray/NSDictionary of strings (JSON-compatible), or dates (non-JSON compatible). If you select Transformable, you must specify whether it is an NSArray or NSDictionary in the second ```Attribute Type``` name field. If it stores non-JSON compatible objects, then you must additionally specify the following \<key=itemClass,value\> under User Info for the attribute.

**Desired Output:** ```@property (nonatomic, strong) NSArray* object;```       *// An array of JSON-compatible objects*

- Attribute Name: ```object```
- Attribute Type: ```Transformable```
- Attribute Type Name: ```NSArray```
- User Info \<key,value\>: ```empty``` if JSON-compatible; ```<itemClass, ClassName>``` if not JSON-compatible

**Desired Output:** ```@property (nonatomic, strong) NSDictionary* object;```   *// A dictionary of JSON-compatible objects*

- Attribute Name: ```object```
- Attribute Type: ```Transformable```
- Attribute Type Name: ```NSDictionary```
- User Info \<key,value\>: ```empty``` if JSON-compatible; ```<itemClass, ClassName>``` if not JSON-compatible


### Relationships

Create new relationships anytime you want to link a model/nestedmodel to another model, or to make arrays/dictionaries.  ```ModelObject``` refers to another entity in your CoreData model. The following examples illustrate all the ways to use relationships:

**Desired Output:** ```@property (nonatomic, strong) ModelObject* object;```

- Relationship Name: ```object```
- Relationship Type: ```To One```

**Desired Output:** ```@property (nonatomic, strong) NSArray* object;```        *// An array of ModelObjects*

- Relationship Name: ```object```
- Relationship Type: ```To Many```
- Relationship Arrangement: ```Ordered [checked]```
 
**Desired Output:** ```@property (nonatomic, strong) NSDictionary* object;```   *// A dictionary of ModelObjects*

- Relationship Name: ```object```
- Relationship Type: ```To Many```
- Relationship Arrangement: ```Ordered [UNchecked]```

## XCode Project Settings

In CouchbaseLite, you may find yourself setting up many one-way relationships since you can always use views to filter data other ways. Keeping one-way relationships significantly improves readability of your Core Data model, but Core Data itself does not recommend using these. In order to prevent users from setting up one-way relationships, XCode will throw warnings notifying the developer of one-way relationships in the Core Data Model file.

Since we are not using the underlying CoreData framework, we recommend not adding your model file to your target build and turning off warnings for one-way relationships. You can do this by following the steps below:

- Click your ```*.xcdatamodeld``` file in your Project Navigator, and then, in the File Inspector on the right (icon that looks like a document), UNCHECK everything under ```Target Membership```. This removes the data model from being built or included with our app.
- Click on your Project in the Project Navigator to display the settings, and make sure that the dropdown selects your Project, and not any of your Targets (By default, it selects your Target, so you'll most likely need to change this. It is the top left hand corner dropdown of your Settings page).
- Click on ```Build Settings``` and search for ```MOMC_NO_INVERSE_RELATIONSHIP_WARNINGS```. The default value is ```NO```; change this to ```YES```.
- You may need to clean your project, close XCode, reopen your project, and build to see the warnings disappear.
