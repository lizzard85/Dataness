DMS.Dataness

**NOTE** This project is still under development, and still needs a lot of features and testing, use at your own risk!

Dataness is a code generator that creates code for DataAccess and BusinessLogic.
Out of the box the library comes with support for C# language and MsSql and MySql databases.
It's possible to extend the library and use it with other programming languages and databases.
This guide is based on the C# implementation, with one of the included database schema readers.

Structure 
	- Core (Required to run the generator)
		- DMS.Dataness.dll
		- DMS.Dataness.Common.dll
		- DMS.Dataness.Readers.Database.dll
		- DMS.Dataness.Readers.Xml.dll
		- DMS.Dataness.Templates.dll
		- DMS.Dataness.Writers.dll
	- Implementation (Optional depending on language and database used)
		- DMS.Dataness.Readers.Database.MySql.dll
		- DMS.Dataness.Readers.Database.MsSql.dll
		- DMS.Dataness.Templates.CSharp.dll
		- DMS.Dataness.Writers.CSharp.dll

Prerequisite
Before starting, make sure you have a project ready for use with Dataness. You should create two class libraries in your project, one for DataAccess and one for BusinessLogic.

1. Setting up the generator

First step in using Dataness is to create a console app to use for generating your code.

Add a reference to DMS.Dataness.dll and any of the optional .dlls you need.

Dataness uses the project files from the DataAccess and BusinessLogic projects, so start by mapping paths for those.

string daPath = @"C:\Path\To\YourProject.DataAccess.csproj";
string blPath = @"C:\Path\To\YourProject.BusinessLogic.csproj";

Dataness will also need the path to where the .dane files are stored. More on .dane files later.

string danePath = @"C:\Path\To\YourProject\DaneTemplates";

In order to tell the generator that we'll be generating C# we need to create a CSharpTemplateProvider and a CSharpWriter as shown below.
CSharpTemplateProvider prov = new CSharpTemplateProvider();
var writer = new CSharpWriter(daPath, blPath, prov);

Also create the SchemaReader you wish to use
var sqlReader = new MySqlDbSchemaReader();

Now create the generator based on those instances, the C# Writer comes with most of the required providers, but you can exchange them if you need to, but for now just use this.
DatanessGenerator generator = new DatanessGenerator(writer, sqlReader, writer.Generator, writer, prov);

Now you can tell the generator to parse the .dane files, you can call .Parse multiple times if you need to parse files in different locations. (Note however this feature is experimental and might cause issues if you parse more than one folder at the moment.)
generator.Parse(xmlPath);

Last but not least, call .Generate() and the generator will now create the code for you.
generator.Generate();

2. Create a .dane template file
Dataness runs on templates defined in .dane files.
Here's an example of such a file:

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
					SELECT * FROM $table WHERE AccessLevel = @accessLevel
				</query>
			</method>
			<method name="GetUsersByType" implementation="Manual">
				<param name="userType" type="int" />
			</method>
		</methods>
	</entity>
</entities>

At the moment there's no schema for it, but I'll add that in the future.

Start by defining the root element, "entities", where you set the relative namespace for the entities in this file, this will be combined with the namespace from your DataAccess and BusinessLogic projects.
Inside the entities tag, you can define one or more entities, by adding an "entity".
The entity needs a name, this will be used as the basis for all classes, ie. UserService, UserRepository and so on.
Inside entity, start by adding the database information, "table" is the name of the table in the database we wish to map our code to. "connection" is optional, but can be used if some items should use a seperate connectionstring, if it's not added here, "db" is used as the default identifier for the connectionstring.
After the db tag, add a "methods" tag, here we will define the methods we want generated.
Method attributes:
	- name: This is the name of the method
	- returntype: can be "list" or "single", determines whether the function returns a collection of items, or a single item. "list" is the default option.
	- implementation: can be "manual" or "auto", auto is the default. A manually implemented method will create an abstract method for manual implementation.
Method child elements:
	- query: This tag contains the query for the method
	- param: parameters for the query, and the method
		- name: name of the parameter, this will be used as a name for the argument passed to the method, and as a parameter in the sql query
		- type: the type of the parameter.
			- Supported Types: Byte, SByte, Short, UShort, Int, UInt, Long, ULong, Float, Double, Decimal, Char, Bool, Object, String, DateTime

That's it, now when you run the generator, your code will be ready for you.

3. Project References
In the project you're generating code for, you need to make sure the BusinessLogic project has a reference to the DataAccess project. No other project should have a reference to the DataAccess project.
Any other projects that need access to data, should add a reference to the BusinessLogic project as all communication with the database needs to go through the BusinessLogic Services.

4. Using the generated code

- Get Service instance
var service = UserService.Instance;

- Create User
var user = service.CreateUser("TestUser", "abc123", "logkey123");
user.AutoLoginToken = "Auto123";
int userId = service.SaveUser(user);

- Load User
var newUser = service.LoadUser(userId);

- Update user
newUser.Username = $"{newUser.Username} {userId}";
service.SaveUser(newUser);

- Use methods Generated from .dane file
var allUsers = service.GetAllUsers();
foreach (var usr in allUsers)
{
	Console.WriteLine(usr.Username);
}

var roleUsers = service.GetUsersByAccessLevel(2);
foreach (var usr in roleUsers)
{
	Console.WriteLine("Admin: " + usr.Username);
}

- Delete user
service.DeleteUser(5);

5. Generated Structure
The code generates 6 different entities:
	- DataAccess
		- DbModel
		- Repository
	- BusinessLogic
		- Item
		- ItemCollection
		- Factory
		- Service

All of the above are generated in three layers that uses inheritance to combine the logic.
Using the UserService as an example, the structure will be like this:

	- BaseService
		- BaseUserService
			- UserService

And this goes for all the generated types.

The BaseService class is used across all Services, and will contain code that's reused across all entities. This file is overridden everytime you generate, so don't change anything there.
The Base<Entity>Service class is where all the methods for the entity is generated, like the base class, this will be overwritten everytime you generate.
Lastly the <Entity>Service class is only generated if the file doesn't already exist, in this file, you can add your own functionality if needed.

And again, this goes for all the different generated types, so for instance the UserRepository, UserItem, UserItemCollection are all safe to add your own functionality to.