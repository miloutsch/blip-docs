## Contacts

The **contacts** extension allows the management of the chatbot's roster, which can be used to store data of the chatbot's clients. It is possible to save information like name, address, gender and other generic information, using the `extras` property. It is also possible to use the contacts fields as variables of the messages sent by the chatbot. This property only allows `string` values and does not allows complex objects. You can also set the `group` property for contacts organization. Events where the `identity` property is from a special group called 'testers' will be ignored on BLiP events dashboard.

To use any feature of **contacts** extension send a command with the following properties:

| Name | Description |
|---------------------------------|--------------|
| id    | Unique identifier of the command.   |
| method    | The command verb  |
| resource | The contact document. |
| type | **"application/vnd.lime.contact+json"** |
| uri    | **/contacts**   |
| to     | **postmaster@msging.net** (not required) |

The command's properties `resource` and `method` can change according to the feature.
A contact object passed as a document `resource` has the following properties:

| Property     | Description                                                        | Example |
|--------------|--------------------------------------------------------------------|---------|
| **identity** | The client identity in a specific channel.                         | `11121023102013021@messenger.gw.msging.net (Messenger user)` |
| **name**   | **Optional** The client's name (string).  | `"Rafael Pacheco"` |
| **gender** | **Optional** The client's gender (string).  | `"male"` |
| **group**   | **Optional** The client's group tag (string).   | `"testers"` |
| **address**   | **Optional** The client's address (string).   | `"83, Paraguassu Street"` |
| **city**   | **Optional** The client's city (string).   | `"Belo Horizonte"` |
| **email**   | **Optional** The client's email (string).   | `"rafaelpa@take.net"` |
| **phoneNumber**   | **Optional** The client's phone number (string).   | `"5531000000000"` |
| **cellPhoneNumber**   | **Optional** The client's cell phone number (string).   | `"5531999999999"` |
| **timezone**   | **Optional** The client's timezone id (int).   | `-3` |
| **culture**   | **Optional** The client's culture info (string).   | `"pt-br"` |
| **extras**   | **Optional** The client's extra informations.         | `{"customerExternalId": "41231", "cpf": "00000000000" }` |

For more information about the supported fields, please refer to the [Lime protocol](http://limeprotocol.org/resources.html#contact) documentation.

#### Message variable replacement

The contacts fields can be used to replace variables on messages sent by the chatbot. To make a replacement in a message, the `metadata` key `#message.replaceVariables` should be present with the value `true` and the message text should have variables in the `${contact.<propertyName>}` format, where `<propertyName>` is the contact property for replacement. It is possible to use all fields from the contact, including the keys in the `extras` property. In this case, is only required to use the `${contact.extras.<extraPropertyName>}` convention, where `<extraPropertyName>` is the value for replacement. If the value is not available, it is only removed from the message.

### Add (or update) a contact

```javascript
client.addMessageReceiver('text/plain', async (message) => {
    await client.sendCommand({  
        id: Lime.Guid(),
        method: Lime.CommandMethod.SET,
        uri: '/contacts',
        type: 'application/vnd.lime.contact+json',
        resource: {
            identity: '11121023102013021@messenger.gw.msging.net',
            name: 'John Doe',
            gender:'male',
            group: 'friends',    
            extras: {
                plan: 'Gold',
                code: '1111'      
            }
        }
    });
});
```

```http
POST https://msging.net/commands HTTP/1.1
Content-Type: application/json
Authorization: Key {YOUR_TOKEN}

{  
  "id": "1",
  "method": "set",
  "uri": "/contacts",
  "type": "application/vnd.lime.contact+json",
  "resource": {
    "identity": "11121023102013021@messenger.gw.msging.net",
    "name": "John Doe",
    "gender":"male",
    "group":"friends",    
    "extras": {
      "plan":"Gold",
      "code":"1111"      
    }
  }
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "method": "set",
    "status": "success",
    "id": "1",
    "from": "postmaster@crm.msging.net/#az-iris5",
    "to": "contact@msging.net",
    "metadata": {
        "#command.uri": "lime://contact@msging.net/contacts"
    }
}
```

```csharp
using System.Threading;
using System.Threading.Tasks;
using Lime.Protocol;
using Take.Blip.Client;
using Take.Blip.Client.Extensions.Contacts;
using Lime.Messaging.Resources;
using System.Collections.Generic;

namespace Extensions
{
    public class ContactMessageReceiver : IMessageReceiver
    {
        private readonly ISender _sender;
        private readonly Settings _settings;
        private readonly IContactExtension _contactExtension;

        public ContactMessageReceiver(ISender sender, Settings settings, IContactExtension contactExtension)
        {
            _sender = sender;
            _settings = settings;
            _contactExtension = contactExtension;
        }

        public async Task ReceiveAsync(Message message, CancellationToken cancellationToken)
        {
            var identity = new Identity("11121023102013021", "messenger.gw.msging.net");
            var contact = new Contact
            {
                Name = "John Doe",
                Gender = Gender.Male,
                Group = "friends",
                Extras = new Dictionary<string, string>
                {
                    {"plan", "gold" },
                    {"code", "1111" },
                }
            };

            await _contactExtension.SetAsync(identity, contact, cancellationToken);
        }
    }
}
```

In order to store informations about a chatbot's client, it is possible to save and update data using **contacts extension**. This sample shows how to add a Messenger customer with identity `11121023102013021@messenger.gw.msging.net` to the chatbot's roster. 

### Get contact

```javascript
client.addMessageReceiver('text/plain', async (message) => {
    var data = await client.sendCommand({  
        id: Lime.Guid(),
        method: Lime.CommandMethod.GET,
        uri: '/contacts/11121023102013021@messenger.gw.msging.net'
    });
    console.log(data);
});
```

```http
POST https://msging.net/commands HTTP/1.1
Content-Type: application/json
Authorization: Key {YOUR_TOKEN}

{  
  "id": "2",
  "method": "get",
  "uri": "/contacts/11121023102013021@messenger.gw.msging.net"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "type": "application/vnd.lime.contact+json",
    "resource": {
        "name": "John Doe",
        "group": "friends",
        "identity": "11121023102013021@messenger.gw.msging.net",
        "gender": "male",
        "extras": {
            "plan": "Gold",
            "code": "1111"
        }
    },
    "method": "get",
    "status": "success",
    "id": "2",
    "from": "postmaster@crm.msging.net/#az-iris3",
    "to": "contact@msging.net/default",
    "metadata": {
        "#command.uri": "lime://contact@msging.net/contacts/11121023102013021@messenger.gw.msging.net"
    }
}
```

```csharp
using System.Threading;
using System.Threading.Tasks;
using Lime.Protocol;
using Take.Blip.Client;
using Take.Blip.Client.Extensions.Contacts;

namespace Extensions
{
    public class ContactMessageReceiver : IMessageReceiver
    {
        private readonly ISender _sender;
        private readonly IContactExtension _contactExtension;
        private readonly Settings _settings;

        public ContactMessageReceiver(ISender sender, Settings settings, IContactExtension contactExtension)
        {
            _sender = sender;
            _settings = settings;
            _contactExtension = contactExtension;
        }

        public async Task ReceiveAsync(Message message, CancellationToken cancellationToken)
        {
            var identity = new Identity("11121023102013021", "messenger.gw.msging.net");

            var contact = await _contactExtension.GetAsync(identity, cancellationToken);
        }
    }
}
```

For the same contact `11121023102013021@messenger.gw.msging.net`, it is possible to get all of its information using a `GET` contact command.

### Get contacts with paging

```javascript
client.addMessageReceiver('text/plain', async (message) => {
    var data = await client.sendCommand({  
        id: Lime.Guid(),
        method: Lime.CommandMethod.GET,
        uri: '/contacts?$skip=0&$take=3'
    });
    data.resource.items.forEach(function (value) {
        console.log(value);
    });  
});
```

```http
POST https://msging.net/commands HTTP/1.1
Content-Type: application/json
Authorization: Key {YOUR_TOKEN}

{  
  "id": "3",
  "method": "get",
  "uri": "/contacts?$skip=0&$take=3"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "type": "application/vnd.lime.collection+json",
    "resource": {
        "total": 12,
        "itemType": "application/vnd.lime.contact+json",
        "items": [
            {
				"identity": "11121023102013021@messenger.gw.msging.net",
				"name": "John Doe",
				"gender":"male", 
				"group":"friends", 
				"extras":{
					"plan":"Gold",
					"code":"1111"
				}
			},
			{
				"identity": "213121@telegram.gw.msging.net",
				"name": "Joseph from Telegram",
				"email":"ze@gmail.com"
			},
			{
				"identity": "5511999990000@take.io",
				"name": "Mary"
			}
        ]
    },
    "method": "get",
    "status": "success",
    "id": "3",
    "from": "postmaster@crm.msging.net/#az-iris5",
    "to": "contact@msging.net",
    "metadata": {
        "#command.uri": "lime://contact@msging.net/contacts?$skip=0&$take=3"
    }
}
```

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Lime.Protocol;
using Take.Blip.Client;
using Take.Blip.Client.Extensions.Contacts;
using teste_documentacao;

namespace Extensions
{
    public class ContactMessageReceiver : IMessageReceiver
    {
        private readonly ISender _sender;
        private readonly Settings _settings;
        private readonly IContactExtension _contactExtension;

        public ContactMessageReceiver(ISender sender, Settings settings, IContactExtension contactExtension)
        {
            _sender = sender;
            _settings = settings;
            _contactExtension = contactExtension;
        }

        public async Task ReceiveAsync(Message message, CancellationToken cancellationToken)
        {
            var take = 3;
            var skip = 0;

            var command = new Command{
                Id = EnvelopeId.NewId(),
                Method = CommandMethod.Get,
                Uri = new LimeUri($"/contacts?skip={skip}&$take={take}")
            };

            var commandResponse = await _sender.ProcessCommandAsync(command, cancellationToken);
            var contacts = commandResponse.Resource as DocumentCollection;
        }
    }
}
```

If you need to get more than one chatbot's contact, you can use a query pagination. This sample shows how to take the **three first roaster's contacts**.

<aside  class="notice">
Note: You can also filter your query with one of the properties of the contact resource, using the <code>filter</code> property:<br><br>

<code>filter=(substringof('{value}',{propertyName}))</code><br><br>

<b>Example</b>: /contacts?$skip=0&$take=20<b>&$filter=(substringof('John Doe',name))</b> </aside>

### Send message with contact name

```javascript
client.addMessageReceiver('text/plain', async (message) => {
    await client.sendCommand({  
        id: Lime.Guid(),
        to: '11121023102013021@messenger.gw.msging.net',
        type: 'text/plain',
        content: 'Hello ${contact.name}, welcome to the ${contact.extras.plan} plan!',
        metadata: {
            '#message.replaceVariables': 'true'
        }
    });
});
```

```http
POST https://msging.net/messages HTTP/1.1
Content-Type: application/json
Authorization: Key {YOUR_TOKEN}

{  
  "id": "4",
  "to": "11121023102013021@messenger.gw.msging.net",
  "type": "text/plain",
  "content": "Hello ${contact.name}, welcome to the ${contact.extras.plan} plan!",
  "metadata": 
  {
    "#message.replaceVariables": "true"
  }
}
```

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Lime.Protocol;
using Take.Blip.Client;
using Lime.Messaging.Contents;

namespace Extensions
{
    public class PlainTextMessageReceiver : IMessageReceiver
    {
        private readonly ISender _sender;
        private readonly Settings _settings;

        public PlainTextMessageReceiver(ISender sender, Settings settings)
        {
            _sender = sender;
            _settings = settings;
        }

        public async Task ReceiveAsync(Message m, CancellationToken cancellationToken)
        {
            var message = new Message
            {
                To = Node.Parse("11121023102013021@messenger.gw.msging.net"),
                Id = Guid.NewGuid().ToString(),
                Content = new PlainText
                {
                    Text = "Hello ${contact.name}, welcome to the ${contact.extras.plan} plan!"
                },
                Metadata = new Dictionary<string, string>
                {
                    {"#message.replaceVariables", "true" }
                }
            };

            await _sender.SendMessageAsync(message, cancellationToken);
        }
    }
}
```

If you have information of some client stored on the chatbot's roster, you can send a customized message using these values as message variables.
To do this, add a metadata `{ "#message.replaceVariables": "true" }` and use any property of the contact resource. This sample shows how to replace a contact name on a welcome message.