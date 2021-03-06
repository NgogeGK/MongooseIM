### Module Description

`Inbox` is an experimental feature implemented as a few seperate modules.
To use it, enable mod_inbox in the config file.

### Options

* **backend** (atom, default: `odbc`) - Database backend to use. For now, only `odbc` is supported.
* **reset_markers** (list, default: `[displayed]`) - List of atom chat markers that when sent, will reset the unread message counter for a conversation.
This works when [Chat Markers](https://xmpp.org/extensions/xep-0333.html) are enabled on the client side.
Possible values are from the set: `displayed`, `received`, `acknowledged`. Setting as empty list (not recommended) means that no chat marker can decrease the counter value.
* **groupchat** (list, default: `[muclight]`) - The list indicating which groupchats will be included in inbox.
Possible values are `muclight` [Multi-User Chat Light](https://xmpp.org/extensions/inbox/muc-light.html) or `muc` [Multi-User Chat](https://xmpp.org/extensions/xep-0045.html).
* **aff_changes** (boolean, default: `true`) - use this option when `muclight` is enabled.
Indicates if MUC Light affiliation change messages should be included in the conversation inbox.
Only changes that affect the user directly will be stored in their inbox.
* **remove_on_kicked** (boolean, default: `true`) - use this option when `muclight` is enabled.
If true, the inbox conversation is removed for a user when they are removed from the groupchat.
* **iqdisc** (atom, default: `no_queue`)

### Note about supported RDBMS

`mod_inbox` executes upsert queries, which have different syntax in every supported RDBMS.
Inbox currently supports the following DBs:

* MySQL via native driver
* PgSQL via native driver
* MSSQL via ODBC driver

### Legacy MUC support
Inbox comes with support for the legacy MUC as well. It stores all groupchat messages sent to
room in each sender's and recipient's inboxes and private messages. Currently it is not possible to
configure it to store system messages like [subject](https://xmpp.org/extensions/xep-0045.html#enter-subject) 
or [affiliation](https://xmpp.org/extensions/xep-0045.html#affil) change.

### Filtering and ordering

Inbox query results may be filtered by time range and sorted by timestamp.
By default, `mod_inbox` returns all conversations, listing the ones updated most recently first.

A client may specify three parameters:

* Start date for the result set (variable `start`, value: ISO timestamp)
* End date for the result set (variable `end`, value: ISO timestamp)
* Order by timestamp (variable `order`, values: `asc`, `desc`)

They are encoded inside a standard XMPP [Data Forms](https://xmpp.org/extensions/xep-0004.html) format.
Dates must be formatted according to [XMPP Date and Time Profiles](https://xmpp.org/extensions/xep-0082.html).
It is not mandatory to add an empty data form if a client prefers to use default values (`<query/>` element may be empty).
However, the IQ type must be "set", even when data form is missing.

Your client application may request the currently supported form with IQ get:

```
Client:

<iq type='get' id='c94a88ddf4957128eafd08e233f4b964'>
  <query xmlns='erlang-solutions.com:xmpp:inbox:0'/>
</iq>

Server:

<iq from='alicE@localhost' to='alicE@localhost/res1' id='c94a88ddf4957128eafd08e233f4b964' type='result'>
  <query xmlns='erlang-solutions.com:xmpp:inbox:0'>
    <x xmlns='jabber:x:data' type='form'>
      <field type='hidden' var='FORM_TYPE'><value>erlang-solutions.com:xmpp:inbox:0</value></field>
      <field var='start' type='text-single'/>
      <field var='end' type='text-single'/>
      <field var='order' type='list-single'>
        <value>desc</value>
        <option label='Ascending by timestamp'><value>asc</value></option>
        <option label='Descending by timestamp'><value>desc</value></option>
      </field>
    </x>
  </query>
</iq>
```

### Example Request

```
Alice sends:

<message type="chat" to="bOb@localhost/res1" id=”123”>
  <body>Hello</body>
</message>

Bob receives:

<message from="alicE@localhost/res1" to="bOb@localhost/res1" id=“123” xml:lang="en" type="chat">
  <body>Hello</body>
</message>

Alice sends:

<iq type="set" id="10bca">
  <inbox xmlns=”erlang-solutions.com:xmpp:inbox:0” queryid="b6">
    <x xmlns='jabber:x:data' type='form'>
      <field type='hidden' var='FORM_TYPE'><value>erlang-solutions.com:xmpp:inbox:0</value></field>
      <field type='text-single' var='start'><value>2018-07-10T12:00:00Z</value></field>
      <field type='text-single' var='end'><value>2018-07-11T12:00:00Z</value></field>
      <field type='list-single' var='order'><value>asc</value></field>
    </x>
  </inbox>
</iq>


Alice receives:

<message from="alicE@localhost" to="alicE@localhost" id="9b759">
  <result xmlns="erlang-solutions.com:xmpp:inbox:0" unread="0" queryid="b6">
    <forwarded xmlns="urn:xmpp:forward:0">
      <delay xmlns="urn:xmpp:delay" stamp="2018-07-10T23:08:25.123456Z"/>
      <message xml:lang="en" type="chat" to="bOb@localhost/res1" from="alicE@localhost/res1" id=”123”>
        <body>Hello</body>
      </message>
    </forwarded>
  </result>
</message>

<iq from="alicE@localhost" to="alicE@localhost/res1" id="b6" type="result">
  <count xmlns='erlang-solutions.com:xmpp:inbox:0'>1</count>
</iq>

```


### Example Configuration

```
{mod_inbox, [{backend, odbc},
             {reset_markers, [displayed]},
             {aff_changes, true},
             {remove_on_kicked, true},
             {groupchat, [muclight]}
            ]},
```

