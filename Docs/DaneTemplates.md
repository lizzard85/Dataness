# .dane template files

Dataness runs on templates defined in .dane files.
At the moment there's no schema for it, but I'll add that in the future.
Here's an example of such a file which maps to a User table in the database:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<entities namespace="Users">
	<entity name="User">
		<db table="Users" connection="readonly" />
		<uses>
			<use name="DatalessTest.BusinessLogic.Users.Enums" in="Item|Factory|Service" />
		</uses>
		<columns>
			<column name="AccessLevel" type="UserAccessLevel" nullable="false" />
		</columns>
		<methods>
			<method name="GetAllUsers" returntype="list">
				<query>
					SELECT * FROM Users
				</query>
			</method>
			<method name="GetUsersByAccessLevel" returntype="list">
				<param name="accessLevel" type="int" />
				<query>
					SELECT * FROM Users WHERE AccessLevel = @accessLevel
				</query>
			</method>
			<method name="GetAccessLevelsInUse" returntype="list" valuetype="int">
				<query>
					SELECT DISTINCT AccessLevel FROM Users
				</query>
			</method>
			<method name="GetUserAccessLevel" returntype="single" valuetype="int">
				<param name="userId" type="int" />
				<query>
					SELECT AccessLevel FROM Users WHERE ID = @userId
				</query>
			</method>
			<method name="GetUsersByType" implementation="Manual">
				<param name="userType" type="int" />
			</method>
		</methods>
	</entity>
</entities>
```

Let try and break down the individual components of the template.

## Entities
```xml
<entities namespace="Users">
</entities>
```

Attributes:
- `namespace` **\*Required\*** The namespace for all entities in this template.

This is the root element for the template. The only thing required here is to add the namespace for the entities in this template, here we use "Users" as we want all classes in in this template to have the same namespace, for instance "MyProject.DataAccess.Users".

## Entity
```xml
<entity name="User">
</entity>
```

Attributes:
- `name` **\*Required\*** The name of the entity

An entity is what we map to a table in the database, so in this case, our "User" entity will be mapped to our Users table when we generate the code.
The name property is used to name alle the different generated classes, so here our generated classes will be called UserItem, UserService and so on.

## DB

```xml
<db table="Users" connection="readonly" />
```

Attributes:
- `table` **\*Required\*** The name of the database table, will become optional once support for views is added
- `connection` Name of the connection to use

This tag is used to tell the generator the name of the table we want to map our entity to.

The "coonnection" attribute is optional and can be used when you want some entities to use a specific connection, defaults to "db".

_**NOTE:** Support for database views, and entities without a table or view is coming_


## Uses

```xml
<uses>
	<use name="DatanessTest.BusinessLogic.Users.Enums" in="Item|Factory|Service" />
</uses>
```

Attributes:
- `name` **\*Required\*** the name of the reference to include.
- `in` **\*Required\*** Where to include the reference

When generating code you might need to add a reference to a namespace the generator doesn't know you need. This can be done using the "uses" element.

Since most references is only needed in specific files, like in the above example where we need a reference to an enum we're mapping a column to, you have to tell the generator which entity types needs this reference.

You do this using the "in" attribute, accepted values are:
- All
- Factory
- Service
- Repository
- Item
- ItemCollection
- DbModel

You can include the namespace in multiple of the generated classes by using | as a seperator, using All is the same as adding all the other options.

## Columns

```xml
<columns>
	<column />
</columns>	
```
You have the option to override columns, for instance if you like in this example want to change the datatype of a column to use an enum instead of an int to represent an accesslevel for the user.
You can also override whether or not the column should be considered nullable.

### Columns - Column
```xml
	<column name="AccessLevel" type="UserAccessLevel" nullable="false" />
```
Attributes:
- `name` **\*Required\*** the name of the column.
- `type` type of the column
- `nullable` whether or not the column is nullable, true|false.

_**NOTE:** More options coming in the future, including creating a class based completely on these columns, instead of a database table/view_

## Methods
```xml
<methods>
	<method />
</methods>
```

The methods tag is where we place all the methods we want generated for this entity.

## Method
```xml
<method name="GetAllUsers" returntype="list">
```
The method tag is used to define the methods for the entity. You can add as many methods as you wish.

Available method attributes:
- `name` This is the name of the method
- `returntype` can be "list" or "single", determines whether the function returns a collection of items, or a single item. "list" is the default option.
- `valuetype` if you wish to return a single value, or a list of single values, you can add a valuetype to the method. This supports the same types as the type on parameters.
- `modifier` The access modifier for the generated method on the service class.

### Method - Param
```xml
	<param name="accessLevel" type="int" />
```
- `name` name of the parameter, this will be used as a name for the argument passed to the method, and as a parameter in the sql query
- `type` the type of the parameter.
	- _Supported Types_: Byte, SByte, Short, UShort, Int, UInt, Long, ULong, Float, Double, Decimal, Char, Bool, Object, String, DateTime
### Method - Query
```xml
	<query>
		SELECT * FROM Users
	</query>
```

The query tag is used to provide the query needed for the repository to get the data, for instance a sql query when generating code with a sql database as the datasource.
