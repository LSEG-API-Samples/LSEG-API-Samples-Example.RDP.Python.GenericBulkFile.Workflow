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

This project uses RDP access credentials with the CFS Bulk file permission.

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

## <a id="how_to_run"></a>How to run the demo application

The first step is to unzip or download the example project folder into a directory of your choice, then set up Python or Docker environments based on your preference.

### <a id="python_example_run"></a>Run the demo application

1. Open Anaconda Prompt and go to the project's folder.
2. Run the following command in the Anaconda Prompt application to create a Conda environment named *ESG* for the project.
    ``` bash
    (base) $>conda create --name CFS python=3.10
    ```
3. Once the environment is created, activate a Conda *ESG* environment with this command in Anaconda Prompt.
    ``` bash
    (base) $>conda activate CFS
    ```
4. Run the following command to the dependencies in the *CFS* environment 
    ``` bash
    (CFS) $>pip install -r requirements.txt
    ```
5. Once the dependencies installation process is success, create a ```.env``` file with the following content
    ``` INI
    RDP_USERNAME=<Your RDP Username>
    RDP_PASSWORD=<Your RDP Password>
    RDP_APP_KEY=<Your RDP App key>
    ```
5. Then run the following command to start the Jupyter Lab application.
    ``` bash
    (CFS) $>jupyter lab
    ```
6. Open a **RDP-Generic-BULK.ipynb**  file and run each cell to learn the RDP CFS Bulk File workflow step by step.

    ![figure-2](images/notebook_file.png "Notebook file")

## <a id="how_to_run_postman"></a>How to run the Postman collection

1. Open the Postman application
2. Click on the "Collections" tab, then click the "Import" button

    ![figure-3](images/postman1.png)

3. Choose the *RDP CFS Bulk API.postman_collection.json* file, then the RDP CFS BUlk API collection will be imported to Postman

    ![figure-4](images/postman3.png)

4. Click on the "Environments" tab, then click the "Import" button

    ![figure-5](images/postman4.png)

5. Choose the *RDP CFS Bulk Environment.postman_environment.json* file, then the RDP CFS Bulk environment will be imported to Postman
6. Click on the "Collections" tab, then choose the *RDP CFS Bulk environment*. Input the ```USERNAME```, ```PASSWORD```, ```APP_KEY```, and ```BUCKET_NAME``` variables with your RDP credential detail.

    ![figure-6](images/postman5.png)

7. The first step is run the "Step 1" Get Access Token using Password Grant (Machine ID)" request, please make sure that the environment on the top right corner is pointed to the *RDP CFS Bulk Environment*

    ![figure-7](images/postman7.png)

8. You can generate the source code from this Postman request by clicking the ```</>``` button and choose prefer language.

    ![figure-8](images/postman8.png)

9. Once the Once authentication is successful, run the requests from step 2 to step 4

    ![figure-9](images/postman9.png)

10. You can generate the source code from these Postman requests by clicking the ```</>``` button and choose prefer language on each request too.

    ![figure-10](images/postman10.png)

11. If the Access Token is expired, the RDP CFS API requests return "token expired" error message as follows:

    ![figure-11](images/postman11.png)

12. You need to refresh the Access Token by requesting "Step 5: Refresh Access Token using Refresh Token"

    ![figure-12](images/postman12.png)


## Next Steps

You may interested in the following resources for more detail about the CFS data usage:
- [Find environmental footprint of your bond portfolio](https://developers.lseg.com/en/article-catalog/article/Environmental_footprint_of_bond_portfolio) article
- [RDP APIs Green Revenues CFS Bulk file Workflow](https://github.com/LSEG-API-Samples/Example.RDP.Python.GreenRevenuesBulk) - a dedicate Green Revenue CFS workflow
- [RDP APIs ESG CFS Bulk file Workflow](https://github.com/LSEG-API-Samples/Example.RDP.Python.ESG.PointinTimeBulk) - a dedicate ESG CFS workflow

And much more on the [Developer Portal](https://developers.lseg.com/en) website.

## <a id="references"></a>References

That brings me to the end of my generic CFS Bulk file workflow project. For further details, please check out the following resources:

* [Refinitiv Data Platform APIs page](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) on the [Refinitiv Developer Community](https://developers.refinitiv.com/) website.
* [Refinitiv Data Platform APIs Playground page](https://api.refinitiv.com).
* [Refinitiv Data Platform APIs: Introduction to the Request-Response API](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/tutorials#introduction-to-the-request-response-api).
* [Refinitiv Data Platform APIs: Authorization - All about tokens](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/tutorials#authorization-all-about-tokens).
* [Limitations and Guidelines for the RDP Authentication Service](https://developers.refinitiv.com/en/article-catalog/article/limitations-and-guidelines-for-the-rdp-authentication-service) article.
* [Getting Started with Refinitiv Data Platform](https://developers.refinitiv.com/en/article-catalog/article/getting-start-with-refinitiv-data-platform) article.
* [CFS API User Guide](https://developers.lseg.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis/documentation#cfs-api-user-guide)


For any questions related to Refinitiv Data Platform APIs, please use the [RDP APIs Forum](https://community.developers.refinitiv.com/spaces/231/index.html) on the [Developers Community Q&A page](https://community.developers.refinitiv.com/).