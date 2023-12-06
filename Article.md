# A Generic and Step By Step Workflow for RDP Client File Store (CFS) Bulk File API

## <a id="intro"></a>Introduction

The Client File Store (CFS) is a capability of Refinitiv Data Platform (RDP) that provides authorization and enables access to content files stored in publisher repositories. LSEG provides various content file domains (aka *bucket*) for consumers such as:
- Bulk ESG data
- Green Revenue data
- Symbology data
- Tick History
- Starmine
And much more.

The Client File Store (CFS) is a component of the RDP which let consumers access via a set of HTTP RESTful APIs. This article contains a step-by-step, generic workflow guide from the authentication and download the file for any bucket (ESG, Symbology, Green Revenue, etc). The example source code is written in [Python](https://www.python.org/) and [Jupyter](https://jupyter.org/) environment. However, our APIs are the web-based API, so any programming langues can connect and consume data using the same concept. 

## <a id="whatis_rdp"></a>What is Refinitiv Data Platform (RDP) APIs?

The [Our Delivery Platform (previously known as Refinitiv Data Platform - RDP)](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) provide various LSEG data and content for developers via easy-to-use Web-based API.

RDP APIs give developers seamless and holistic access to all of the LSEG content such as Environmental Social and Governance (ESG), News, Research, etc, and commingled with their content, enriching, integrating, and distributing the data through a single interface, delivered wherever they need it.  The RDP APIs delivery mechanisms are the following:
* Request - Response: RESTful web service (HTTP GET, POST, PUT or DELETE) 
* Alert: delivery is a mechanism to receive asynchronous updates (alerts) to a subscription. 
* Bulks:  deliver substantial payloads, like the end-of-day pricing data for the whole venue. 
* Streaming: deliver real-time delivery of messages.

This example project is focusing on the Request-Response: RESTful web service delivery method only.  

![figure-1](images/01_rdp.png "LSEG Data Platform content set")

For more detail regarding RDP, please see the following APIs resources: 
- [Quick Start](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/quick-start) page.
- [Tutorials](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/tutorials) page.

## <a id="what_is_cfs"></a>What is CFS?

**Client File Store (CFS)** aka File Distribution is a capability of Refinitiv Data Platform (RDP) that provides authorization and enables access to content files stored in publisher-supplied repository. CFS defines content ownership that publisher are isolated. And subscribers can trust the source of content.

CFS is engineered as a self-service metadata tool intend for publishers and subscribers. CFS provides bucket and file-set to organize files to simplify the interaction with publishers or subscribers CFS doesn't store file directly. Actual files are store in publisher-supplied. AWS S3 only one type storage that supported by current CFS.

### Bucket

CFS facilitates buckets for use by Publishers to organize file-sets and files. Buckets store metadata about the files stored in publisher-supplied repositories. Buckets align with subscriptions and can contain multiple file-sets and files.

Publishers are responsible for creating buckets with the CFS API. This is a one-time process. The resulting bucket is owned one or more Publishers and is assigned a unique name that cannot be assigned to another bucket. A Publisher can have multiple buckets if they provide more than one dataset.

Claims are used to control access to buckets, file-sets and files. CFS does not manage or create claims, CFS only enforces them. Claims must be created in AAA. Subscribers must have at least one of the claims on the bucket in order to access the bucket.

Attributes are used to allow Subscribers to filter and search for content. Attributes are one method that a Subscriber can use to find files and/or file-sets.

### Packages

A package is an indivisible set of file-sets that are all delivered together. Packages can consist of multiple file-sets that a grouping of file-sets.

The publisher will define the metadata for each package of content available to subscribers in CFS.  Publishers are responsible for creating Packages and assigning claims to them.

Publisher need to create package first and then publisher able to create file-sets into Packages.

### Fileset

A file-set is an indivisible set of files that are all delivered together. They can consist of multiple files that make up one large file or a grouping of files that represent related content. A file-set can also contain a single file. The Publisher decides the appropriate organization of their file-sets.

Publishers are responsible for creating file-sets into Packages. A Publisher can have multiple file-sets in a bucket.

Once all files have been added to the file-set and are ready for download, the Publisher updates the file-set status to READY. This enables the Publisher to control the release of their files and sets expectations for when the files will be available to subscribers. File-sets with a status of READY cannot be updated or modified. Updates must be published as a new file-set.

To access a file-set, Subscribers must have access to the bucket in which the file-set resides and have all of the claims associate with the file-set.

### Files

Subscribers can only access the files to which they are entitled. On AWS S3, Subscribers can access files using a signed URI that redirects to the file on AWS S3 for downloading.

Files are available for a defined period of time that is determined by the Publisher.

Please find more detail on the [CFS API User Guide](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/documentation#cfs-api-user-guide) document.

## <a id="prerequisite"></a>Prerequisite

Before I am going further, there is some prerequisite, dependencies, and libraries that the project is needed.

### Access to the RDP with the your desire Bulk file permission

This project uses RDP access credentials with the CFS Bulk file permission.

Please contact your Refinitiv representative to help you with the RDP account and the bucket permission.

### Internet Access

This demonstration connects to RDP on AWS via a public internet.

### Python and Jupyter Notebook.

This project uses [Python](https://www.python.org/) and [Jupyter](https://jupyter.org/) environment.

The Python [Anaconda](https://www.anaconda.com/distribution/) or [MiniConda](https://docs.conda.io/en/latest/miniconda.html) distribution/package manager is recommended.

## <a id="rdp_workflow"></a>RDP APIs Application Workflow

### Step 1: Authentication with RDP APIs

The CFS APIs are considered protected resources and require that your application be authenticated before making a data call. This authentication and provisioning of access token is based on OAuth 2.0 specification. The first step of an application workflow is to get a token from RDP Auth Service, which will allow access to the protected resource, i.e. data REST API. 

The API requires the following access credential information:
- Username: The username/machine-id. 
- Password: Password associated with the username/machine-id. 
- Client ID: This is also known as ```AppKey```, and it is generated using an App key Generator. This unique identifier is defined for the user or application and is deemed confidential (not shared between users). The client_id parameter can be passed in the request body or as an “Authorization” request header that is encoded as base64.
- Grant Type ```password```: This is for initial authentication request. An application does not have any token, so it requests new tokens using username/password combination.

The HTTP request for the RDP APIs Authentication service is as follows:

``` HTTP
POST /auth/oauth2/v1/token HTTP/1.1
Accept: */*
Content-Type: application/x-www-form-urlencoded
Host: api.refinitiv.com:443
Content-Length: XXX

username=RDP_USERNAME
&password=RDP_PASSWORD
&client_id=RDP_APP_KEY
&grant_type=password
&takeExclusiveSignOnControl=true
&scope=trapi
```
![Alt text](images/02_rdp_workflow.png)

Once the authentication success, the function gets the RDP Auth service response message and keeps the following RDP token information in the variables.
- **access_token**: The token used to invoke REST data API calls as described above. The application must keep this credential for further RDP APIs requests.
- **refresh_token**: Refresh token to be used for obtaining an updated access token before expiration. The application must keep this credential for access token renewal.
- **expires_in**: Access token validity time in seconds.

Next, after the application received the Access Token (and authorization token) from RDP Auth Service, all subsequent REST API calls will use this token to get the data. Please find more detail regarding RDP APIs workflow in the following resources:
- [RDP APIs: Introduction to the Request-Response API](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/tutorials#introduction-to-the-request-response-api) page.
- [RDP APIs: Authorization - All about tokens](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/tutorials#authorization-all-about-tokens) page.

Example source code in Python:
```Python
import requests 

username = 'your RDP Machine-ID'
password = 'your RDP Password'
clientId = 'your RDP APP Key'

RDP_HOST= 'https://api.refinitiv.com'
acccess_token = None
refresh_token = None
expires_in = 0

# Send HTTP Request
auth_url = f'{RDP_HOST}/auth/oauth2/v1/token'
payload = f'grant_type=password&username={username}&client_id={clientId}&password={password}&takeExclusiveSignOnControl=True&scope=trapi'
try:
    response = requests.post(auth_url, 
                             headers = {'Content-Type':'application/x-www-form-urlencoded'}, 
                             data = payload, 
                             auth = (clientId, '')
                )
except requests.exceptions.RequestException as exp:
    print(f'Caught exception: {exp}')
```

The next step is checking the authentication request result, then get the token information and assign them to local variables for the later use.

``` Python
if response.status_code == 200:  # HTTP Status 'OK'
    print('Authentication success')
    access_token = response.json()['access_token']
    refresh_token = response.json()['refresh_token']
    expires_in = int(response.json()['expires_in'])

if response.status_code != 200:
    print(f'RDP authentication failure: {response.status_code} {response.reason}')
    print(f'Text: {response.text}')
```
Now we have the access token information that can be used for further data requests.

### <a id="rdp_get_data"></a>Requesting Data from RDP APIs

That brings us to requesting the RDP APIs data. All subsequent REST API calls must include the Access Token via the *Authorization* HTTP request message header as shown below to get the data. 
- Header: 
    * Authorization = ```Bearer <RDP Access Token>```

Please notice *the space* between the ```Bearer``` and ```RDP Access Token``` values.

If the token is still valid and has appropriate scope, the request is allowed to access the data.

The application then creates a request message in a JSON message format or URL query parameter based on the interested service and sends it as an HTTP request message to the Service Endpoint. Developers can get RDP APIs the Service Endpoint, HTTP operations, and parameters from Refinitiv Data Platform's [API Playground page](https://api.refinitiv.com/) - which is an interactive documentation site developers can access once they have a valid Refinitiv Data Platform account.

### Step 2: Listing the packageId using the Bucket Name

To request the CFS Bulk data, the first step is to send an HTTP ```GET``` request to the RDP ```/file-store/v1/file-sets?bucket={bucket-name}``` endpoint to list all FileSets under the input ```bucket-name```.

The HTTP Request structure is as follows:

```HTTP
GET /file-store/v1/file-sets?bucket={bucket-name}, HTTP/1.1
Host: api.refinitiv.com
Authorization: Bearer <Access Token>
```

The example bucket names for RDP content set are as follows:

|              Content             |                  Bucket Name                 |           
|:--------------------------------:|:-------------------------------------------:|
| Financial Markets Reference Data | bulk-FMRD                                    | 
| Symbology                        | bulk-Symbology                               | 
| ESG                              | bulk-ESG                                     | 
| ESG - Point in Time              | bulk-ESG                                     | 
| Tick History                     | TICKHISTORY_VBD_NO_EMBARGO                   | 
| Green Revenue                    | bulk-GreenRevenue                            | 
| Starmine                         | STARMINE_PREDICTIVE_ANALYTICS_SMARTECON_LIVE | 

**Note**: The bucket name is *case-insensitive*.

I am using the ```bulk-ESG``` as an example bucket name for the Python code.

```Python
# set Bucket Name, this notebook use bulk-ESG as an example
bucket_name = 'bulk-ESG'

#step 2 - list Package IDs from bucket name

CFS_url = f'{RDP_HOST}/file-store/v1/file-sets?bucket={bucket_name}'

try:
    response = requests.get(CFS_url, headers={'Authorization': f'Bearer {access_token}'})
except requests.exceptions.RequestException as exp:
    print(f'Caught exception: {exp}')

if response.status_code == 200:  # HTTP Status 'OK'
    print('Receive list Package IDs from RDP APIs')
else:
    print(f'RDP APIs: CFS request failure: {response.status_code} {response.reason}')
    print(f'Text: {response.text}')
```
You can check the example data response to see the example of PackageIds from the following code:

```Python
#Example of the first entry of package IDs

print(json.dumps(response.json()['value'][0], sort_keys=True, indent=2, separators=(',', ':')))
```

Result:
```JSON
{
  "attributes":[
    {
      "name":"ContentType",
      "value":"Symbology SEDOL"
    }
  ],
  "availableFrom":"2023-11-12T16:15:14Z",
  "availableTo":"2023-12-12T16:15:14Z",
  "bucketName":"bulk-ESG",
  "contentFrom":"1970-01-01T00:00:00Z",
  "contentTo":"2023-11-12T15:55:00Z",
  "created":"2023-11-12T16:15:14Z",
  "files":[
    "4c88-afd6-e880c53b-b2fb-ecd47fc3297a"
  ],
  "id":"4013-7266-3759750e-b77b-c19ff93186d5",
  "modified":"2023-11-12T16:15:34Z",
  "name":"Bulk-ESG-Global-Symbology-EquitySEDOL-v2-Jsonl-Init-2023-11-12T16:01:55.121Z",
  "numFiles":1,
  "packageId":"4976-f976-fc3caef0-82d1-c345db924b6f",
  "status":"READY"
}
```

### Step 2.5: Listing the packageId using the Bucket Name - Paging

By default, the ```/file-store/v1/file-sets?bucket={bucket-name}``` endpoint always returns 25 results per request. You can adjust the number of return results via the ```pageSize``` query parameter, the maximum number is **100**.

```HTTP
GET /file-store/v1/file-sets?bucket={bucket-name}&pageSize={number}, HTTP/1.1
Host: api.refinitiv.com
Authorization: Bearer <Access Token>
```
Let's try with ```pageSize=2``` as an example.

Python Code:

```Python
#step 2.5 - list Package IDs from bucket name - with pageSize 2
CFS_url = f'{RDP_HOST}/file-store/v1/file-sets?bucket={bucket_name}&pageSize=2'

try:
    response = requests.get(CFS_url, headers={'Authorization': f'Bearer {access_token}'})
except requests.exceptions.RequestException as exp:
    print(f'Caught exception: {exp}')


if response.status_code == 200:  # HTTP Status 'OK'
    print('Receive list Package IDs from RDP APIs')
else:
    print(f'RDP APIs: CFS request failure: {response.status_code} {response.reason}')
    print(f'Text: {response.text}')
```
Now we get 2 entries per request from the API as we set via ```pageSize=2``` parameter:

```JSON
{
  "@nextLink":"/file-store/v1/file-sets?bucket=bulk-ESG&pageSize=2&skipToken=ZmlsZXNldElkPTQwMjAtMTUwZS0yMWY3ODEzZC04MGU0LWYwZjU0NGRlOTliYw",
  "value":[
    {
      "attributes":[
        {
          "name":"ContentType",
          "value":"Symbology SEDOL"
        }
      ],
      "availableFrom":"2023-11-12T16:15:14Z",
      "availableTo":"2023-12-12T16:15:14Z",
      "bucketName":"bulk-ESG",
      "contentFrom":"1970-01-01T00:00:00Z",
      "contentTo":"2023-11-12T15:55:00Z",
      "created":"2023-11-12T16:15:14Z",
      "files":[
        "4c88-afd6-e880c53b-b2fb-ecd47fc3297a"
      ],
      ...
    },
    {
      "attributes":[
        {
          "name":"ContentType",
          "value":"ESG Sources"
        },
        {
          "name":"ResultCount",
          "value":"104064"
        }
      ],
      "availableFrom":"2023-11-26T17:03:58Z",
      "availableTo":"2023-12-10T17:03:57Z",
      "bucketName":"bulk-ESG",
      "contentFrom":"2023-11-19T16:45:00Z",
      "contentTo":"2023-11-26T16:45:00Z",
      "created":"2023-11-26T17:03:58Z",
      "files":[
        "4e67-a909-938ec235-a37b-f1386462f093"
      ],
      ...
    }
  ]
}
```
There is a ```@nextLink``` node contains the URL for requesting the next page of results with the following HTTP request.

```HTTP
GET {@nextLink URL}, HTTP/1.1
Host: api.refinitiv.com
Authorization: Bearer <Access Token>
```

Example of Python code:

```Python
if '@nextLink' in response.json():
    next_link = response.json()['@nextLink']
    #step 2.5 - list Package IDs from bucket name - with pageSize 2 - navigate to next link
    CFS_url = f'{RDP_HOST}{next_link}'
    
    try:
        response = requests.get(CFS_url, headers={'Authorization': f'Bearer {access_token}'})
    except requests.exceptions.RequestException as exp:
        print(f'Caught exception: {exp}')
    
    if response.status_code == 200:  # HTTP Status 'OK'
        print('Receive list Package IDs from RDP APIs')
    else:
        print(f'RDP APIs: CFS request failure: {response.status_code} {response.reason}')
        print(f'Text: {response.text}')
```

Result:
```JSON
{
  "@nextLink":"/file-store/v1/file-sets?bucket=bulk-ESG&skipToken=ZmlsZXNldElkPTQwNGUtNGVjMC0zNDc2Y2YyYy04OTEzLWI1NmE3NTE0MzBkNA&pageSize=2",
  "value":[
    {
      "attributes":[
        {
          "name":"ContentType",
          "value":"ESG EU BMR"
        }
      ],
      "availableFrom":"2023-11-19T16:19:04Z",
      "availableTo":"2023-12-03T16:19:04Z",
      "bucketName":"bulk-ESG",
      "contentFrom":"1970-01-01T00:00:00Z",
      "contentTo":"2023-11-19T15:55:00Z",
      "created":"2023-11-19T16:19:04Z",
      "files":[
        "45da-1478-f3a8e6e6-8be9-259a707e006c"
      ],
      "...
    },
    {
      "attributes":[
        {
          "name":"ContentType",
          "value":"ESG Scores"
        }
      ],
      "availableFrom":"2023-11-26T17:08:15Z",
      "availableTo":"2023-12-10T17:08:15Z",
      "bucketName":"bulk-ESG",
      "contentFrom":"1970-01-01T00:00:00Z",
      "contentTo":"2023-11-26T16:25:00Z",
      "created":"2023-11-26T17:08:15Z",
      "files":[
        "4a9e-47f3-a7c7a831-bf07-3ca70634d15e"
      ],
      ...
    }
  ]
}
```
### Step 3: Listing the Filesets of the Bulk ESG Data with the packageId

The next step is calling the CFS API with the bucket name and package Id to list all FileSets using **the package Id**. The API endpoint is ```/file-store/v1/file-sets?bucket={bucket-name}&packageId={packageId}```

The HTTP Request structure is as follows:

``` HTTP
GET /file-store/v1/file-sets?bucket={bucket-name}&packageId={packageId} HTTP/1.1
Host: api.refinitiv.com
Authorization: Bearer <Access Token>
```
The example bucket names for RDP content set are as follows:

|              Content             |                  Bucket Name                 |              Example of Package ID             |
|:--------------------------------:|:-------------------------------------------:|:---------------------------------------------:|
| Financial Markets Reference Data | bulk-FMRD                                    | 4d48-d7ff-edcc3d38-8243-a4f7517962b8           |
| Symbology                        | bulk-Symbology                               | 4c80-73a0-fcef949b-bfde-2b9b8117cfb0           |
| ESG                              | bulk-ESG                                     | 4288-ebb6-93372235-acb2-89882a826af1           |
| ESG - Point in Time              | bulk-ESG                                     | 4173-aec7-8a0b0ac9-96f9-48e83ddbd2ad           |
| Tick History                     | TICKHISTORY_VBD_NO_EMBARGO                   | 4c01-ab9e-db594a31-a8f5-5b7852ec4638           |
| Green Revenue                    | bulk-GreenRevenue                            | Summary: 4e94-6d63-fea034dc-90e2-de33895bd4e9  |
| Green Revenue                    | bulk-GreenRevenue                            | Standard: 4316-d43b-81c40763-8e6a-0dbec8162ab1 |
| Starmine                         | STARMINE_PREDICTIVE_ANALYTICS_SMARTECON_LIVE | 40d4-1404-58533484-afe8-718650a4e0d4           |

I am using the ```4288-ebb6-93372235-acb2-89882a826af1``` of the ```bulk-ESG``` as an example package id for the Python code.

Python code:
```Python
# pick the packageId you need and set to the packageId variable
packageId = '4288-ebb6-93372235-acb2-89882a826af1'

#step 3 - get file id from bucket name
CFS_url = f'{RDP_HOST}/file-store/v1/file-sets?bucket={bucket_name}&packageId={packageId}'

try:
    response = requests.get(CFS_url, headers={'Authorization': f'Bearer {access_token}'})
except requests.exceptions.RequestException as exp:
    print(f'Caught exception: {exp}')

if response.status_code == 200:  # HTTP Status 'OK'
    print('Receive FileSets list from RDP APIs')
else:
    print(f'RDP APIs: CFS request failure: {response.status_code} {response.reason}')
    print(f'Text: {response.text}')
```
The FileSets response message from the API is as follows:

```JSON
{
  "attributes":[
    {
      "name":"ContentType",
      "value":"ESG Raw Full B"
    }
  ],
  "availableFrom":"2023-11-12T17:17:33Z",
  "availableTo":"2023-11-26T17:17:32Z",
  "bucketName":"bulk-ESG",
  "contentFrom":"1970-01-01T00:00:00Z",
  "contentTo":"2023-11-12T16:05:00Z",
  "created":"2023-11-12T17:17:33Z",
  "files":[
    "4544-874e-9da0efa0-8051-c734a79d5c61",
    "4c35-1775-c1a590ea-8376-ac6c1546b908"
  ],
  "id":"401f-b3a2-1650edf4-ae9c-e65ea076e128",
  "modified":"2023-11-12T17:40:28Z",
  "name":"Bulk-ESG-Global-Raw-Full-SchemeB-v1-Env-Jsonl-Init-2023-11-12T16:11:09.024Z",
  "numFiles":2,
  "packageId":"4288-ebb6-93372235-acb2-89882a826af1",
  "status":"READY"
}
```

The File ID is in the ```files``` array above. I am demonstrating just one file (```4c35-1775-c1a590ea-8376-ac6c1546b908```).

### Step 4: Get the Bulk file URL on AWS S3

The next step is getting the file URL on Amazon AWS S3 service with the RDP ```/file-store/v1/files/{file ID}/stream``` endpoint.

The HTTP Request structure is as follows:

``` HTTP
GET /file-store/v1/files/{fileId}/stream?doNotRedirect=true HTTP/1.1
Host: api.refinitiv.com
Authorization: Bearer <Access Token>
```

The File URL is in the ```url``` attribute of the response message.

Python code:
```Python
# try just one file
file_id = response.json()['value'][0]['files'][1] #'4c35-1775-c1a590ea-8376-ac6c1546b908'

#step 4 - get file URL from file id
FileID_url = f'{RDP_HOST}/file-store/v1/files/{file_id}/stream?doNotRedirect=true'

try:
    response = requests.get(FileID_url, headers={'Authorization': f'Bearer {access_token}'})
except requests.exceptions.RequestException as exp:
    print(f'Caught exception: {exp}')

if response.status_code == 200:  # HTTP Status 'OK'
    print('Receive File URL from RDP APIs')
else:
    print(f'RDP APIs: CFS request failure: {response.status_code} {response.reason}')
    print(f'Text: {response.text}')

file_url = response.json()['url']
print(file_url)
```
Result
```bash
'https://a206464-bulk-esg.s3.amazonaws.com/Bulk-ESG-Global-Raw-Full-SchemeB-v1/2023/11/12/Bulk-ESG-Global-Raw-Full-SchemeB-v1-Env-Init-2023-11-12T16%3A11%3A09.024Z-part0.jsonl.gz?x-request-Id=f1acbb88-a85c-4ad2-95d5-48a20b774fd6&x-package-id=4288-ebb6-93372235-acb2-89882a826af1&x-client-app-id=b4842f3904fb4a1fa18234796368799086c63541&x-file-name=Bulk-ESG-Global-Raw-Full-SchemeB-v1-Env-Init-2023-11-12T16%3A11%3A09.024Z-part0.jsonl.gz&x-fileset-id=401f-b3a2-1650edf4-ae9c-e65ea076e128&x-bucket-name=bulk-ESG&x-uuid=GESG1-178570&x-file-Id=4c35-1775-c1a590ea-8376-ac6c1546b908&x-fileset-name=Bulk-ESG-Global-Raw-Full-SchemeB-v1-Env-Jsonl-Init-2023-11-12T16%3A11%3A09.024Z&x-event-external-name=cfs-claimCheck-download&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEOn%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCLBeDWjpDLBL3JDJNeGkbKWbdsIFxtNlAfR6n2aCMLeAIgHXdlyuG9WPIpGySOcsl82UmRLELracnxxFBbBvtJCzQqmgIIQhAEGgw2NDIxNTcxODEzMjYiDJKbrz%2Fv47u856n3iCr3AcOtX7SgriBm8XZK%2FObxziVADHu5Mm6G56tmDz5mTP19NcP29i3Mk5ZPthSzfUBNE7AWflhMaSXsjoLL4jgDtC1YXa0uO0crbr7JLjzUZm%2B7E6Y6bXa%2B7NsBWeGWB8U9qz45xUrV34cxPTeG612H%2BW3VQ4Urjja3RoV4PNpATQtBJLRCE3uixRJXLhyMSWggsAUbFYhllkJt6Zhj3XvR3QzFzOBt45Dl3Ur5SRtcSMNVntcEf3CbN8jsBtYbeGrjynGjKyeHZwEbaBuqkAwohTITUkr9EhZ6pi%2F948aTB3AeMlXoCJymOLQzHbgFErqNVHvRpAxyk60wuZ78qgY6nQHRLeYyAANubgCbXUDwE7EGHSWJEfUZv6%2ByRF3T%2FfiWOUNXWSbGqilkfWdUDtZf6z0%2FZBcZ9xyJGbb9ayHTJrermcTIeUoT16TqpjTAOq3ddVlVmusQrGbbIjmNv2LaSGrcJBPIg0jqGV6Z0zwlpC7W%2BRjsJlaBFWRO52srYq4LqounhMHQr20nqUDX37Nq7c4gJXuyhC8WmEFJTkg3&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20231123T083714Z&X-Amz-SignedHeaders=host&X-Amz-Expires=21599&X-Amz-Credential=ASIAZLA4M7GHGKC4VFSX%2F20231123%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=28143f64524165b3906d8bf6181f1c5e44163c4f9a6958f5246a4472a8c1ba04'
```

### Step 5: Downloading the file

Based on the S3 ```file_url``` above, the actual file name is *Bulk-ESG-Global-Raw-Full-SchemeB-v1-Env-Init-2023-11-12T16_11_09.024Z-part0.jsonl.gz*. So you need to replace the escape character ```%3A``` with ```_``` (underscore) character.

**Note**: If you cannot download the file, please wait for a while and then retry download the file from the URL (```file_url```). Please do not flush the download requests. I am demonstrating with the [polling2](https://pypi.org/project/polling2/) library.

Python code:

```Python
#step 5 - Downlaod file
import polling2

zipfilename = file_url.split("?")[0].split("/")[-1].replace("%3A","_")
print(f'Downloading File {zipfilename} ...')

def test_result(response):
    return response.status_code == 200

try:
    response = polling2.poll(lambda: requests.get(file_url), 
                            step = 10,
                            poll_forever = True,
                            check_success= test_result)
except requests.exceptions.RequestException as exp:
    print(f'Caught exception: {exp}')

if response.status_code == 200:  # HTTP Status 'OK'
    print('Receive File Successfully')
    open(zipfilename, 'wb').write(response.content)
    print(f'{zipfilename} Saved')
else:
    print(f'RDP APIs: Request file failure: {response.status_code} {response.reason}')
    print(f'Text: {response.text}')
```

Result:
```bash
Downloading File Bulk-ESG-Global-Raw-Full-SchemeB-v1-Env-Init-2023-11-12T16_11_09.024Z-part0.jsonl.gz ...
Receive File Successfully
Bulk-ESG-Global-Raw-Full-SchemeB-v1-Env-Init-2023-11-12T16_11_09.024Z-part0.jsonl.gz Saved
```

Now you get the CFS Bulk file that you can extract and read the file.

That is all for the RDP CFS Bulk File workflow.

### Step 6: Refresh Token with RDP APIs

Before the session expires (based on the ```expires_in``` parameter, in seconds) , an application needs to send a Refresh Grant request message to RDP Authentication service to get a new access token before further request data from the platform.

The API requires the following access credential information:
- Refresh Token: The current Refresh Token value from the previous RDP Authentication call
- Client ID: This is also known as ```AppKey```, and it is generated using an App key Generator. This unique identifier is defined for the user or application and is deemed confidential (not shared between users). The client_id parameter can be passed in the request body or as an “Authorization” request header that is encoded as base64.
- Grant Type ```refresh_token```: This is for getting a new Access Token. 

The HTTP request for the RDP APIs Authentication service is as follows:

``` HTTP
POST /auth/oauth2/v1/token HTTP/1.1
Accept: */*
Content-Type: application/x-www-form-urlencoded
Host: api.refinitiv.com:443
Content-Length: XXX

refresh_token={current_refresh_token}
&grant_type=refresh_token
&client_id=RDP_APP_KEY
```

Once the authentication success, the function gets **access_token**, **refresh_token**, and **expires_in** from the RDP Auth service response message the same as the previous RDP Authentication call. An application must keep those value for the next Refresh Token call.

#### Caution: API Limit

The RDP Authentication service has the API limit described on the [RDP APIs: Limitations and Guidelines for the RDP Authentication Service](https://developers.lseg.com/en/article-catalog/article/limitations-and-guidelines-for-the-rdp-authentication-service) article.  If the application flushes the authentication request messages (both ```password``` and ```refresh_token``` grant_type) beyond the limit, the account will be blocked by the API Gateway. 

Python code:
```Python
#step 6 - Refreshing Token

# Send HTTP Request
auth_url = f'{RDP_HOST}/auth/oauth2/v1/token'
payload = f'grant_type=refresh_token&client_id={clientId}&refresh_token={refresh_token}'
try:
    response = requests.post(auth_url, 
                             headers = {'Content-Type':'application/x-www-form-urlencoded'}, 
                             data = payload, 
                             auth = (clientId, '')
                )
except requests.exceptions.RequestException as exp:
    print(f'Caught exception: {exp}')

if response.status_code == 200:  # HTTP Status 'OK'
    print('Refresh Token success')
    access_token = response.json()['access_token']
    refresh_token = response.json()['refresh_token']
    expires_in = int(response.json()['expires_in'])

if response.status_code != 200:
    print(f'RDP authentication failure: {response.status_code} {response.reason}')
    print(f'Text: {response.text}')
```

### Step 7: Revoke Token to ending the session.

This revocation mechanism allows an application to invalidate its tokens if the end-user logs out, changes identity, or exits the respective application. Notifying the authorization server that the token is no longer needed allows the authorization server to clean up data associated with that token (e.g., session data) and the underlying authorization grant.

The API requires the following HTTP Header and Credential parameter information:
- Header: 
    * Authorization = ```Basic <App Key in Base64 format>```

Please notice *the space* between the ```Basic``` and ```App Key in Base64 format``` values.
- Body parameter
    * token: The current ```Access Token``` value from the previous RDP Authentication call

The HTTP request for the RDP APIs Authentication service is as follows:

``` HTTP
POST /auth/oauth2/v1/revoke HTTP/1.1
Accept: */*
Content-Type: application/x-www-form-urlencoded
Host: api.refinitiv.com:443
Authorization: Basic <App Key in Base64>
Content-Length: XXX

token={current_Access_token}
```

Python code:
```Python
#step 7 - Revoking Token
import base64

clientId_bytes = clientId.encode('ascii')
base64_bytes = base64.b64encode(clientId_bytes)
clientId_base64 = base64_bytes.decode('ascii')

# Send HTTP Request
auth_url = f'{RDP_HOST}/auth/oauth2/v1/revoke'
payload = f'token={access_token}'
try:
    response = requests.post(auth_url, 
                             headers = {
                                 'Content-Type':'application/x-www-form-urlencoded',
                                 'Authorization': f'Basic {clientId_base64}'
                             }, 
                             data = payload, 
                             auth = (clientId, '')
                )
except requests.exceptions.RequestException as exp:
    print(f'Caught exception: {exp}')

if response.status_code == 200:  # HTTP Status 'OK'
    print('Revoke Token success')
if response.status_code != 200:
    print(f'RDP authentication failure: {response.status_code} {response.reason}')
    print(f'Text: {response.text}')
```

## Next Steps

You may interested in the following resources for more detail about the CFS data usage:
- [Find environmental footprint of your bond portfolio](https://developers.lseg.com/en/article-catalog/article/Environmental_footprint_of_bond_portfolio) article
- [RDP APIs Green Revenues CFS Bulk file Workflow](https://github.com/LSEG-API-Samples/Example.RDP.Python.GreenRevenuesBulk) - a dedicate Green Revenue CFS workflow
- [RDP APIs ESG CFS Bulk file Workflow](https://github.com/LSEG-API-Samples/Example.RDP.Python.ESG.PointinTimeBulk) - a dedicate ESG CFS workflow

And much more on the [Developer Portal](https://developers.lseg.com/en) website.

[To Be Done]

## <a id="references"></a>References

That brings me to the end of my generic CFS Bulk file workflow project. For further details, please check out the following resources:

* [Refinitiv Data Platform APIs page](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) on the [Refinitiv Developer Community](https://developers.lseg.com/) website.
* [Refinitiv Data Platform APIs Playground page](https://apidocs.refinitiv.com/Apps/ApiDocs).
* [Refinitiv Data Platform APIs: Introduction to the Request-Response API](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/tutorials#introduction-to-the-request-response-api).
* [Refinitiv Data Platform APIs: Authorization - All about tokens](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/tutorials#authorization-all-about-tokens).
* [Limitations and Guidelines for the RDP Authentication Service](https://developers.lseg.com/en/article-catalog/article/limitations-and-guidelines-for-the-rdp-authentication-service) article.
* [Getting Started with Refinitiv Data Platform](https://developers.lseg.com/en/article-catalog/article/getting-start-with-refinitiv-data-platform) article.
* [CFS API User Guide](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/documentation#cfs-api-user-guide).


For any questions related to Refinitiv Data Platform APIs, please use the [RDP APIs Forum](https://community.developers.refinitiv.com/spaces/231/index.html) on the [Developers Community Q&A page](https://community.developers.refinitiv.com/).




