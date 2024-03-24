# Token Ranges

To see the tokens for each node on our cluster, we use the nodetool ring command:
`docker exec -it scylla-node1 nodetool ring`

Another way to show the tokens present on a specific node is with the describing command using the keyspace as a parameter:
`docker exec -it scylla-node1 nodetool describering scyllau`



