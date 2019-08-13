# Authorization

## User account creation

Account creation is done by performing an *HTTP POST* request to our backend.

**POST** https://api.platform.muzzley.com/v3/i18n/legacy/account


In order for an account to be created, the following fields must be provided in a JSON body payload:

* **name** : name of the user.
* **email** : user email
* **password** : user password
* **appId** : namespace that uniquely identifies your application in our platform. You can find it in our [Selfcare Back Office](https://selfcare.habit.io)
* **device** : identifies the type of device you're using (_iphone, ipad, android_)

Example:

```
{
  "name": "John Doe",
  "email": "john.doe@email.com,
  "password": "user_password",
  "appId": "your_app_namespace",
  "device": "iphone"
}
```

If the request succeeds, you can safely ignore the resulting payload, and proceed with the login.

If an error occurs, you can examine the ***X-Error*** header for more information about the error.

For example, if it is **1211**, it means "User already exists".

## Login

User login is accomplished by performing an *HTTP GET* using this endpoint:

**GET** 
https://api.platform.muzzley.com/v3/i18n/auth/authorize?client_id=CLIENT_ID&response_type=password&scope=application,user&username=USERNAME&password=PASSWORD

The following parameters placeholders need to be replaced with your own:

* **USERNAME** : user email *(escaped)*
* **PASSWORD** : user password *(escaped)*
* **CLIENT_ID** : uniquely identifies your application in our platform. You can find it in our [Selfcare Back Office](https://selfcare.habit.io). ***Attention**: client_id is not the same as the application namespace.*


If it succeeds, the result of this operation is an object with the user authorization info which contains the following fields.

```
{
  "client_id": String,
  "owner_id": String (User ID),
  "access_token": String,
  "refresh_token": String,
  "expires": "2019-11-10T13:28:59.542+0000",
  "grant_type": "password",
  "endpoints": {
      "http": String,
      "mqtt": String,
   }
}
```

If an error occurs, you can examine the header ***X-Error*** for a detailed cause of the error.

If it's **1601**, it means that "User credentials are invalid"

## Refresh Token

If the **access_token** you've acquired on login expires, you need to refresh it by using the **refresh_token** that was provided on login.

To request a new token, you need to perform an *HTTP GET* and replace the **your_client_id** and **your_refresh_token** parameters with your own.

**GET** https://api.platform.muzzley.com/v3/i18n/auth/exchange?client_id=your_client_id&refresh_token=your_refresh_token&grant_type=password

If the request is successful you should receive a new authorization info in the same format as on login:

```
{
  "client_id": String,
  "owner_id": String (User ID),
  "access_token": String,
  "refresh_token": String,
  "expires": "2019-11-10T13:28:59.542+0000",
  "grant_type": "password",
  "endpoints": {
      "http": String,
      "mqtt": String,
   }
}
```


# Support

For more information contact us at support@habit.io  
