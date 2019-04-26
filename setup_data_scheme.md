[ &larr; back](setup_root.md)
<br/>
# Data Models

For each one of the projects that you want to include in the project you will need to describe the model as well as its possible relations with any other model. The description should be placed in a json file following the [json specs](#json-specs) for this purpose. You will need to store all these json files in a single folder. Another limitation is that each model should have a unique name independently of it's type. From now on, in this document, we will assume that all json files for each one of your data models will be stored in the directory `/your-path/json-files`

### JSON Specs

Each json file describes one and only one model. However, one model can reference to another model using association mechanism described above. Withing `associations` block it is required to specify model names from another json files.

For each model we need to specify the following fields in the json file:

Name | Type | Description
------- | ------- | --------------
*name* | String | Name of the model (it is recommended to use snake_case naming style to obtain nice names in the auto-generated GraphQL API).
*storageType* | String | Type of storage where the model is stored. So far can be one of __sql__(for local relational databases supported by [sequelize](#http://docs.sequelizejs.com/) such as PostgreSql/MySql etc. ) or __Webservice__ for any database that your project would remotely connect to. 
*attributes* | Object | The key of each entry is the name of the attribute and the value should be the a string indicating the type of the attribute. See the [table](#types-spec) below for allowed types. Example: ```{ "attribute1" : "String", "attribute2: "Int" }```
*associations* | Object | The key of each entry is the name of the association and the value should be an object describing corresponding association. See [Associations Spec](#associations-spec) section below for details.

### Supported Data Types
The following types are allowed for the attributes field.

 Type |
------- |
String |
Int |
Float |
Boolean |



### Associations Spec

We will consider four possible types of associations:
1. belongsTo
2. hasOne
3. hasMany
4. belongsToMany

For all type of association, except for association of type 4 (belongsToMany), the necessary arguments would be:

name | Type | Description
------- | ------- | --------------
*type* | String | Type of association (like belongsTo, etc.)
*target* | String | Name of model to which the current model will be associated with.
*targetKey* | String | A unique identifier of the association for the case where there appear more than one association with the same model. 
*targetStorageType* | String | Type of storage where the target model is stored. So far can be one of __sql__ or __Webservice__.
*label* | String | Name of the column in the target model to be used as a display name in the GUI.
*sublabel* | String | Optional name of the column in the target model to be used as a sub-label in the GUI.

When association is of type 4, it's necessary to describe two extra arguments given that the association is made with a cross table. These arguments are:

name | Type | Description
------- | ------- | --------------
*sourceKey* | String | Key to identify the source id
*keysIn* | String | Name of the cross table

## NOTE:
 It's important to notice that when association involves an external web service then foreign key that refers remote elements should be explicitly written into the attributes field of the given local model.
 
 
 When BOTH models have `sql` storageType foreign keys should NOT be explicitly written and will be added automatically by our code generator.

Example:
```
{
  "model" : "book",
  "storageType" : "sql",
  "attributes" : {
    "title" : "String"
  },
  "associations":{
      "publisher" : {
        "type" : "belongsTo", // FK to publisher will be stored in the Book model
        "target" : "publisher", // Model's name is `publisher`
        "targetKey" : "publisher_id", // Local alias for this association
        "targetStorageType" : Webservice", //  It's a remote database
        "label" : "name" // Show in GUI the name of the publisher taken from external DB
        }
  }
}
```

## NOTE:

The same data model description files (.json) can be used for generating both, the [BACKEND](setup_backend.md) and [FRONTEND OR GUI](setup_gui.md). Fields such as  *`label`* and *`sublabel`* in the model specification that are only needed for GUI generator are ignored by the backend generator.

### About the associations type

In ScienceDB, association relationships are based on the [sequelize ORM terminology](http://docs.sequelizejs.com/manual/tutorial/associations.html). As usually, in any association type the foreign-key shall be placed inside one of the two associated tables. However here both table models need to be notified about that link. This association information shall be placed in each model definition JSON files. Below we consider the models for two tables A and B, that are defined in the files A.json and B.json correspondingly.

Within the body of any model JSON file, the model that is described in this file is considered as a source model, and any other model is considered as target. Therefore, within the JSON source it is required to specify how current model is associated with all it's targets.

To refer association link within source model JSON file, the next keywords can be used:

* belongsTo
* hasOne
* hasMany
* belongsToMany


The meaning of these keywords and their valid combinations are discussed from the point of view of the standard association relationships: __* : 1__ (many-to-one), __* : *__ (many-to-many) and __1:1__ (one-to-one).

1. __Many-to-One__

This case happens when more than one element of the table A can reference the same element of table B. In this case, the foreign-key has to be created in the table A.

Example: Table A contains a list of employees and each of them can work in a single department only (catalog B).

A.json: belongsTo B;
B.json: hasMany A;
(table A keeps a not unique foreignkey_B)

2. __Many-to-Many__

Based on the previous example, this type of relationship underlines the fact, that an employee can belong to more than one department as well as the department can incorporate more than one employee. In this case, a new relation table has to be created that would hold the foreign-key pairs that define employee-to-department associations.

Example:
A.json belongsToMany B;
B.json belongsToMany A;
(AB relation table with foreignkey_A and foreignkey_B is created automatically)

3. __One-to-One__

This relation type contains restriction: an absence of the possibility to relate more than one pair of the elements. In this case it is a subject of intuition which table shell hold the foreign-key. However, the foreign-keys can't repeat and shall be unique.

For example, the A table is a catalog of well studied species that have strict registration number, whereas table B contains all discovered species, some of which are not cataloged yet.

A.json: hasOne B;
B.json: belongsTo A;
(table B keeps a unique foreignkey_A)