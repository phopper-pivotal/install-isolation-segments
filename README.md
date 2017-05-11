# Install Isolation Segments on Pivotal Cloud Foundry

## Install latest CF CLI

Download and install the latest (at least >= 6.26) CF CLI from [CF CLI Downloads](https://github.com/cloudfoundry/cli#downloads).

Copy CF CLI binary to a directory that's on our path execution.

```
$ ~/bin/cf -v
cf version 6.26.0+9c9a261fd.2017-04-06
```

## Clone and install isolation segments tile
We will want to download the isolation segments tile from PivNet, clone tile to segments thah align to my requirements, upload and configure tile.

### Download isolation segments tile from PivNet
Download [isolation segments tile](https://network.pivotal.io/products/isolation-segment).

**Note:** you will need to have an account on PivNet to download this tile.

### Download tile replicator
We will need to create at least 1 replicate of the isolation tile. A unique name is given to each tile, one that will be used as the isolation segment we will create later via CF CLI. The replicator can be downloaded from [here](https://network.pivotal.io/products/isolation-segment).

![Isolation Tile Downloads](./images/Isolation_Segment_Tile_Replicator_download_page.png)

Once downloaded, unpack the ZIP file. It will contain three binaries, please use the one respective to the OS where you will be replicating the tile. This may be macOS (Darwin), and so on... **If on macOS or Lunux be sure to set the files to executable.**

### Clone tile
```
replicator \
    -name "name_of_my_isolation_segment" \
    -path /absolute/path/to/tile.pivotal \
    -output /absolute/path/to/output.pivotal
```

**Note:** the "name" as mentioned in the command has a length limitation of 32 characters.

## Upload and configure isolation segments tile
Let's upload the replicated tile, configure the tile, and apply changes
### Upload tile to Ops Manager
In a browser navigate to your Ops Manager URL and login.

Select "Import a Product"

![Import a Product](./images/Import_a_Product.png)

**Note:** tile will take some time to upload

Follow the [Pivotal documents](http://docs.pivotal.io/pivotalcf/1-10/opsguide/installing-pcf-is.html#config) for configuring the tile according to your network settings

**Several Notes**
* When configuring the Networking settings, do not select "Forward SSL to Isolation Segment Router with ERT Router certificates", this is an experimental feature that will be removed in a future version of the tile.
* There will be two network configurations you can choose from;
   * "Forward SSL to Isolation Segment Router with provided certificates" - should be the default for all non-development environments. You can copy in the certificate used on the externl load balancer. 
     * Load balancer can forward encrypted traffic to the Elastic Runtime Router for the isolation segment. Be sure to complete the fields for the Router SSL Termination Certificate and Private Key and Router SSL Ciphers. 
     * If you need to create a new certificate, you can specify a separate wildcard domain that's specific to the isolation segment.
   * "Forward unencrypted traffic to Elastic Runtime Router" - this is a good setting to use in a development environment where load balancing is not required
![Configure Networking](./images/Isolation_Segments_configure_tile.png)

Once configured, apply changes.

## CF CLI
Now that the isolation segments tile has been applied to the environment we can now begin to setup the necessary segments, assign them to org and spaces and then push an application to the newly created isolation segment.

### Create an isolation segment
The name of the isolation segment **must** be the name we assigned to the tile above. Otherwise you will get a `NoCompatibleCell` error when pushing an application. In this example we will call our isolation segment `is-test`, please replace for your environment.

**Please verify the version of CF CLI before proceeding, otherwise we will get errors that `... is not a registered command`.**

```sh
$ cf create-isolation-segment is-test
Creating isolation segment is-test as admin...
OK
```

Let's validate the isolation segment was created successfully

```
$ cf isolation-segments
Getting isolation segments as admin...
OK

name         orgs
shared       
is-test  
```

To list the isolation segment for an org or space;

```
cf org is-org
Getting info for org is-org as admin...

name:                 is-org
domains:              cfapps.haas-46.pez.pivotal.io, tcp.haas-46.pez.pivotal.io
quota:                default
spaces:               dev
isolation segments:   is-test
```

```
cf space dev
Getting info for space dev in org is-org as admin...

name:                dev
org:                 is-org
apps:                scale-demo
services:            
isolation segment:   is-test
space quota:         
security groups:     all_open, default_security_group
```

### Enable isolation segments for an organization
Let's enable this created isolation segment for an organization, note `is-org` is only an example please replace with your applicable organization

```
cf enable-org-isolation is-org is-test
Enabling isolation segment is-test for org is-org as admin...
OK
```

### Set isolation segments as default for an organization (any space created will be assigned to the isolation segment)
For non-test purposes we will want to assign an org isolation segment. Once done applications staged will be auctioned off in our new isolation segment

#### Retrieve isolation segment guid, JSON from V3 CAPI
We will need the isolation segments guid from the JSON result of a query against V3 of CAPI

```json
cf curl "/v3/isolation_segments?names=is-test"   
{
   "pagination": {
      "total_results": 1,
      "total_pages": 1,
      "first": {
         "href": "https://api.cfapps.haas-46.pez.pivotal.io/v3/isolation_segments?names=is-test&page=1&per_page=50"
      },
      "last": {
         "href": "https://api.cfapps.haas-46.pez.pivotal.io/v3/isolation_segments?names=is-test&page=1&per_page=50"
      },
      "next": null,
      "previous": null
   },
   "resources": [
      {
         "guid": "f194b9e4-f91b-4fef-b084-60d239306604",
         "name": "is-test",
         "created_at": "2017-05-11T15:33:57Z",
         "updated_at": "2017-05-11T15:33:57Z",
         "links": {
            "self": {
               "href": "https://api.cfapps.haas-46.pez.pivotal.io/v3/isolation_segments/f194b9e4-f91b-4fef-b084-60d239306604"
            },
            "organizations": {
               "href": "https://api.cfapps.haas-46.pez.pivotal.io/v3/isolation_segments/f194b9e4-f91b-4fef-b084-60d239306604/organizations"
            },
            "spaces": {
               "href": "https://api.cfapps.haas-46.pez.pivotal.io/v3/isolation_segments/f194b9e4-f91b-4fef-b084-60d239306604/relationships/spaces"
            }
         }
      }
   ]
}
```

#### Retrieve organization guid, JSON from V3 CAPI
We will need the organizationss guid from the JSON result of a query against V3 of CAPI

```json
cf curl /v3/isolation_segments/c8538feb-d38a-428d-b6d9-9ee7149385f3/organizations 
{
   "pagination": {
      "total_results": 1,
      "total_pages": 1,
      "first": {
         "href": "https://api.cfapps.haas-46.pez.pivotal.io/v3/isolation_segments/c8538feb-d38a-428d-b6d9-9ee7149385f3/organizations?page=1&per_page=50"
      },
      "last": {
         "href": "https://api.cfapps.haas-46.pez.pivotal.io/v3/isolation_segments/c8538feb-d38a-428d-b6d9-9ee7149385f3/organizations?page=1&per_page=50"
      },
      "next": null,
      "previous": null
   },
   "resources": [
      {
         "guid": "785c432e-46a1-481b-8037-878bdf802477",
         "created_at": "2017-05-11T13:37:02Z",
         "updated_at": "2017-05-11T14:11:59Z",
         "name": "is-org",
         "links": {}
      }
   ]
}
```

#### Set default isolation segment for an organization
We will need the isolation segment guid and organization guid to assemble necessary command to set a default isolation segment for an organization

```
$ cf curl /v3/organizations/[REPLACE with organization_guid]/relationships/default_isolation_segment -X PATCH -d '{ "data": { "guid": "[REPLACE with isolation segment guid]" } }'
```

an example;

```
$ cf curl /v3/organizations/785c432e-46a1-481b-8037-878bdf802477/relationships/default_isolation_segment -X PATCH -d '{ "data": { "guid": "f194b9e4-f91b-4fef-b084-60d239306604" } }'

{
   "data": {
      "guid": "f194b9e4-f91b-4fef-b084-60d239306604"
   },
   "links": {
      "self": {
         "href": "https://api.cfapps.haas-46.pez.pivotal.io/v3/organizations/785c432e-46a1-481b-8037-878bdf802477/relationships/default_isolation_segment"
      }
   }
}
```

#### Enable isolation segments for an existing space
If a space within an org already existed we can assign that space to the isolation segment

```
$ cf set-space-isolation-segment dev is-test
Updating isolation segment of space dev in org is-org as admin...
OK

In order to move running applications to this isolation segment, they must be restarted.
```

#### Let's push an application
It's all setup, so let's stage a container and have it auctioned off to a Diego cell in our new isolation segment

```
$ cf push
```
