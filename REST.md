

__Warning: Do not use any path marked PRIVATE, they will begin to require credentials in the near future__

###GET /servers/
Get the current registered servers
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
{
	"server_name":{
		"Addr":(string),
		"Iaddr":{
			"Ip":(string),
			"Gateway":(string),
			"Subnet":(int)
		},
		"Nodes":(int),
		"Max":(int),
		"Id":(int),
		"ServerID":(int),
		"Iface":(string),
		"Switches":[
			{
				"Addr":(string),
				"Iface":(string),
				"Brand":(int),
				"Id":(int)
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
	"Addr":(string),
	"Iaddr":{
		"Ip":(string),
		"Gateway":(string),
		"Subnet":(int)
	},
	"Nodes":(int),
	"Max":(int),
	"Id":-1,
	"ServerID":(int),
	"Iface":(string),
	"Switches":[
		{
			"Addr":(string),
			"Iface":(string),
			"Brand":(int),
			"Id":(int)
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
```
curl -X PUT http://localhost:8000/servers/foxtrot -d '{"Addr":"172.16.6.5","Iaddr":{"Ip":"10.254.6.100","Gateway":"10.254.6.1","Subnet":24},"Nodes":0,"Max":10,"ServerID":6,"Id":-1,"Iface":"eth0","Switches":[{"Addr":"172.16.1.1","Iface":"eno3","Brand":1,"Id":5}],"Ips":null}}'
```

###GET /servers/{id}
Get a server by id
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
{
	"Addr":(string),
	"Iaddr":{
		"Ip":(string),
		"Gateway":(string),
		"Subnet":(int)
	},
	"Nodes":(int),
	"Max":(int),
	"Id":(int),
	"ServerID":(int),
	"Iface":(string),
	"Switches":[
		{
			"Addr":(string),
			"Iface":(string),
			"Brand":(int),
			"Id":(int)
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
```
curl -X DELETE http://localhost:8000/servers/5
```

###UPDATE /servers/{id}
####PRIVATE
Update server information
#####BODY
```
{
	"Addr":(string),
	"Iaddr":{
		"Ip":(string),
		"Gateway":(string),
		"Subnet":(int)
	},
	"Nodes":(int),
	"Max":(int),
	"Id":(int),
	"ServerID":(int),
	"Iface":(string),
	"Switches":[
		{
			"Addr":(string),
			"Iface":(string),
			"Brand":(int),
			"Id":(int)
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
```
curl -X UPDATE http://localhost:8000/servers/5 -d '{"Addr":"172.16.4.5","Iaddr":{"Ip":"10.254.4.100","Gateway":"10.254.4.1","Subnet":24},"Nodes":0,"Max":30,"Id":5,"ServerID":4,"Iface":"eno3","Switches":[{"Addr":"172.16.1.1","Iface":"eth4","Brand":1,"Id":3}],"Ips":null}'
```

###GET /testnets/
Get all testnets which are currently running
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
[
	{
		"Id":(int),
		"Blockchain":(string),
		"Nodes":(int),
		"Image":(string)
	},...

]
```

###POST /testnets/
Add and deploy a new testnet
#####BODY
```
{
	"Servers":[(int),(int)...],
	"Blockchain":(string),
	"Nodes":(int),
	"Image":(string)
}
```
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
Success
```

###GET /testnets/{id}
Get data on a single testnet
#####RESPONSE
```
HTTP/1.1 200 OK
Date: Mon, 22 Oct 2018 15:31:18 GMT
{
	"Id":(int),
	"Blockchain":(string),
	"Nodes":(int),
	"Image":(string)
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
	"Id":(int),
	"TestNetId":(int),
	"Server":(int),
	"LocalId":(int),
	"Ip":(string)
]
```
