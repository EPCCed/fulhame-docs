# HPE Apollo 70 Cluster

The Fulhame system is an HPE Apollo 70 cluster comprising 64 compute nodes connected together with a single infiniband fabric.
In addition, there are 2 login nodes that share a common software environment and file-system with the compute nodes, a service node, an admin node, and 3 storage nodes.


## Compute Nodes

Each compute node consists of two 32-core Marvell ThunderX2 processors running at 2.2GHz, and 256GB DDR4 memory running at 2667MHz arranged as 8 DIMMs per socket. The processors on this system are set to use 1 hardware thread (SMT) per core, giving 64 cores per node.


Item                | Value
--------------------------------------|---------------
Processor           | Marvell ThunderX2 (ARMv8)
Clock speed         | 2.2GHz   
Cores per processor | 32
Cores per node      | 64
Vector width        | 128 bit
Double precision FLOPS per cycle | 8
Maximum node DP GFLOP/s | 1126.4
L1 cache (core)     | 32KB
L2 cache (core)     | 256KB
L3 cache (shared)   | 32MB
Memory per node     | 256GB
Memory per core     | 4GB
Memory channels (per proc) | 8
Specified memory bandwidth (per proc) | 160GB/s
Specified memory bandwidth (per core) | 5.0GB/s
Measured memory bandwidth  (per node) | 221.48GB/s


## Support Nodes

There are 2 login nodes, 1 service node, 1 admin node and 3 storage nodes. All nodes are dual socket and have the same processor, DRAM & infiniband configuration as the compute nodes.

## Infiniband Fabric
The fabric connecting all of the nodes is a Mellanox EDR (25Gb/s) Infiniband using a non-blocking fat tree topology.

## Layout

Schematic overview of the system.

