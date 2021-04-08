---
title: Store data
description: Lorem ipsum. 
---

# Store data

Now we can get started with _storing_ the stuff we want to archive on Filecoin. We first need to prepare it into a format that Lotus can manage.

:::danger
Do not store personal data on the Filecoin network, even if it's encrypted. Access control is on the roadmap, but until then only store public and static data on the Filecoin network. 
:::

## Prepare your data

For the purposes of this tutorial, we're going to store the [ISS_COORDS_2021-03-25 dataset](https://data.nasa.gov/Space-Science/ISS_COORDS_2021-03-25/qti9-kibp) from NASA on the Filecoin network. If you've got some other data you'd prefer to store on Filecoin, go ahead and use that!

1. Download the `ISS_COORDS_2021-03-25` dataset:

    ```shell
    cd ~
    curl -o ISS_COORDS_DATASET.tar https://ipfs.io/ipfs/QmWNi1ygVKBqbZ2E3RqRzQFbSmpzjpf3WTeQebakLJWZSu
    ```

1. Create a folder called `filecoin-payload-folder` to hold the payload.

    ```shell
    mkdir filecoin-payload-folder
    ```

1. Extract the NASA dataset into `filecoin-payload-folder`:

    ```shell
    tar -xvf ISS_COORDS_DATASET.tar -C filecoin-payload-folder
    ```

1. Create a block of random data to _pad_ the total size of our payload:

    If you're on macOS run:

    ```shell
    dd if=/dev/urandom of=padded-4gb-file.bin bs=1m count=4096 
    ```

    If you're on Linux run: 

    ```shell
    dd if=/dev/urandom of=padded-4gb-file.bin bs=1M count=4096
    ```

1. Move `padded-4gb.file.bin` into the `filecoin-payload-folder`:

    ```shell
    mv padded-4b-file.bin filecoin-payload-folder
    ```

1. Pack everything into a `.tar` file:

    ```shell
    tar -cvf ~/filecoin-payload.tar ~/filecoin-payload-folder
    ```

We now have our payload file ready to be stored using the Filecoin network.

## Add data to Lotus

We need to tell our Lotus lite-node which file we want to store using Filecoin.

1. Import the payload into the `lotus daemon` using the `import` command: 

    ```shell
    lotus client import ~/filecoin-payload.tar 
    ```

    Lotus creates a distributed-asyclic-graph (DAG) based off the payload. This process takes a few minutes. Once's it's complete Lotus will output the root CID of the payload.

    ```shell
    > Import 3, Root bafykb...
    ```

1. Make a note of the CID `bafykb...`; we'll use it in an upcoming section.

Now that Lotus knows which file we want to use, we can create a deal with a Filecoin miner to store our data!

## Find a miner that meets your needs

Before we can store data, we need to select a suitable miner. The Filecoin network allows data storage miners to compete with one another by offering different terms for pricing, acceptable data sizes, and other important deal parameters. 

There are a few resources available for finding dependable miners that will accept your data. 

### Find a miner through the MinerX program

1. Go to [plus.fil.org/miners](https://plus.fil.org/miners/).
1. Using the table, find a miner that suits your needs. For the sake of this tutorial, look for a miner that is:
    a. Close to you.
    a. Offering verified-data deals for 0 FIL.
 
1. Once you have found a suitable miner, copy their `miner_id` from the **Miner ID** column:

    ![](./images/miner-x-listings.png)

    Some miners list multiple miner IDs. For these miners, just copy one of the IDs:

    ![](./images/miner-with-multiple-miner-ids.png)

1. Write down ID of the miner you want to use. We'll be referring to it in the next section.

### Use a miner reputation system

The MinerX program is a great resource, but it represents a small portion of the entire Filecoin mining community. Filecoin reputation systems like [FilRep](https://filrep.io) can help compare miners based on their past performance and provide useful information about the deal parameters that a miner will accept.

Using FilRep you can see and compare important miner parameters and metrics including location, storage power in the network, pricing, and overall success rate. The column selection widget lets you see even more details, including the minimum and maximum file sizes that a miner will accept:

![](./images/filrep-select-columns.png)

Miner reputation systems can be used alongside programs like MinerX to find the best miner for your needs. To see what FilRep has to say about a member of the MinerX program, just paste the **Miner ID** from the MinerX list into the search box at [filrep.io](https://filrep.io).

## Create a deal 

To complete this section you need the data CID you received after running `lotus client import` and the ID of a miner you want to use.

1. Start the interactive deal process:

    ```shell
    lotus client deal
    ```

    The interactive deal assistant will now ask you some questions.

1. Specify the CID of the file you want to backup on Filecoin. This is the CID that you got from running `lotus client import ~/filecoin-payload.tar`:

    ```shell
    Data CID (from lotus client import): bafykbz...
    ```

1. Wait for Lotus to finish building the `.car` file.

    ```shell
    > .. calculating data size 
    ```

    The duration of this process depends on the size of your file and the specification of your Lotus node. Lotus took around 25 minutes to build the `.car` file of a ~7.5GB file with an 8-core CPU and 16GB RAM.

1. Enter the number of days you want to keep this file on Filecoin for. The minimum is 180 days:

    ```shell
    > Deal duration (days): 365 
    ``` 

1. Tell Lotus whether or not this is a Filecoin+ deal. Since we signed up to Filecoin+ and added some DataCap to our wallet in an earlier step, we'll select `yes` here:

    ```shell
    > Make this a verified deal? (yes/no): yes
    ```

1. Enter the miner ID from the previous section: 

    ```shell
    > Miner Addresses (f0.. f0..), none to find: f3141592654 
    ```

    <!-- TODO: find out what happens after you throw in a MINER_ID. -->

1. Enter `0` when asked how much FIL we are willing to spend for this storage deal:

    ```shell
    > Maximum budget (FIL): 0.5
    ```

    <!-- TODO: find out what happens after you throw in a MINER_ID. -->

    Normally, we would enter a value of around `0.5 FIL` here. However, since we picked a miner that is accepting 0 FIL deals for verified storage deals, and we are a verified client, then we don't actually need to spend any FIL here!

1. Specify how many miners you want your file to be replicated over. The default it one 

    ```shell
    Deals to make (1): 1
    ```

1. Confirm your transaction:

    ```shell
    > -----
    > Proposing from f136b5uqa73jni2rr745d3nek4uw6qiy6b6zmmvcq
    >         Balance: 2 FIL
    > 
    > Piece size: 8GiB (Payload size: 7.445GiB)
    > Duration: 7200h0m0s
    > Total price: ~0 FIL (0 FIL per epoch)
    > Verified: true
    > 
    > Accept (yes/no): yes
    ```

1. Lotus will returns a **Deal CID**:

    ```shell
    .. executing
    Deal (f023978) CID: bafyreict2zhkbwy2arri3jgthk2jyznck47umvpqis3hc5oclvskwpteau
    ```

## Wait for the deal to complete

We need to wait for the miner to accept our deal and _seal_ the data. This process can take up to 24 hours to complete, depending on how much data we asked the miner to store.

1. List successful and pending deals by using the `list-deals` command:

    ```shell
    lotus client list-deals
    ```
    <!-- TODO: show what happens when you list the deals. -->

    If you cannot see your deal in the list, the deal may have failed. Use `--list-failed` to see failed deals:

    ```shell
    lotus client list-deals --list-failed
    ```

    <!-- TODO: show what happens when you list failed the deals. -->

## Next steps

Now that you've added some data onto the Filecoin network [we can move into retrieving data →](./retrieve-data)
