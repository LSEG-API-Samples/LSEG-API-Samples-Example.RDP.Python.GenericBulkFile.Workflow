# A Generic and Step By Step Workflow for RDP Client File Store (CFS) Bulk File API
- version: 1.0.0
- Last update: Nov 2023
- Environment: Jupyter Notebook
- Prerequisite: [Access to RDP credentials](#prerequisite)

Example Code Disclaimer:
ALL EXAMPLE CODE IS PROVIDED ON AN “AS IS” AND “AS AVAILABLE” BASIS FOR ILLUSTRATIVE PURPOSES ONLY. REFINITIV MAKES NO REPRESENTATIONS OR WARRANTIES OF ANY KIND, EXPRESS OR IMPLIED, AS TO THE OPERATION OF THE EXAMPLE CODE, OR THE INFORMATION, CONTENT, OR MATERIALS USED IN CONNECTION WITH THE EXAMPLE CODE. YOU EXPRESSLY AGREE THAT YOUR USE OF THE EXAMPLE CODE IS AT YOUR SOLE RISK.

## <a id="intro"></a>Introduction

This demo application shows the generic workflow of the Refinitiv Data Platform (RDP) CFS Bulk API. The workflow can be applied to any Bucket (ESG, Symbology, Green Revenue, etc). I am demonstrating the workflow in [Python](https://www.python.org/) and [Jupyter](https://jupyter.org/) environment. However, the RDP APIs are the web-based API that any programming langues can connect and consume data from via the HTTP RESTful API. 

## <a id="prerequisite"></a>Prerequisite

Before I am going further, there is some prerequisite, dependencies, and libraries that the project is needed.

### Access to the RDP with the your desire Bulk file permission

This project uses RDP access credentials with the  CFS Bulk file permission.

Please contact your Refinitiv representative to help you with the RDP account and services.

### Internet Access

This demonstration connects to RDP on AWS via a public internet.

### Python and Jupyter Notebook.

This project uses [Python](https://www.python.org/) and [Jupyter](https://jupyter.org/) environment.

The Python [Anaconda](https://www.anaconda.com/distribution/) or [MiniConda](https://docs.conda.io/en/latest/miniconda.html) distribution/package manager is recommended on.

## <a id="whatis_rdp"></a>What is Refinitiv Data Platform (RDP) APIs?

The [Refinitiv Data Platform (RDP) APIs](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) provide various Refinitiv data and content for developers via easy-to-use Web-based API.

RDP APIs give developers seamless and holistic access to all of the Refinitiv content such as Environmental Social and Governance (ESG), News, Research, etc, and commingled with their content, enriching, integrating, and distributing the data through a single interface, delivered wherever they need it.  The RDP APIs delivery mechanisms are the following:
* Request - Response: RESTful web service (HTTP GET, POST, PUT or DELETE) 
* Alert: delivery is a mechanism to receive asynchronous updates (alerts) to a subscription. 
* Bulks:  deliver substantial payloads, like the end-of-day pricing data for the whole venue. 
* Streaming: deliver real-time delivery of messages.

This example project is focusing on the Request-Response: RESTful web service delivery method only.  

![figure-1](images/01_rdp.png "Refinitiv Data Platform content set")

For more detail regarding the Refinitiv Data Platform, please see the following APIs resources: 
- [Quick Start](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/quick-start) page.
- [Tutorials](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/tutorials) page.

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

[To be done]