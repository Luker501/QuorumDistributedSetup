# Quorum Distributed Setup
How to setup up a distributed Quorum blockchain network with Istanbul consensus from scratch.

1. Build Quorum as described in the [getting set up](https://github.com/jpmorganchase/quorum/wiki/Getting-Set-Up) section - note that Constellation or Tessera is not required for this walkthrough guide. Ensure that PATH contains geth and bootnode. *Note that your geth needs to be the quorum version and should be at least version 1.8 for the following to work*. 
2. Install [istanbul-tools](https://github.com/jpmorganchase/istanbul-tools) and place the `istanbul` binary into PATH.
3. Create a working directory for each of the X number of initial validator nodes. Assign one node to be the lead for network initialisation.
4. Change into the lead node's working directory and generate the setup files for X initial validator nodes by executing `istanbul setup --num X --nodes --quorum --save --verbose` **only execute this instruction once, i.e. not X times**. This command will generate several items of interest: `static-nodes.json`; `genesis.json`; and nodekeys for all the initial validator nodes which will sit in numbered directories (i.e. from 0 to X-1). 
5. Update `static-nodes.json` to include the intended IP and port numbers of all initial validator nodes. In `static-nodes.json`, you will see a different row for each node. For the rest of the installation guide, row Y refers to node Y and row 1 is assumed to correspond to the lead node. Additionally, if you are creating a permissioned network (where only whitelisted nodes can attach to each other), then you need to now copy `static-nodes.json` and rename the copied file `permissioned-nodes.json`.
6. In each node's working directory, create a data directory called `data`, and inside `data` create the `geth` directory.
7. Now we will generate initial accounts for any of the nodes by executing `geth --datadir data account new` in the required node's working directory. The resulting public account address printed in the terminal should be recorded. 
8. To add accounts to the initial block, edit the `genesis.json` file in the lead node's working directory and update the `alloc` field with the account(s) that were generated at previous step.
9. Next we need to distribute the files created in part 4, which currently reside in the lead node's working directory, to all other nodes. To do so, place `genesis.json` in the working directory of all nodes, place `static-nodes.json` and `permissioned-nodes.json` in the data folder of each node and place `Z/nodekey` in node (Z-1)'s `data/geth` directory.
10. Initialize all nodes with `geth --datadir data init genesis.json`
11. Start all nodes and send into background with `PRIVATE_CONFIG=ignore nohup geth --datadir data --permissioned --nodiscover --istanbul.blockperiod 5 --syncmode full --mine --minerthreads 1 --verbosity 5 --networkid 10 --rpc --rpcaddr 0.0.0.0 --rpcport 22000 --rpcapi admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul--emitcheckpoints --port 30303 2>>node.log &` Note that if you are *not* using a permissioned network, then remove the `--permissioned` tag.

Your initial validator nodes will now be operational and you may attach to any node by executing `geth attach data/geth.ipc` in the required node's working directory. 

Note that part 11 starts Quorum without privacy support, as could be evidenced in prefix `PRIVATE_CONFIG=ignore`.
