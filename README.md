#Genesis

##Installation
* clone repository
* `cd router`
* `go get`
* `go build`

##Configuration
Configuration options are located in `config.json` in the same directory as the binary

* __builder__: The application to use to build the nodes
* __ssh-user__: The default username for ssh
* __ssh-password__: The default password for ssh
* __vyos-home-dir__: The location to put the vyos script
* __listen__: The socket to listen on 
* __rsa-key__: The location of the ssh private key
* __rsa-user__: The corresponding username for that private key
* __verbose__: Enable or disable verbose mode
* __server-bits__: The bits given to each server's number
* __cluster-bits__: The bits given to each clusters's number
* __node-bits__: The bits given to each nodes's number
* __thread-limit__: The maximum number of threads that can be used for building
* __build-mode__: Can set the build mode to allow for building in "standalone" mode, for demo purposes
* __ip-prefix__: Used for the IP Scheme
* __allow-exec__: Set to true to enable the /exec/ calls. __This is an unsafe option to enable__
* __docker-output-file__: The location instead the docker containers where the clients stdout and stderr will be captured

###Config Environment Overrides
These will override what is set in the config.json file, and allow configuration via
only ENV variables
* `BUILDER`
* `SSH_USER`
* `SSH_PASSWORD`
* `VYOS_HOME_DIR`
* `LISTEN`
* `RSA_KEY`
* `RSA_USER`
* `VERBOSE` (only need to set it)
* `SERVER_BITS`
* `CLUSTER_BITS`
* `NODE_BITS`
* `THREAD_LIMIT`
* `BUILD_MODE`
* `IP_PREFIX`
* ALLOW_EXEC (only need to set it)
* DOCKER_OUTPUT_FILE

###Additional Information
* Config order of priority ENV -> config file -> defaults
* `ssh-user`,`ssh-password` and `rsa-user`, `rsa-key` are both used, starting with pass auth then falling back to key auth


##IP Scheme
We are using ipv4 so each address will have 32 bits.

The following assumptions will be made
* Each server will have a relatively unique `serverId`
* This uniqueness need only apply to servers which will contain nodes which communicate with each other
* There are going to be 3 ip addresses reserved from each subnet
* Nodes in the same docker network are able to route between each other by default

For simplicity, the following variables will be used
* `A` = `ip-prefix`
* `B` = `server-bits`
* `C` = `cluster-bits`
* `D` = `node-bits`

Note the following rules
* `A`,`B`,`C`, and `D` cannot be 0
* ceil(log2(`A`)) + `B` + `C` + `D` <= 32
* `D` must be atleast 2
* (`B`^2) = The maximum number of servers
* (`C`^2) = The number of cluster in a given server
* (`D`^2 - 3) = How many nodes are groups together in each cluster
* (`D`^2 - 3) * (`C`^2) = The max number of nodes on a server
* (`D`^2 - 3) * (`C`^2) * (`B`^2) = The maximum number of nodes that could be on the platform

###What is a cluster?

Each cluster corresponds to a subnet,docker network,and vlan. 
Containers in the same cluster will have minimal latency applied to them. In the majority of cases
it is best to just have one node per cluster, allowing for latency control between all of the nodes.

###How is it all calculated?
Given a node number `X` and a `serverId` of `Y`,
Let `Z` be the cluster number, and the earlier mentioned variables applied
`Z`= `X`uint32(uint32(node)/(1 << conf.NodeBits) - ReservedIps)

##REST API

###GET /servers/
Get the current registered servers
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
{
	"server_name":{
		"addr":(string),
		"iaddr":{
			"ip":(string),
			"gateway":(string),
			"subnet":(int)
		},
		"nodes":(int),
		"max":(int),
		"id":(int),
		"serverID":(int),
		"iface":(string),
		"switches":[
			{
				"addr":(string),
				"iface":(string),
				"brand":(int),
				"id":(int)
			}
		]
	},
	"server2_name":{...}...
}
```
#####EXAMPLE
```
curl -XGET http://localhost:8000/servers/
```

###PUT /servers/{name}
####PRIVATE
Register and add a new server to be 
controlled by the instance
#####BODY
```
{
	"addr":(string),
	"iaddr":{
		"ip":(string),
		"gateway":(string),
		"subnet":(int)
	},
	"nodes":(int),
	"max":(int),
	"id":-1,
	"serverID":(int),
	"iface":(string),
	"switches":[
		{
			"addr":(string),
			"iface":(string),
			"brand":(int),
			"id":(int)
		}
	]
}
```
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
<server id>
```

#####EXAMPLE
```bash
curl -X PUT http://localhost:8000/servers/foxtrot -d \
'{"addr":"172.16.6.5","iaddr":{"ip":"10.254.6.100","gateway":"10.254.6.1","subnet":24},
"nodes":0,"max":10,"serverID":6,"id":-1,"iface":"eth0","switches":[{"addr":"172.16.1.1","iface":"eno3","brand":1,"id":5}],"ips":null}}'
```

###GET /servers/{id}
Get a server by id
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
{
	"addr":(string),
	"iaddr":{
		"ip":(string),
		"gateway":(string),
		"subnet":(int)
	},
	"nodes":(int),
	"max":(int),
	"id":(int),
	"serverID":(int),
	"iface":(string),
	"switches":[
		{
			"addr":(string),
			"iface":(string),
			"brand":(int),
			"id":(int)
		}
	]
}
```

###DELETE /servers/{id}
####PRIVATE
Delete a server
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
Success
```

#####EXAMPLE
```bash
curl -X DELETE http://localhost:8000/servers/5
```

###UPDATE /servers/{id}
####PRIVATE
Update server information
#####BODY
```
{
	"addr":(string),
	"iaddr":{
		"ip":(string),
		"gateway":(string),
		"subnet":(int)
	},
	"nodes":(int),
	"max":(int),
	"id":(int),
	"serverID":(int),
	"iface":(string),
	"switches":[
		{
			"addr":(string),
			"iface":(string),
			"brand":(int),
			"id":(int)
		}
	]
}
```
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
Success
```

#####EXAMPLE
```bash
curl -X UPDATE http://localhost:8000/servers/5 -d \
 '{"addr":"172.16.4.5","iaddr":{"ip":"10.254.4.100","gateway":"10.254.4.1","subnet":24}, 
 "nodes":0,"max":30,"id":5,"serverID":4,"iface":"eno3","switches":[{"addr":"172.16.1.1","iface":"eth4","brand":1,"id":3}],"ips":null}'
```

###GET /testnets/
Get all testnets which are currently running
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
[
	{
		"id":(int),
		"blockchain":(string),
		"nodes":(int),
		"image":(string)
	},...

]
```

###POST /testnets/
Add and deploy a new testnet
#####BODY
```
{
	"servers":[(int),(int)...],
	"blockchain":(string),
	"nodes":(int),
	"image":(string),
	"resources":{
		"cpus":(string),
		"memory":(string)
	},
	"params":(Object containing params specific to the chain/client being built)
}
```
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
Success
```
#####EXAMPLE
```bash
curl -X POST http://localhost:8000/testnets/ -d '{"servers":[3],"blockchain":"ethereum","nodes":3,"image":"ethereum:latest",
"resources":{"cpus":"2.0","memory":"10gb"},"params":null}'
```

###GET /testnets/{id}
Get data on a single testnet
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
{
	"id":(int),
	"blockchain":(string),
	"nodes":(int),
	"image":(string)
}
```

###GET /testnets/{id}/nodes/
####PRIVATE
Get the nodes in a testnet
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
[
	{
		"id":(int),
		"testNetId":(int),
		"server":(int),
		"localId":(int),
		"ip":(string)
	},...
]
```


###GET /status/nodes/
Get the nodes that are running in the latest testnet
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
[
	{
		"name":"whiteblock-node0",
		"server":4
	},...
]
```
#####EXAMPLE
```bash
curl -XGET http://localhost:8000/status/nodes/
```



###POST /exec/{server}/{node}
Execute a command on a given node
#####BODY
```
<bash command>
```
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
<command results>
```

#####EXAMPLE
```bash
curl -X POST http://localhost:8000/exec/4/0 -d 'ls'
```


###GET /params/{blockchain}/
Get the build params for a blockchain
#####RESPONSE
```json
[
	{"chainId":"int"},
	{"networkId":"int"},
	{"difficulty":"int"},
	{"initBalance":"string"},
	{"maxPeers":"int"},
	{"gasLimit":"int"},
	{"homesteadBlock":"int"},
	{"eip155Block":"int"},
	{"eip158Block":"int"}
]
```
#####EXAMPLE
```bash
curl -X GET http://localhost:8000/params/ethereum
```

###GET /defaults/{blockchain}
Get the default parameters for a blockchain
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
{
	"chainId":15468,
	"networkId":15468,
	"difficulty":100000,
	"initBalance":100000000000000000000,
	"maxPeers":1000,
	"gasLimit":4000000,
	"homesteadBlock":0,
	"eip155Block":0,
	"eip158Block":0
}
```
#####EXAMPLE
```bash
curl -X GET http://localhost:8000/defaults/ethereum
```

###GET /log/{server}/{node}
Get both stdout and stderr from the blockchain process

#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
<The contents>
```

#####EXAMPLE
```bash
curl -X POST http://localhost:8000/exec/4/0 -d 'ls'
```

###GET /nodes
Get the nodes for the latest testnet
#####RESPONSE
```json
[
    {
        "id": 1647,
        "testNetId": 134,
        "server": 3,
        "localId": 0,
        "ip": "10.6.0.2",
        "label": ""
    },
    {
        "id": 1648,
        "testNetId": 134,
        "server": 3,
        "localId": 1,
        "ip": "10.6.0.6",
        "label": ""
    }
]
```
#####EXAMPLE
```bash
curl -X GET http://localhost:8000/nodes
```

##Blockchain Specific Parameters

###Geth (Go-Ethereum)
__Note:__ Any configuration option can be left out, and this entire section can even be null,
the example contains all of the defaults

####Options
* `chainId`: The chain id set in the genesis.conf
* `networkId`: The network id
* `difficulty`: The initial difficulty set in the genesis.conf file
* `initBalance`: The initial balance for the accounts
* `maxPeers`: The maximum number of peers for each node
* `gasLimit`: The initial gas limit
* `homesteadBlock`: Set in genesis.conf
* `eip155Block`: Set in genesis.conf
* `eip158Block`: Set in genesis.conf

####Example (using defaults)
```json
{
	"chainId":15468,
	"networkId":15468,
	"difficulty":100000,
	"initBalance":100000000000000000000,
	"maxPeers":1000,
	"gasLimit":4000000,
	"homesteadBlock":0,
	"eip155Block":0,
	"eip158Block":0
}
```
###Syscoin (RegTest)

####Options
* `rpcUser`: The username credential
* `rpcPass`: The password credential
* `masterNodeConns`: The number of connections to set up for the master nodes
* `nodeConns`:  The number of connections to set up for the normal nodes
* `percentMasternodes`: The percentage of the network consisting of master nodes

* `options`: Options to set enabled for all nodes
* `senderOptions`: Options to set enabled for senders
* `receiverOptions`: Options to set enabled for receivers
* `mnOptions`: Options to set enabled for master nodes

* `extras`: Extra options to add to the config file for all nodes
* `senderExtras`: Extra options to add to the config file for senders
* `receiverExtras`: Extra options to add to the config file for receivers
* `mnExtras`: Extra options to add to the config file for master nodes

####Example (using defaults)
```json
{
	"rpcUser":"username",
	"rpcPass":"password",
	"masterNodeConns":25,
	"nodeConns":8,
	"percentMasternodes":90,
	"options":[
		"server",
		"regtest",
		"listen",
		"rest"
	],
	"senderOptions":[
		"tpstest",
		"addressindex"
	],
	"mnOptions":[],
	"receiverOptions":[
		"tpstest"
	],
	"extras":[],
	"senderExtras":[],
	"receiverExtras":[],
	"mnExtras":[]
}
```
