# C\# Read Only Interfaces 

When generating cached methods, Dataness returns read only interfaces instead of the underlying item class.
For instance a UserItem will be returned as a IUserReadOnly interface insted.

This is done to prevent modification of the items in the cache.

The generated code supports changing the interface which is used, as long as the new interface extends the generated read only interface.

Using a UserItem as an example, we might want to extend the user with an extra property, for instance IsAdmin.

Now when we get a cached list of users, we cannot see this property, as it's not part of the IUserReadOnly interface.

Here's how to fix that.

## 1. Create a new interface like this:

```C#
public interface IExtendedUserReadOnly : IUserReadOnly {
	bool IsAdmin { get; }
}
```

## 2.Implement the interface
After adding the interface, update your UserItem so it implements the new interface
```C#
public class UserItem : BaseUserItem, IExtendedUserReadOnly
{
	public bool IsAdmin => true;
}
```

## 3. Updating Service
We need to tell our UserService that it should now return a different read only interface.
This is done by adding the name of your new interface as a generic argument for the BaseUserService, as shown here.
```C#
public class UserService : BaseUserService<UserService, UserItem, UserItemCollection, UserRepository, UserFactory, UserDbModel, IExtendedUserReadOnly>
{
}
```

## 4. Updating ItemCollection
Like the UserService, we also need to update UserItemCollection in the same way..

Again this is done by adding the name of your new interface as a generic argument for the BaseUserItemCollection, as shown here.
```C#
public class UserItemCollection : BaseUserItemCollection<UserItem, IExtendedUserReadOnly>
{
}
```

## 5. Updating Factory
And lastely, we need to do the same to our UserFactory.

Again this is done by adding the name of your new interface as a generic argument for the BaseUserFactory, as shown here.
```C#
public class UserFactory : BaseUserFactory<UserItem, UserItemCollection, UserDbModel, IExtendedUserReadOnly>
{
}
```

That's it, your cached methods will now return a IExtendedUserReadOnly insted of IUserReadOnly.
