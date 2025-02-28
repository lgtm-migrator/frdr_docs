## Adding a repository to the FRDR Discovery Service

The Federated Research Data Repository (FRDR) Discovery Service is designed to index research data repositories in Canada. Repositories indexed by FRDR include: 

* data repositories hosted at Canadian universities, which serve as a deposit point for researchers working at those institutions; 
* government data portals; and 
* domain-specific repositories hosted at larger or independent research centres, which are known to researchers working in a given field.

The FRDR staff are working on adding new repositories to the index. Criteria which make a repository more likely to be prioritized include the following:

* Support for one of the metadata API formats currently implemented in the FRDR harvester. Currently, this is OAI-PMH, CKAN, CSW, MarkLogic, OpenDataSoft, Socrata, and certain repositories with Google Sitemaps. Support for additional formats may be added in the future.
* If a repository holds more than just research data -- for example, some university institutional repositories also hold theses and pre-print articles -- it should have some method of querying for only research data.
* A plausible workflow for plaintext/keyword search and retrieval of datasets from the repository.
* The existence of a reliable point of contact at the host institution for resolving technical and/or metadata issues.

For more information, please see the [Metadata Harvesting Policy](/policies/en/metadata_harvesting/).
	
If you would like to include your repository’s metadata in FRDR, please contact us at [support@frdr-dfdr.ca](mailto:support@frdr-dfdr.ca?subject=query%20re%20harvesting%20a%20repository) to discuss the process.

## ProQuest Central Discovery Index
All datasets harvested by the Federated Research Data Repository (FRDR) are now available via the ProQuest Central Discovery Index (CDI). Here are the instructions to make FRDR discoverable  in ProQuest interfaces, including Summon, Primo, and Alma. By following the instructions, you will make all of FRDR's data—more than 117,000 datasets—visible for your users.

1. Navigate to your library’s ProQuest Client Center: https://clientcenter.serialssolutions.com/CC/Login/Default.aspx <br/>
2. Search for Database > Name Contains > FRDR <br/>
<img src="/docs/img/screenshots/feed_proquest/1.png" alt="ProQuest Client Center"/><br/>
3. Open the record found (code:MFDEJ). Click Edit and enable the inclusion in your ProQuest subscriptions, e.g. Summon.
<br/><img src="/docs/img/screenshots/feed_proquest/2.png" alt="ProQuest Subscription"/><br/>
4. From that point on, all FRDR's records will be displayed for your users.<br/>

Please contact support@frdr-dfdr.ca if you need assistance.

We are aware that Alma users are unable to add the FRDR collection at this time. More information and updated instructions will be added as soon as we have them.

## For developers
This documentation is for users who want to develop applications or web sites that use FRDR technology.

### Search API

FRDR discovery uses [Globus Search Platform](https://docs.globus.org/api/search/) for its back end, which itself is based on Elasticsearch. The FRDR harvester consumes feeds from various repositories across Canada, including the FRDR repository itself, and creates entries in the FRDR search index.

The Search API has two forms. Simple queries (for example, those not defining facets) may be accessed using the GET form for ease. More complicated queries may use the POST form to specify richer requirements. In either case, the result format is the same.

----

*GET Method*

URL: https://search.api.globus.org/v1/index/ea384560-88cf-4e6e-b379-956793192fe8/search?q=term

Query Parameters:

* (required) **q**: A string; the query to be executed.
* (optional) **offset**: Zero based offset into the result set, used for paging. Default 0, Maximum 10,000.
* (optional) **limit**: Maximum number of results to return. Default 10.
----

*POST Method*

URL: https://search.api.globus.org/v1/index/frdr/search

Query Parameters: None

POST Parameters:

* (required) Request Body: a GSearchRequest JSON document

Example GSearchRequest:

```javascript
{
  "@datatype": "GSearchRequest",
  "@version": "2016-11-09",
  "q": "(bears AND black) OR (trees AND green)",
  "offset": "20",
  "limit": "10",
  "advanced": true
}
```

----

*Query Syntax*

Two separate syntaxes for specifying the query are supported: standard and advanced. The standard query allows only for basic text matching. All queries will be processed, and results which best match the input will be provided. This form is typically appropriate for environments where end-users will be providing the content of the query string. The advanced syntax (used when the advanced value is set to true) supports ranges, regular expressions, matching on particular fields and other more sophisticated capabilities. The advanced syntax is subject to errors in parsing such as badly formed ranges or mis-named fields. As such, it is more appropriate for use when queries are generated by machine or as a result of assisting an end-user in building a query. The syntax is based on the [Elastic search query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax) with the following exceptions:
1. Wildcards in field names are not allowed. So, for example, the query "book.\*:(quick brown)" is not permitted.
2. The _missing_ and _exists_ query terms are not permitted.


### Item Deposit

**Note: these instructions are for Linux and Mac.  Windows instructions are being developed.**

You can use the [Globus Python Client](http://globus-sdk-python.readthedocs.io/en/stable/) to interact with Globus services, including FRDR.  Before starting, you must log into FRDR with the user account that you will use for deposit, and you should also make sure you have been given permission to deposit.

Install [Python](https://www.python.org/downloads/).

Install Virtualenv:

```bash
sudo pip install virtualenv
```

Install the Globus SDK python module:

```bash
sudo pip install globus-sdk
```

Obtain a copy of the REST API Client code by [contacting us](/repo/contactus).  This code is currently not publicly released.

Change directory to where the REST API Client code was downloaded, and then install it:

```bash
make install
```

To see the commands available to you, you can execute:
```bash
./globus-publish-client --help
```

Run the REST API Client code for the first time:

```bash
./globus-publish-client --service-url "https://demo.frdr-dfdr.ca/v1/api" --list-schemas
```
You will be instructed to copy and paste a URL into your web browser. The browser you use should already be logged in, as the user that will be depositing the items. You can then copy your authentication token back into the command line. You will end up with a JSON file that contains auth tokens in your home directory (default mode 0600).

To submit an item, the dataset bitstreams must already exist on a Globus endpoint somewhere accessible to the user that you logged in as above. The item metadata must also exist in the local filesystem in JSON format. Please contact a curator at support@frdr-dfdr.ca to receive a copy of the template.

You need to determine the storage group that the item will be deposited to.  You can obtain the list with this command:

```
./globus-publish-client --service-url "https://demo.frdr-dfdr.ca/v1/api" --list-collections
```

You will see a list like this:
```
[
  {
    "name": "Storage Group 1",
    "id": 3
  },
  {
    "name": "Storage Group 2",
    "id": 5
  }
]
```
You are interested in the ID of the storage group, it will be needed in the command to submit the item.  In this case, we will choose ID 3.

The command to submit the item (all arguments shown are required) is this:

```bash
./globus-publish-client \
  --service-url "https://demo.frdr-dfdr.ca/v1/api" \
  --create-submission --collection-id 3 \
  --metadata-file item.json \
  --transfer-data \
  --data-endpoint endpoint-uuid-goes-here --data-directory "/my_data/"
```

The UUID must point to a valid and running Globus Connect endpoint, the path must exist, and the depositing user must have permission to read data from that path on that endpoint.

This command will return with JSON similar to this:

```javascript
Dataset record created:
{
  "dc.date.issued": "2018-02-27",
  "dc.contributor.author": "Smith, Jane",
  "globus.shared_endpoint.path": "/1/unpublished/publication_64/",
  "dc.date.accessioned": "2018-02-27",
  "globus.shared_endpoint.name": "49eba39a-9987-11e7-ac63-22000a92523b",
  "id": 64,
  "dc.title": "My Dataset Title",
  "dc.publisher": "University of Somewhere",
  "dc.date.available": "2018-02-27"
}

Transfer request result: {
  "message": "The transfer has been accepted and a task has been created and queued for execution",
  "resource": "/transfer",
  "code": "Accepted",
  "request_id": "qNFY8ByuN",
  "task_link": {
    "resource": "task",
    "title": "related task",
    "DATA_TYPE": "link",
    "rel": "related",
    "href": "task/f25d0050-1cc2-11e8-b718-0ac6873fc732?format=json"
  },
  "submission_id": "f25d0051-1cc2-11e8-b718-0ac6873fc732",
  "DATA_TYPE": "transfer_result",
  "task_id": "f25d0050-1cc2-11e8-b718-0ac6873fc732"
}
id of transfer task is  f25d0050-1cc2-11e8-b718-0ac6873fc732
```
You need to record the dataset record ID.  In this case, 64.  Wait until you receive an email that your data has been transferred to FRDR, then you can finalize the submission with this command:

```bash
./globus-publish-client --service-url "https://demo.frdr-dfdr.ca/v1/api" --submission-id 64 --submit --wait
```

Once this command returns, you will see the DOI of the submitted item.  Wait 15 minutes for the DOI to resolve, then you can put that into a web browser and view the submission in FRDR.

If this command returned a timeout error, you can ignore it.  However, in order to query the item and determine the DOI at this point, you can execute this command:

```bash
./globus-publish-client --service-url "https://demo.frdr-dfdr.ca/v1/api" --dataset-id 64 --display-dataset

```

After a **submission** has been successfully and completely submitted it is now called a **dataset**.   So you use the same ID as above in the query, but the arguments are now referring to dataset.

Alternately, instead of submitting the item in 2 separate calls, the `--submit` and `--wait` flags can be added to the original command.  This will result in the deposit being started, the dataset files transferred, and the item being submitted and finalized all in one transaction.  However, this does make the script unresponsive while waiting for the dataset to be transferred, so it is really only suitable for very small datasets.
