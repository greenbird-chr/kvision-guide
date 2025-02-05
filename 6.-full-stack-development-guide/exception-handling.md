# Exception Handling

When the backend code throws an unhandled exception, it will be logged on the server side and then propagated by KVision to the frontend side. There are of course many different classes of exceptions on the JVM and it's not possible to transform those types directly to the JS side. So only the exception message is propagated and the generic `Exception` object is created on the frontend side.&#x20;

There is a special `io.kvision.remote.ServiceException` class defined in common KVision code. Unlike all other exception classes this one will be propagated directly from `ServiceException` to `ServiceException` and will not be logged on the backend side. It should be used as typical business error indication.

{% code title="Backend.kt" %}
```kotlin
actual class PasswordService : IPasswordService {
    override suspend fun changePassword(oldPassword: String, newPassword: String) {
        if (oldPassword == newPassword) {
            throw ServiceException("You should really change your password")
        }
        // ...
    }
}
```
{% endcode %}

{% code title="Frontend.kt" %}
```kotlin
val passwordService = PasswordService()
try {
    passwordService.changePassword(oldPassword, newPassword)
} catch (e: ServiceException) {
    Alert.show("Error", e.message)
}
```
{% endcode %}

## User-defined exceptions

Since KVision 5.8.2 it is possible to define custom exception types in the common module, that will be propagated from the backend to the fronted side. Such exception need to inherit from `AbstractServiceException` and the serialization configuration of polymorphic serializers need to be declared with `RemoteSerialization.customConfiguration` property.

```kotlin
@Serializable
class MyFirstException(override val message: String) : AbstractServiceException()

@Serializable
class MySecondException(override val message: String) : AbstractServiceException()

fun configureSerialization() {
    RemoteSerialization.customConfiguration = Json {
        serializersModule = SerializersModule {
            polymorphic(AbstractServiceException::class) {
                subclass(MyFirstException::class)
                subclass(MySecondException::class)
            }
        }
    }
}
```

When this function is called on both the frontend and the backend (e.g. in both `main()` functions), you should be able to throw custom exceptions on the backend side and catch them on the frontend side.
