# .dane template files

Dataness runs on templates defined in .dane files.
Here's an example of such a file:
```xml
<?xml version="1.0" encoding="utf-8" ?>
<entities namespace="Users">
	<entity name="User">
		<db table="Users" connection="readonly" />
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
At the moment there's no schema for it, but I'll add that in the future.

Start by defining the root element, "entities", where you set the relative namespace for the entities in this file, this will be combined with the namespace from your DataAccess and BusinessLogic projects.

Inside the entities tag, you can define one or more entities, by adding an "entity".

The entity needs a name, this will be used as the basis for all classes, ie. "User" for UserService, UserRepository and so on.

Inside entity, start by adding the database information, "table" is the name of the table in the database we wish to map our code to. "connection" is optional, but can be used if some items should use a seperate connectionstring, if it's not added here, "db" is used as the default identifier for the connectionstring.

After the db tag, add a "methods" tag, here we will define the methods we want generated.

Method attributes:
- **name**: This is the name of the method
- **returntype**: can be "list" or "single", determines whether the function returns a collection of items, or a single item. "list" is the default option.
- **valuetype**: if you wish to return a single value, or a list of single values, you can add a valuetype to the method. This supports the same types as the type on parameters.
- **implementation**: can be "manual" or "auto", auto is the default. A manually implemented method will create an abstract method for manual implementation.

Method child elements:
- **query**: This tag contains the query for the method
- **param**: parameters for the query, and the method
	- *__name__*: name of the parameter, this will be used as a name for the argument passed to the method, and as a parameter in the sql query
	- *__type__*: the type of the parameter.
		- _Supported Types_: Byte, SByte, Short, UShort, Int, UInt, Long, ULong, Float, Double, Decimal, Char, Bool, Object, String, DateTime

