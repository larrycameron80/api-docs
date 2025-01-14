##Authentication

For authentication, you need to generate your API key under your Paxful account settings in developer tab

Example of request content
```
apikey=c3KJ7M5qw2SKSrw3QnQA8D2DHZCpwTlx&nonce=1455035029943&offer_hash=Agq1Bpw7oX9&apiseal=9b67236984c40a7c66892782cc3b0426833e699a1b44e1a5b44d63d82ba75767
```

In which following parameters are required:

   * apikey - Your key generated under your Paxful account settings, unique per user
   * nonce - current unix timestamp (required to protect against replay attacks)
   * apiseal - signature (digest) of the request params passed through HMAC-SHA256 construct
   
   
To generate the apiseal, you need to pass the request payload (i.e. apikey, nonce + request parameters) through hash method using the secret key provided from the UI

#####Examples
OpenSSL:

```
echo -n “apikey=c3KJ7M5qw2SKSrw3QnQA8D2DHZCpwTlx&nonce=1455035029943&offer_hash=Agq1Bpw7oX9” | openssl dgst -sha256 -hmac f8KjbW13VxrY1ziOF5j48Kd9OYxSmleT
```

```
(stdin)= 9b67236984c40a7c66892782cc3b0426833e699a1b44e1a5b44d63d82ba75767
```

PHP: minimal working example

```
// Payload which is sent to server
$payload = [
    'apikey' => 'YOURAPIKEY',
    'nonce' => time(),
];

// Generation of apiseal
// Please note the PHP_QUERY_RFC3986 enc_type
$apiseal = hash_hmac('sha256', http_build_query($payload, null, '&', PHP_QUERY_RFC3986), 'YOURSECRET');

// Append the generated apiseal to payload
$payload['apiseal'] = $apiseal;

// Set request URL (in this case we check your balance)
$ch = curl_init('https://paxful.com/api/wallet/balance');

// NOTICE that we send the payload as a string instead of POST parameters
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($payload, null, '&', PHP_QUERY_RFC3986));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Accept: application/json; version=1',
    'Content-Type: text/plain',
]);

// fetch response
$response = curl_exec($ch);

// convert json response into array
$data = json_decode($response);

var_dump($data);

curl_close($ch);
```

Javascript:

``` html
<script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/3.1.2/rollups/hmac-sha256.js"></script>
<script>
    var xhr = new XMLHttpRequest(),
        secret = 'aefij3ldaase_ase23fdAdwjnA2123fFa',
        body = 'apikey=' + 'dgsdrij234fsdfgkhr' + '&nonce=' + Date.now() + '&offer_hash=Agq1Bpw7oX9&margin=50';
    
    var seal = CryptoJS.HmacSHA256(body, secret);

    xhr.open('POST', 'https://www.paxful.com/api/offer/list');
    xhr.setRequestHeader('Content-Type', 'text/plain');
    xhr.setRequestHeader('Accept', 'application/json');  
    xhr.send(body + '&apiseal=' + seal);    
</script>
```

Python 3:
```python

import hmac
import time
from hashlib import sha256
from urllib.parse import urlencode

import requests  # pip install requests

API_URL = "https://paxful.com/api/offer/list"
API_KEY = "<your api key>"
API_SECRET = "<your api secret>"
nonce = int(time.time())

payload = {"apikey": API_KEY, "nonce": nonce}
payload = urlencode(sorted(payload.items()))
apiseal = hmac.new(API_SECRET.encode(), payload.encode(), sha256).hexdigest()
data_with_apiseal = payload + "&apiseal=" + apiseal
headers = {"Accept": "application/json", "Content-Type": "text/plain"}
resp = requests.post(API_URL, data=data_with_apiseal, headers=headers)


```

Python 2.7:
```python

import time
import hmac
import requests
import urllib
from hashlib import sha256
nonce = int(time.time())
url = 'https://paxful.com/api/offer/list'
payload = {'apikey':pax_key, 'nonce':nonce}
payload = urllib.urlencode(sorted(payload.items()))
apiseal = hmac.new(pax_secret, payload, sha256).hexdigest()
data_with_apiseal = payload + '&apiseal=' + apiseal
headers = {'Accept': 'application/json', 'Content-Type': 'text/plain'}
resp = requests.post(url, data = data_with_apiseal, headers=headers)

```

C#
```csharp
using System;
using System.IO;
using System.Net;
using System.Text;
using System.Security.Cryptography;


// Here is the method that generates hmac
class HmacGenerate
{
   public string GenerateHMAC(string secret, string payload)
   {
       var keyBytes = Encoding.UTF8.GetBytes(secret);
       var hmac = new HMACSHA256 { Key = keyBytes };
       var rawSig = hmac.ComputeHash(Encoding.UTF8.GetBytes(payload));
       return BitConverter.ToString(rawSig).Replace("-", string.Empty).ToLower();
   }
}

//Here we make the request
public class WebRequestPost
{
   public static void Main()
   {
       HmacGenerate hmac = new HmacGenerate();
       //Generate today's date in the format of dd.MM.yyyy
       DateTime dateAndTime = DateTime.Now;
       var date = dateAndTime.ToString("dd.MM.yyyy");
       //Variables apiKey, secret are used for hmac generation, you need to use your secret and api key from user settings
       var apiKey = "yourApiKey";
       var secret = "yourApiSecret";
       //Variables offerHash, margin are used for this example request, those variables should contain your information
       var offerHash = "2zEeNVVADeb";
       var margin = 50;
       //Variables body and theKey is used for request body, theKey holds generate hmac
       var body = "apikey=" + apiKey + "&nonce=" + date + "&offer_hash=" + offerHash +"&margin=" + margin;
       var theKey = "&apiseal="+hmac.GenerateHMAC(secret,body);
       
       // Create a request using a URL that can receive a post.
       WebRequest request = WebRequest.Create("https://paxful.com/api/offer/list");
       // Set the Method property of the request to POST.
       request.Method = "POST";
       // Create POST data and convert it to a byte array.
       string postData = body + theKey;
       byte[] byteArray = Encoding.UTF8.GetBytes(postData);
       // Set the ContentType property of the WebRequest.
       request.ContentType = "application/text";

       // Set the ContentLength property of the WebRequest.
       request.ContentLength = byteArray.Length;
       // Get the request stream.
       Stream dataStream = request.GetRequestStream();
       // Write the data to the request stream.
       dataStream.Write(byteArray, 0, byteArray.Length);
       // Close the Stream object.
       dataStream.Close();
       // Get the response.
       WebResponse response = request.GetResponse();
       // Display the status.
       Console.WriteLine(((HttpWebResponse)response).StatusDescription);
       // Get the stream containing content returned by the server.
       dataStream = response.GetResponseStream();
       // Open the stream using a StreamReader for easy access.
       StreamReader reader = new StreamReader(dataStream);
       // Read the content.
       string responseFromServer = reader.ReadToEnd();
       // Display the content.
       Console.WriteLine(responseFromServer);
       // Clean up the streams.
       reader.Close();
       dataStream.Close();
       response.Close();
   }
}
```

##Request and response formats

Valid Request headers **MUST** contain

```
Accept: application/json; version=1
Content-Type: text/plain
```

Currently there's only version=1 supported. When a new version becomes available, if no "version" is set, it will use the latest one. For compatibility you should use the version header appended to the Accept Header

Response formats are always in json containing data *status* *timestamp* and *data* or *error* (in case of errors)

Example of a successful response

``` javascript
{
    "status": "success",
    "timestamp": 1455032576,
    "data": {
        "success": true,
        "offer_hash": "Agq1Bpw7oX9"
    }
}
```

Example of error response

``` javascript
{
    "status": "error",
    "timestamp": 1455032576,
    "error": {
        "code": 404,
        "message": "Not Found"
    }
}
```

All Responses are expected to have status code 200

Response also contains two custom header

```
X-RateLimit-Limit
X-RateLimit-Remaining
```

The _X-RateLimit-Limit_ shows your overall limit, while the _X-RateLimit-Remaining_ shows how many requests you're allowed to make until you hit your hour limit.

##Testing correctness of API seal

Some libraries are generating HMAC incorrectly or have multiple ways of doing it. We have example tester that you can compare your generated API seal and API seal generated by our correct example code.

https://raw.githubusercontent.com/paxful/api-docs/master/api_test.htm