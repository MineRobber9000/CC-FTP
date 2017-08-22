# CC-FTP Protocol

|Details   |      |
|----------|------|
|Name      |CC-FTP|
|Version   |0.1.0 |
|Updates   |None  |
|Updated by|None  |

*TODO: when this is completed, copy and format as a template for [CraftOS-Standards](https://github.com/oeed/CraftOS-Standards).*

## Packets

All CC-FTP packets must be tables. If sent using rednet, packets SHOULD use the protocol "ccftp". Packets have different members depending on if they come from a client or the server.

Client to server packets must include:

Type|Member Name|Description
----|-----------|-----------
`string`|sVerb|A CC-FTP [verb](#verbs). If this is invalid, the server should respond with code `400` (Bad verb)
`table`|tArgs|The arguments. Differ by verb.

Server to client packets must include:

Type|Member Name|Description
----|-----------|-----------
`number`|nCode|A CC-FTP [response code](#codes). If this is invalid, the client can ignore it.
`string`|sReadable|A readable interpretation of the code. Can be used as an error message if the code is of negative meaning.
`table`|tArgs|The arguments. Differ by code and verb being responded to.

## Verbs

Verbs are as follows:

Verb|Meaning|Arguments|Response
----|-------|---------|--------
INIT|Initialize connection. (Should be sent before anything else)|[Capabilities](#capabilities)|A code `100` packet containing capabilities and requirements.
GET|Get a file from the server.|The file to get, as a string.|The contents of the file, if allowed to access it.
LIST|List a directory.|The directory, as a string.|The response of `fs.list(dir)`.
PASS|Get the server's passcode. (Used to encrypt username and password)|None|The passcode, as a string.
LOGIN|Log in to the server.|Username and encrypted password.|Code `200` on success, code `402` on failure, and a token (number)
PUT|Put a file onto the server (may require log in)|If log in required, the token.|See [Response to PUT].

### Response to PUT

When responding to a PUT request, there are 3 possible response codes.

Code|Means
----|-----
203 |File was successfully written.
402 |You need to login. (Always expect 402, even when not putting a file. Servers are allowed to use 402 to respond to anything.)
403 |File couldn't be written to.

## Codes

Codes are separated into 100-number blocks as such:

* `1xx` - Connection details. (such as capabilities and server versions)
* `2xx` - Success.
* `3xx` - Warnings. Dynamic, so if you recieve one of these, make sure to display the `sReadable` to the user and tell them to notify the app developer, as this is used for deprecation of verbs and such. 
* `4xx` - Failures.

The meanings of all used codes are here:

Code|Meaning|Example `sReadable`
-|-|-
100|Used as a response to INIT, arguments contain capabilities (`tArgs.tCaps`) and requirements. (`tArgs.tReqs`)|"Requested data"
200|General success, used as response to GET unless the file in question doesn't exist|"OK"
201|Requested passcode, arguments contain the passcode.|"Passcode enclosed"
202|Correct login, arguments contain token|"Login successful"
203|File successfully written|"File text.txt successfully stored."
3xx|Warning. Specific warning contained in `sReadable`.|"WARNING: Capability 'delete' is non-standard and may not be supported by this server."
400|Bad verb.|"The verb VERB is invalid."
401|Bad login|"Login unsuccessful."
402|Login required. Can be used to block access to verbs or files.|"The action PUT requires a login on the server."
403|Impossible action, `sReadable` contains a reason why.|"Cannot write to read-only file 'rom/programs/http/wget.lua'"

## Capabilities

Current standard capabilities:

Value|Meaning
-----|-------
`dynamic`|Files are dynamic and served "on-the-spot". Generally means read-only.
`read-only`|Files cannot be written to. Used for distribution servers as well as most dynamic servers.

## Requirements

Requirements include:

Value|Meaning
-----|-------
`login-to-put`|You must login to PUT files on the server.
