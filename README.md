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

### Upload tile to Ops Manager

### Add tile to Ops Manager

### Configure tile

## CF CLI

### Create an isolation segment

### Enable isolation segments for an organization

### Set isolation segments as default for an organization (any space created will be assigned to the isolation segment)

#### Retrieve isolation segment guid, JSON from V3 CAPI

#### Retrieve organization guid, JSON from V3 CAPI

#### Set default isolation segment for an organization

#### Enable isolation segments for an organization

#### Enable isolation segments for an existing space

#### Let's push an application
