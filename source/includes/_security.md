# Authentication

```php
<?php

$data = [
	'Email' => 'broker@example.com',
	'Password' => 'BrokerPassword'
];

$url  = 'https://apiuat.paymentshield.co.uk/Security/Login';
$json = json_encode($data);
$curl = curl_init();

curl_setopt_array($curl, [
  CURLOPT_URL => $url,
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 30,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => 'POST',
  CURLOPT_POSTFIELDS => $json,
  CURLOPT_HTTPHEADER => [
	"Content-Type: application/json"
  ]
]);

$response = curl_exec($curl);
$err = curl_error($curl);

curl_close($curl);

$response = $err
	? "cURL Error $err"
	: $response;
```

```http
POST https://apiuat.paymentshield.co.uk/security/login HTTP/1.1
Content-Type: application/json
{
    "Email": "broker@example.com", 
    "Password": "BrokerPassword"
}
```

> Replace the `Email` and `Password` fields with your credentials.

```http
POST https://apiuat.paymentshield.co.uk/security/login HTTP/1.1
Content-Type: application/json
{
    "UserId": "123456", 
    "Passkey": "aaaaaaaa"
}
```

> You can use `UserId` and `Passkey` authentication instead of email and password. The UserId is also known as Seller Number.



We have a custom authentication scheme due to our internal system; we don't use OAuth or similar.

The general flow of authentication is like this:

 1. Authenticate by POSTing either `email`+`password` or `userid`+`passkey` in a JSON body to the Login endpoint
 2. We'll send back a response containing your `UserId` (a constant, unique integer ID) and a session `Token` (GUID) which is valid for 40 minutes. The Token's validity is reset to the full time whenever it is used for an authenticated API call. You can store the `Token` in local storage within your app.
 3. You should know your `SystemId`, a GUID identifying the piece of software calling the API. This is somewhat like an API key (with the `UserId` and `Token` together forming a kind of API secret).
 4. In all subsequent requests to any of our API services, send the `UserId`, `Token`, and `SystemId` in **HTTP headers**.

### Login

To login, POST to the `/Security/Login` endpoint as shown in the code samples on the right.


### Token Refresh

We provide a Token Refresh endpoint to function as a "keep alive", which will renew your session active time, extending the session. It is essentially a no-op, and your session token will stay the same.

We recommend you build this into your integration, for example on button presses, to avoid having to log users back in regularly.

POST to the `/Security/Refresh` endpoint with the usual security headers and no body. The response will be a simple JSON body with a `StatusCode` of OK. If the session has already expired, you will receive the usual 401 error with status `AuthenticationRefused`.