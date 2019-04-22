---
title: Staple Invoice Data Extraction REST API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - python
  - javascript

includes:
  - errors

search: true
---

# Introduction
### What is this API for? 
Scanning invoices and extracting data! Just send up an invoice, and we will return a JSON of the document including as much data as we could extract, along with details of where that information was on the page, and a new tagged image of the document. 

This API is optimized for computer generated PDFs, which will have the best performance. However, pdf scans, png and high resolution jpg are also supported. Currently the API is set with a 60 second time out, which will be extended shortly.

The API is load balanced, and should be able to handle a significant number of requests in parallel, but, as we are in Beta, we appreciate your patience with any issues that may occur regarding dropped responses.

Staple AI reserves the right to make changes to the API, including registration process, access and response values at any time. That means that the returned values you see now are not fixed, and may be subject to change.   

### Free Demo
See our <a href="https://demo.staple.io/">demo</a> if you wish to sample the invoice scanning using a simple drag and drop interface without using the API.

### In the future 
Please expect the API to change and improve over the coming weeks and months are we refine our data extraction abilities. Dictionaries keys of retruned JOSN will not be removed, however new ones may be added, and our accuracy will be improving consistently.

We will be optimising our data extraction techniques, and allowing receipts, purchase orders and delivery notes. Each of these will have a seperate optimized endpoint, however we are also adding a generic endpoint that will both classify and scan the document (just incase you don't want to have to presort documents before sending them).

# Usage
The API is currently **FREE** to use while in Beta - provided different users register new accounts (see below). Free access it limited to 1,000 calls, but may be extended on request by contacting <a href="mailto:josh@staple.io">josh@staple.io</a>. Feel free to spread this document to other parties you may feel would be interested. In short - go nuts.

## Python Examples for API usage

The following is example code to use the API using python. The API is designed to return data to the user in a convienient manner to be used within existing software builds.

## Registration Endpoint

New users are required to register for the API. Simply decide on a username and password and you will be given two (2) tokens: an access token and refresh token.

```python
# To POST
# import required packages
import requests
import pprint
import json

# define the url
registration_url = 'http://scanning.staple.io/registration'

# define the headers
headers = {
  'Content-Type': 'application/json'
}

# define the payload - with chosen username and password
# these should be lowercase
payload = {
  “username”: “mynewusername”,
  “password”: “mysupersecretpassword”
}

# call the API and print the response
response = requests.request('POST', registration_url, json = payload, allow_redirects=False)
pprint.pprint(json.loads(response.text))
```

## Returned value: success
If the post is successful, the access token and refresh tokens will be returned as a string. Save these for use later.
```JSON
{"message": "user mynewusername was created", "access_token": "***********", "refresh_token": "*******"}
```

## Returned value: with error
Username already exists. For example:
```JSON
{"message": "user mynewusername already exists"}
```

## Other errors
API call was made incorrectly.


## TokenRefresh Endpoint

Access tokens are only valid for a period of time (currently set to 1 week). However, using your refresh token you may retrieve a new access token for use.

```python
# define the URL
refresh_url = 'http://scanning.staple.io/token/refresh'

# define the empty payload
payload = {}

# define the headers
headers = {
  'Authorization' : 'Bearer ' + refresh_token
}

# call the API and print the response

response = requests.request('POST', refresh_url, headers = headers, json = payload, allow_redirects=False)
pprint.pprint(json.loads(response.text))
```

###  Responses:
In the case that the post is successful, you will be given a new access token. Use it wisely.
```JSON
{"access_token": "*******************"}
```

## Login Endpoint

If you have forgotten/lost your refresh and/opr access token, then retrieve them using the login endpoint.

```python
# define the URL
login_url = 'http://scanning.staple.io/login'

# define the payload
payload = {
    'username': “mynewusername”,
     'password': “mysupersecretpassword”
}

# define the header
headers = {
    'Content-Type': 'application/json'
 }

# Call the API and print the response
response = requests.request('POST', login_url, headers = headers, json = payload, allow_redirects=False)

pprint.pprint(response.text)
```

### Responses:
If the user name already exists then you will recieve the following response
```JSON
{"message": "logged in as mynewusername", "access_token": "***************", "Refresh_token": "*****************"}
```

## Invoice Endpoint

Once you have a valid access token (either from registration, login or refresh), you may now call the API.

The input should be either a pdf, png or jpg. Please bear in mind multipay pdfs and larger png images will take longer to process. Files which are too large will currently cause a system error (sad kitten section).

```python
# define the endpoint

invoice_url = 'http://scanning.staple.io/invoice'

# define the file you wish the send
invoice_file_name = ‘myexampletestinvoice.pdf’
files = {'file': open(invoice_file_name,'rb')}

# define the headers - this should include a valid access token
headers = {
  'Authorization': 'Bearer ' + access_token
}

# call the API and print the response - this may take up 1 minute depending on the

# document size and complexity
response = requests.request('POST', invoice_url, headers = header, files = files, allow_redirects=False)

# print the response
pprint.pprint(json.loads(response.text))

# remove some prepended data from the base64 before conversion
Tagged_image = invoice_data['base64'].replace('data:image/png;base64,','')

# remove image from dict before printing
Invoice_data['base64'] = None

# print the response
pprint.pprint(json.loads(response.text))

# convert the base64 section into a png and save it
import base64
with open("imageToSave.png", "wb") as fh:
    image_64_decode = base64.b64decode(Tagged_image)
    fh.write(image_64_decode)
```

Returned values: this contains all of the data extracted from the invoice. Also tables are represented in 2 formats - standardised header and original headers.

```JSON
{
    "Type": "Invoice",
    "Client": null,
    "table_detected": true,
    "Invoice number": "42245626",
    "PO number": "PORUK-08237",
    "Invoice date": "08/14/2018",
    "Other date": "08/03/2018",
    "Account number": "270-519912-270",
    "Swift code": "HFU9CATT",
    "Total": 171.5,
    "Tax total": 0,
    "Currency": [
        "GBP",
        "GBP"
    ],
    "LineItems": [
        {
            "item ": [
                "20"
            ],
            "Material ": [
                "1C-ACC6-ENT",
                "ACC 6 ENT, 1 Cameras"
            ],
            "Quantity ": [
                "1 EA"
            ],
            "Price ": [
                "171.50"
            ],
            "Amount ": [
                "171.50"
            ]
        }
    ],
    "LineItems_standardHeader": [
        {
            "Other0": "20",
            "Description": "1C-ACC6-ENT ACC 6 ENT, 1 Cameras",
            "Quantity": 1,
            "UnitAmount": 171.5,
            "LineAmount": 171.5
        }
    ],
    "Billing address": "Company \n Staple. \n 8 32 Carpenter St \n Singapore",
    "Shipping address": "Windmill Antiques \n BroadEye Hill \n Staffordshire \n England, \n UNITED KINGDOM",
    "Company name": " Staple ",
    "Sender address": " Box 389 \n 101 \n 1001 West Broadway \n Calgary, AB, P6J 4E4 \n Canada \n ",
    "Other address": "Generic Corporation Canada",
    "base64": "VERYLONGSTRINGHERE"
}
```

Here the Base64 image encoding field has been omitted. This is a very long  string - so it may be worth removing this value from the string before printing as in the example.












# Errors
## Time-out (error code 504)

Such an error indicated the API did not complete its Scan in the 60 second connection window - and may likely occur in the case of multipage documents. This time window is in process of being extended.

## The sad kitten (failure to scan)

> The returned JSON for a sad kitten error is:

```json
[
  {
    "status": "Internal processing error",
    "log": "We could not process this document.",
    "advice": "Try a different pdf.",
    "moreAdvice": "Ensure you have uploaded an invoice.",
    "base64": "base64_encoding_of_sad_kitten_image"
  }
]
```

> The image may be represented back as a png if you really want to see the kitten.

<aside class="warning">Seeing the sad kitten is always a bad thing. It means that we were unable to interpret the data on your document. </aside>

The sad kitten means that our API could not process the document you posted, in which case a base 64 image of a very sad kitten is returned.

The image may be represented back as a png if you really want to see the kitten.

```python
# remove some prepended data from the base64 before conversion
Tagged_image = invoice_data['base64'].replace('data:image/png;base64,','')

# remove image from dict before printing
Invoice_data['base64'] = None

# print the response
pprint.pprint(json.loads(response.text))

# convert the base64 section into a png and save it
import base64
with open("verysadkittenimage.png", "wb") as fh:
    image_64_decode = base64.b64decode(Tagged_image)
    fh.write(image_64_decode)

```

Although logs are created internally when sad kitten errors occur - we would appreciate if API users would send the offending documents to <a href="mailto:josh@staple.io">josh@staple.io</a>, so we can ensure this error does not happen again. We never want to see the sad kitten and neither do you :(.




