# Relayer Setup for GoZ
Guide to connect to the Game of Zones Hub.

## Relayer

```
# add aliases to your preferred shell

# bash/zsh aliases
alias C_AIB='aib-goz-1'
alias C_GOZ='gameofzoneshub-1a'
alias ADDR_AIB='cosmos1dee8te2quczv4gkyckxk9nlr3aefyw77c73pqw'
alias ADDR_GOZ='cosmos1xnc4hwja48e77nms0a49eqkdw36d7dqzrj006n'
alias RPC_GOZ='http://35.233.155.199:26657'

# fish shell aliases
set -Ux C_AIB aib-goz-1
set -Ux C_GOZ gameofzoneshub-1a
set -Ux ADDR_AIB cosmos1dee8te2quczv4gkyckxk9nlr3aefyw77c73pqw
set -Ux ADDR_GOZ cosmos1xnc4hwja48e77nms0a49eqkdw36d7dqzrj006n
set -Ux RPC_GOZ http://35.233.155.199:26657

# remove your old relayer config folder
rm -rf ~/.relayer

# setup a new relayer config folder
rly cfg init

# add at least two zones via their JSON files
rly chains add-dir ./zones/

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
rly q bal $C_GOZ

# create a new path between both zones
rly pth gen -f $C_AIB transfer $C_GOZ transfer $AIB_GOZ

# ensure path exists
rly tx link $AIB_GOZ

# try sending tokens, e.g bits from aib to goz

# for bash/zsh
rly tx transfer $C_AIB $C_GOZ 500bits true $(rly ch addr $C_GOZ)

# for fish
rly tx transfer $C_AIB $C_GOZ 500bits true (rly ch addr $C_GOZ)
```

## debugging relayer issues
```
# relayer errors when trying to send
# you need to make sure when you run `rly tx link $AIB_GOZ`
# you get a checkmark next to channel created, e.g.
Channel created: [aib-goz-1]chan{oigbfuzeen}port{transfer}

# if you created a path 90 minutes ago, you  may need to
# create a new path due to the default path timeout

# client: packet commitment verification failed
# if you get this error you may have unsent tx in the queue
# view unrelayed txs with:
rly q unrelayed $AIB_GOZ

# you may see some unrelayed txs
│{"src":["1","2"]}

# you can relay unsent txs with
rly tx rly $AIB_GOZ

#----------
# err(sdk: out of gas) 
# if you get this error you may have to increase the gas amount
rly ch edit $C_GOZ gas {{ higher_amount }}

# then relay unsent again with 
rly tx rly $AIB_GOZ

#----------
# err(sdk: insufficient funds)
# if you get this error you need to add more tokens to your account
# or alternatively increase the gas amount
# see how much gas a transaction would cost by appending -d 

# for example to see how much sending all unrelayed txs would cost is 
rly tx rly $AIB_GOZ -d
```

## gaiacli
```
# query your balance on the local chain
gaiacli q bank balances $ADDR_AIB

# how to query your balance on another blockchain
gaiacli q bank balances $ADDR_GOZ --node=$RPC_GOZ --chain-id=gameofzoneshub-1a

# send tokens via local RPC
gaiacli tx send $ADDR_AIB $YOUR_AIB_ADDR 1000bits

# how to send tokens via remote RPC
gaiacli tx send $ADDR_GOZ $YOUR_GOZ_ADDR 100000doubloons --node=$RPC_GOZ --chain-id=gameofzoneshub-1a
```
