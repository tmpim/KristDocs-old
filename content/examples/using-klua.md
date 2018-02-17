+++
type="page"

title = "Using k.lua"
weight = 2

creatordisplayname = "Drew Lemmy"
creatoremail = "drew@lemmmy.pw"
lastmodifierdisplayname = "Drew Lemmy"
lastmodifieremail = "drew@lemmmy.pw"
+++

{{%alert theme="info" %}}This guide assumes you have already [**installed k.lua**](../installing-klua).{{% /alert %}}

## Requiring and initialising dependencies
First, make sure you have required all the dependencies for k.lua:

```lua
local w = require("w") -- allows interaction with krist websocket api (for realtime data)
local r = require("r") -- makes http requests easier
local k = require("k") -- the krist api itself
local jua = require("jua") -- makes events easier
os.loadAPI("json.lua") -- to parse data returned by the krist api
local await = jua.await

-- initialise w.lua, r.lua and k.lua
r.init(jua)
w.init(jua)
k.init(jua, json, w, r)
```

## Understanding Jua
Jua is a callback system for ComputerCraft, inspired by JavaScript callbacks. It gets rid of the usual event loop, and replaces it with `jua.on("event", callback)`. For example:

```lua
local jua = require("jua")

jua.on("key", function(event, key)
	-- this code is called every time there is a `key` event
	print("Key pressed: " .. keys.getName(key))
end)

jua.on("terminate", function()
	-- this event is required to ensure we can actually close our program
	jua.stop()
	printError("Terminated")
end)

jua.go(function()
	-- jua is ready, and you can run all your code in here
	print("Jua is ready.")
end)
```

## Making requests to the Krist API
### Supported API calls
K.lua currently supports the following API calls:

- [`address(address)`](https://github.com/justync7/k.lua/blob/master/k.lua#L48) ([api docs](http://krist.ceriat.net/docs/#api-AddressGroup-GetAddress)) - Gets information about an address, such as its balance
- [`addressTransactions(address, limit, offset)`](https://github.com/justync7/k.lua/blob/master/k.lua#L55) ([api docs](http://krist.ceriat.net/docs/#api-AddressGroup-GetAddressTransactions)) - Gets recent transactions by an address
- [`addressNames(address)`](https://github.com/justync7/k.lua/blob/master/k.lua#61) ([api docs](http://krist.ceriat.net/docs/#api-AddressGroup-GetAddressNames)) - Gets names owned by an address
- [`addresses(limit, offset)`](https://github.com/justync7/k.lua/blob/master/k.lua#67) ([api docs](http://krist.ceriat.net/docs/#api-AddressGroup-GetAddresses)) - List all addresses
- [`rich(limit, offset)`](https://github.com/justync7/k.lua/blob/master/k.lua#73) ([api docs](http://krist.ceriat.net/docs/#api-AddressGroup-GetRichAddresses)) - List the addresses with the highest balance
- [`transactions(limit, offset)`](https://github.com/justync7/k.lua/blob/master/k.lua#79) ([api docs](http://krist.ceriat.net/docs/#api-TransactionGroup-GetTransactions)) - List all transactions
- [`latestTransactions(limit, offset)`](https://github.com/justync7/k.lua/blob/master/k.lua#85) ([api docs](http://krist.ceriat.net/docs/#api-TransactionGroup-GetLatestTransactions)) - List all recent transactions
- [`transaction(transactionID)`](https://github.com/justync7/k.lua/blob/master/k.lua#91) ([api docs](http://krist.ceriat.net/docs/#api-TransactionGroup-GetTransaction)) - Get information about a single transaction
- [`makeTransaction(privatekey, to, amount, metadata)`](https://github.com/justync7/k.lua/blob/master/k.lua#97) ([api docs](http://krist.ceriat.net/docs/#api-TransactionGroup-MakeTransaction)) - Make a transaction

### Making a request
To make a request, you need to `await()` the API call, for example:

```lua
-- success is true if the request went ok, false if not
-- address contains information about the address
local success, address = await(k.address, "khugepoopy")
assert(success, "Failed to get address.")

print("Address: " .. address.address)
print("Balance: " .. address.balance)
```

This code should go inside the `jua.go` call.

### Using websockets
K.lua also supports websockets. These allow you to receive event data from the Krist node in realtime.

```lua
local function openWebsocket()
	local success, ws = await(k.connect, "your-private-key")
	assert(success, "Failed to get websocket URL")

	print("Connected to websocket.")

	-- here we subscribe to the 'transactions' event
	local success = await(ws.subscribe, "transactions", function(data)
		-- this function is called every time a transaction is made
		local transaction = data.transaction

		print("Transaction made:")
		print("From: " .. transaction.from)
		print("To: " .. transaction.to)
		print("Value: " .. transaction.value .. " KST")
	end)
	assert(success, "Failed to subscribe to event")
end

jua.go(function()
	openWebsocket()
	-- your other code
end)
```