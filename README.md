# Neo Sisdam API & Response Information (V1)

This document provides information about the HTTP status codes and their corresponding descriptions used in our API responses.

## Status Codes

| Status Code  | Description                               |
|--------------|-------------------------------------------|
| 00           | Successfully Retrieve Data                |
| 01           | Something Went Wrong (Server)             | 
| 10           | Username or Email Not Found               |
| 11           | Account Should be Activated in Neo Sisdam |
| 12           | Username/Email/Password wrong             |
| 30           | Token Invalid                             |
| 31           | Token can't be refreshed. Re-login        |
| 32           | Token Blacklisted                         |
| 33           | Authorization Token Not Found             |
| 34           | Unauthorized                              |

## Status Code Details

### 00 - OK

The request was successful, and the server has returned the requested data.

### 01 - Server Error

Something Went Wrong on the server.

### 10 - Account Not Found

Username or Email Not Found in the databases.

### 11 - Account Not Activated Yet

Account Access Should be Activated First.

### 12 - Wrong Authentication

The request was invalid or malformed. Please check your request and try again.

### 30 - Token Invalid

Bearer Token was Invalid

### 31 - Token Can't Be Refreshed

User should re-login again.

### 32 - Token Blacklisted

Bearer Token was Blacklisted due expired.

### 33 - Token Not Found

Bearer Token Not Found or Not Provided yet.

### 34 - Unauthorized

User don't have access to the resource.

## API General Endpoint Documentation
**Base URL:** `https://neosisdam.univpancasila.ac.id/api/v1`

| Endpoint                            | Method | Request/Param                          | Description                           |
|-------------------------------------|--------|----------------------------------------|---------------------------------------|
| /login                              | `POST` | {username,password}                    | Authentication                        |
| /user                               | `GET`  | -                                      | Get User Authentication Information   |
| /master/pegawai/get-all             | `GET`  | query params: `filter-pegawai` (dosen) | Get Employee Information              |
| /master/pegawai/dosen/{kode_dosen}  | `GET`  | url params `kode_dosen`                | Get Lecturer Detail Information       |
| /master/pegawai/detail/{id_pegawai} | `GET`  | url params `id_pegawai`                | Get Employee Detail Information       |



## Usage

- When making API requests, always check the HTTP status code in the response to determine the outcome of your request.
- Use the status code and description to troubleshoot issues with your API requests.
- Refer to our API documentation for more specific information on each endpoint and its expected responses.

## API Wifi Management Documentation

| Endpoint                            | Method | Request/Params                         | Description                           |
|-------------------------------------|--------|----------------------------------------|---------------------------------------|
| /wifi-management/                   | `GET`  | -                                      | Get All Wifi Account                  |
| /wifi-management/check-available    | `POST` | {user,type(username/email) }           | Check Username/Email availability     |
| /wifi-management/create-account     | `POST` | {user,password,type (username/email)}  | Create Wifi Account                   |
| /wifi-management/disable-account    | `POST` | {account_id, rule (yes/no) }           | Disable Wifi Account                  |
| /wifi-management/change-password    | `POST` | {account_id,new_password)              | Change Wifi Password                  |


# Using OAuth
This document provides information about how to use OAuth NeoSisdam to make user easily access your website using neo account.

At first Neo Sisdam will provide the Client ID, Client Secret and Callback URI

```
CLIENT ID: your-client-id
CLIENT SECRET: your-client-secret
CALLBACK URI: your-apps-callback
```
Client ID is visible for every client, while Client Secret should put it somewhere else that nobody can know about it caused private visibility.

**Base URL:** `https://neosisdam.univpancasila.ac.id/`

| Endpoint         | Method | Request                                                                                             | Description                   |
|------------------|--------|-----------------------------------------------------------------------------------------------------|-------------------------------|
| /oauth/authorize | `GET`  | query params : client_id, reedirect_uri, response_type (code), scope (mandatory), state (mandatory) | Gain Code Access              |
| /oauth/token     | `POST` | form-data: grant_type (authorization_code), client_id, client_secret, redirect_uri, code            | Gain Access and Refresh Token |

## Example
### 1. Prepare Callback Routes
provide a callback routes for neosisdam that will give information credentials to the third party you have.
```php
route::get('auth/callback',function(){
    //http://localhost/auth/callback
});
```
### 2. Create a Button "Login with Neo Sisdam"
because we're gonna using OAuth 2.0 concept, user can authorize their neo sisdam account to other third parties app
with easily only using a single click.
```php
<form method="POST" action="/login-neo">
    @csrf
    <button type="submit">Login with Neo Sisdam</button>
</form>
```
### 3. Authorize to Neo Sisdam Apps
```php
Route::get('/login-neo',function(){
    session()->put('state', $state = Str::random(40));
    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'your-apps-callback',
        'response_type' => 'code',
        'scope' => 'get-email',
        'state' => $state
    ]);
    return redirect()->to('https://neosisdam.univpancasila.ac.id/oauth/authorize?'.$query);
});
```
as you can see from above, you need to provide some ``state`` to securely communication between our Neo Sisdam
with your third parties app. User will ask to Authorize their account like this.

if user choose not to deny the authorizes, the callback will get some query parameters like this

```json
//http://localhost/auth/callback?state=kmd0QD7eXDyaI497SIq4Uqlaa1REtXZDYk45dQBr&error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.&hint=The+user+denied+the+request&message=The+resource+owner+or+authorization+server+denied+the+request.
{
    "state":"random-string-state",
    "error":"access_denied",
    "error_description":"The resource owner or authorization server denied the request.",
    "hint":"The user denied the request",
    "message":"The resource owner or authorization server denied the request."
}
```
if user choose to `authorize` their account, the callback will get some query parameters like this
```json
{
    "code":"random code",
    "state":"random-string-state"
}
```

### 3. Exchange Code to Gain Access and Refresh Code.
Exchange the code to get Acces and Refresh Code by using this **url endpoint**

`https://neosisdam.univpancasila.ac.id/oauth/token`

here's the full example by putting some logic in your callback routes

```php
Route::get('/auth/callback',function(){
    $state = session()->get('state');
    throw_unless(
        strlen($state) > 0 && $state === request()->state,
        InvalidArgumentException::class,
        'Invalid state value.'
    );
    $response = Http::asForm()
    ->post('https://neosisdam.univpancasila.ac.id/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id-code',
        'client_secret' => 'your-client-secret-code',
        'redirect_uri' => 'your-callback-url',
        'code' => request()->code,
    ]);
    return $response->json();
});
```
here's the response when grant the `bearer token` 

```json
{
    "token_type":"Bearer",
    "expires_in":1296000,
    "access_token":"some-random-access-token",
    "refresh_token":"some-random-refresh-token"
}
```
as you can see from the results json, Neo Sisdam will give you the expire time of access also refresh_token.
the expire time is `seconds` format.


