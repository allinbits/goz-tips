# Relayer Setup for GoZ
Guide to connect to the Game of Zones Hub.

## Relayer

```
# add aliases to your shell (these are for fish shell)
set -Ux C_AIB aib-goz-1
set -Ux C_GOZ gameofzoneshub-1a
set -Ux ADDR_AIB cosmos1dee8te2quczv4gkyckxk9nlr3aefyw77c73pqw
set -Ux ADDR_GOZ cosmos1xnc4hwja48e77nms0a49eqkdw36d7dqzrj006n
set -Ux RPC_GOZ http://35.233.155.199:26657

# remove your old relayer config folder
rm -rf ~/.relayer

# setup relayer config folder
rly cfg init

# add at least two zones via their JSON files
rly chains add-dir ~/goz-zones/

# initialize light clients for both chains
rly lite init -f $C_AIB
rly lite init -f $C_GOZ

# restore aib account key
rly keys restore $C_AIB testkey "mnemonic"
rly ch edit $C_AIB key testkey
rly q bal $C_AIB

# restore goz account key
rly keys restore $C_GOZ testkey "mnemonic"
rly ch edit $C_GOZ key testkey

# create a new path between both zones
rly pth gen -f $C_AIB transfer $C_GOZ transfer $AIB_GOZ

# ensure path exists
rly tx link $AIB_GOZ

# send tokens
rly tx transfer $C_AIB $C_GOZ 500bits true (rly ch addr $C_GOZ)
```

## gaiacli
```
# query your balance on the local chain
gaiacli q bank balances $ADDR_AIB

# how to query your balance on another blockchain
gaiacli q bank balances $ADDR_GOZ --node=$RPC_GOZ
```
