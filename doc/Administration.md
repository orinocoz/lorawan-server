# Server Administration

You can administrate and manage the server via a set of web-pages or via a REST API.
By default, the server listens on HTTP port 8080 and expects "admin" as both username and password.

The port and default credentials (which are set when the server database is created)
can be changed in the [`sys.config`](../lorawan_server.config). The credentials can
be then altered via the admin interface.

## REST API

The following REST resources are made available:

  Resource                  | Methods          | Explanation
 ---------------------------|------------------| ------------------------------------------------
  /applications             | GET              | Supported LoRaWAN applications
  /users                    | GET, POST        | Users of the admin interface
  /users/*ABC*              | GET, PUT, DELETE | User *ABC*
  /gateways                 | GET, POST        | LoRaWAN gateways
  /gateways/*123*           | GET, PUT, DELETE | Gateway with MAC=*123*
  /multicast_channels       | GET, POST        | Class C multicast channels
  /multicast_channels/*123* | GET, PUT, DELETE | Multicast channel with DevAddr=*123*
  /ignored_nodes            | GET, POST        | Nodes ignored by the server
  /ignored_nodes/*123*      | GET, PUT, DELETE | Ignored node with DevAddr=*123*
  /devices                  | GET, POST        | Devices registered for over-the-air activation (OTAA)
  /devices/*123*            | GET, PUT, DELETE | Device with DevEUI=*123*
  /nodes                    | GET, POST        | Active network nodes, both ABP and activated OTAA
  /nodes/*123*              | GET, PUT, DELETE | Active network node with DevAddr=*123*
  /txframes                 | GET              | Frames scheduled for transmission
  /txframes/*123*           | GET, DELETE      | Frame with ID=*123*
  /rxframes                 | GET              | Recent received frames
  /handlers                 | GET              | Backend handlers
  /handlers/*ABC*           | GET, DELETE      | Backend handler for the Group *ABC*
  /connectors               | GET              | Backend connectors
  /connectors/*ABC*         | GET, DELETE      | Backend connector *ABC*

### Filtering

To list only some items the REST API accepts the `_filters` query parameter, which
shall contain URL encoded JSON. For instance:

http://server:8080/rxframes?_filters={"devaddr":"22222222"}

### Sorting
The REST API accepts `_sortField` and `_sortDir` query parameters to sort the list. The
`_sortDir` can be either `ASC` or `DESC`. For instance:

http://server:8080/rxframes?_sortField=datetime&_sortDir=ASC

### Pagination
The REST API accepts `_page` and `_perPage` query parameters to paginate lists,
for instance:

http://server:8080/rxframes?_page=2&_perPage=20

The server also inserts the HTTP header `X-Total-Count` indicating the total item count.


## Web Admin

The management web-pages are available under `/admin`. It is just a wrapper around
the REST API.

You (at least) have to:
 * Add LoRaWAN gateways you want to use to the *Gateways* list.
 * Configure each device you want to use:
   * To add a device activated by personalization (ABP), create a new *Nodes* list entry.
   * To add an OTAA device, create a new *Devices* list entry and start the device. The *Nodes*
     list will be updated automatically once the device joins the network.

### Users

List of user identities that can manage the server. All have the same access rights.

### Gateways

For each LoRaWAN gateway you can set:
 * *MAC* address of the gateway
 * *TX Chain* identifies the gateway "RF chain" used for downlinks; usually 0
 * *NetID* of the network
 * *Location* and *Altitude* of the gateway

![alt tag](https://raw.githubusercontent.com/gotthardp/lorawan-server/master/doc/images/admin-gateway.png)

### Devices

For each device, which may connect to your network, you can set:
 * *DevEUI* of the device
 * *Region* that determines the LoRaWAN regional parameters
 * *Application* identifier corresponding to one of the [Handlers](Handlers.md) configured.
 * *AppID*, which denotes application-specific group or behaviour.
 * *Arguments*, which is an opaque string with application-specific settings.
 * *AppEUI* and *AppKey*
 * *FCnt Check* to be used for this device
   * *Strict 16-bit* (default) or *Strict 32-bit* indicate a standard compliant counter.
   * *Reset on zero* behaves like a "less strict 16-bit", which allows personalised (ABP)
     devices to reset the counter.
     This weakens device security a bit as more reply attacks are possible.
   * *Disabled* disables the check for faulty devices.
     This destroys the device security.
 * *Can Join?* flag that allows you to prevent the device from joining

Once the device joins the network, the *Link* field will contain a reference to the *Nodes* list.

Optionally, you can also define a set of [ADR](ADR.md) parameters. Once the device
joins the network, the server will attempt to configure the device accordingly.

![alt tag](https://raw.githubusercontent.com/gotthardp/lorawan-server/master/doc/images/admin-device.png)

### Nodes

Nodes are active devices. For each network node you can set:
 * *DevEUI* of the device
 * *Region* that determines the LoRaWAN regional parameters
 * *Application* identifier corresponding to one of the [Handlers](Handlers.md) configured.
 * *AppID*, which denotes application-specific group or behaviour.
 * *Arguments*, which is an opaque string with application-specific settings.
 * *NwkSKey* and *AppSKey*
 * *FCnt Check* to be used for this device (see the Devices section for more explanation).

Optionally, you can also set the [ADR](ADR.md) parameters. The server will attempt
to configure the device accordingly.

Below the configuration options you can monitor the performance of the device. You
can see the assumed [ADR](ADR.md) parameters and two graphs that display the last
50 received frames.

The *Downlinks* table lists frames created by the application, which are scheduled for
transmission. Class A devices listen for downlinks only for 2 seconds after an uplink
transmission, so it may take a while until all messages are transmitted.

![alt tag](https://raw.githubusercontent.com/gotthardp/lorawan-server/master/doc/images/admin-link-status.png)


## Backup and Restore

Use the `dbexport` script to backup your list of users, gateways, devices and nodes.
This will create several `db*.json` files. Use the `dbimport` script to write these
files back to the server database.

The database is stored in the `Mnesia.lorawan@localhost` directory. To upgrade
the database structure or recover from database errors you should do `dbexport`,
then shutdown the server, update the server binaries, delete the Mnesia directory,
start the server and do `dbimport`.
