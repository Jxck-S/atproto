### Introduction

This SDK attempts to implement everything that provides ATProto. Due to the unstable state of the protocol (it grows and changes fast) and a bit of outdated documentation, only the client side is supported yet. There is support for Lexicon Schemes, XRPC clients, and Firehose for now. All models, queries, and procedures are generated automatically. The main focus is on the lexicons of atproto.com and bsky.app, but it doesn't have a vendor lock on it. Feel free to use the code generator for your own lexicon schemes. SDK also provides utilities to work with CID, NSID, AT URI Schemes. DAG-CBOR, CAR files.

### Requirements

- Python 3.7 or higher.
- Access to Bsky if you don't have own server.

### Installing

``` bash
pip3 install -U atproto
```

### Quick start

First of all, you need to create the instance of the XRPC Client. To do so you have 2 major options: asynchronous, and synchronous. The difference only in import and how you call the methods. If you are not familiar with async use sync instead.

For sync:
```python
from atproto import Client

client = Client()
# by default, it uses the server of bsky.app. To change this behaviour pass the base api URL to constructor
# Client('https://my.awesome.server/xrpc')
```

Fro async:
```python
from atproto import AsyncClient

client = AsyncClient()
# by default, it uses the server of bsky.app. To change this behaviour pass the base api URL to constructor
# AsyncClient('https://my.awesome.server/xrpc')
```

In the snippets below only the sync version will be presented.

Right after the creation of the Client instance you probably want to access the full API and perform actions by profile. To achieve this you should log in to the network using your handle and password. The password could be an app-specific one.

```python
from atproto import Client

client = Client()
client.login('my-username', 'my-password')
```

You are awesome! Now you feel to pick any high-level method that you want and perform it!

Code to send post:
```python
from atproto import Client

client = Client()
client.login('my-username', 'my-password')
client.send_post(text='Hello World!')
```

Useful links to continue:
- [List of all methods with documentation](https://atproto.readthedocs.io/en/latest/xrpc_clients/client.html).
- [Examples of using the methods](https://github.com/MarshalX/atproto/tree/main/examples).

### Documentation

The documentation is live at [atproto.blue](https://atproto.blue/).

### Getting help

You can get help in several ways:
- Report bugs, request new features by [creating an issue](https://github.com/MarshalX/atproto/issues/new).
- Ask questions by [starting a discussion](https://github.com/MarshalX/atproto/discussions/new).
- Ask questions in [Discord server](https://discord.gg/PCyVJXU9jN).

### Advanced usage

I'll be honest. The high-level Client that was shown in the "Quick Start" section is not a real ATProto API. It's syntax sugar built upon the real XRPC methods! The high-level methods are not cover the full need of developers. To be able to do anything that you want you should know to work with low-level API. Let's dive into it!

The basics:
- Namespaces – classes that group sub-namespaces and the XRPC queries and procedures. Built upon NSID ATProto semantic.
- Model – dataclasses for input, output, and params of the methods from namespaces. Models describe Record and all other types in the Lexicon Schemes.

The client contains references to the root of all namespaces. It's `app` and `bsky` for now.
```python
from atproto import Client
Client().com
Client().bsky
```

To dive deeper you can navigate using hints from your IDE. Thanks to well-type hinted SDK it's much easier.
```python
from atproto import Client
Client().com.atproto.server.create_session(...)
Client().com.atproto.sync.get_blob(...)
Client().bsky.feed.get_likes(...)
Client().bsky.graph.get_follows(...)
```

The endpoint of the path is always the method that you want to call. The method presents a query or procedure in XRPC. You should not care about it much. The only thing you need to know is that the procedures required data objects. Queries could be called with or without params at all.

To deal with methods we need to deal with models! Models are available in the `models` module and have NSID-based aliases. Let's take a look at it.
```python
from atproto import models
models.ComAtprotoIdentityResolveHandle
models.AppBskyFeedPost
models.AppBskyActorGetProfile
# 90+ more...
```

The model classes in the models aliases could be:
- Data model
- Params model
- Response model
- Record model
- Type model
- Type reference model

The only thing you need to know is how to create instances of models. Not with all models you will work as model-creator. For example, Response models will be created by SDK for you.

There are a few ways how to create the instance of a model:
- Dict-based
- Class-based
- Class-based with keyword arguments

The instances of data and params models should be passed as arguments to the methods that were described above.

Dict-based:
```python
from atproto import Client


client = Client()
client.login('my-username', 'my-password')
# The params model will be created automatically internally for you!
print(client.com.atproto.identity.resolve_handle({'handle': 'marshal.dev'}))
```

Class-based:
```python
from atproto import Client, models


client = Client()
client.login('my-username', 'my-password')
params = models.ComAtprotoIdentityResolveHandle.Params('marshal.dev')
print(client.com.atproto.identity.resolve_handle(params))
```

Class-based with keywords:
```python
from atproto import Client, models


client = Client()
client.login('my-username', 'my-password')
params = models.ComAtprotoIdentityResolveHandle.Params(handle='marshal.dev')
print(client.com.atproto.identity.resolve_handle(params))
```

Tip: look at typehint of the method to figure out the name and the path to the input/data model!

Pro Tip: use IDE autocompletion to find necessary models! Just start typing the method name right after the dot (`models.{type method name in camel case`).

Models could be nested as hell. Be ready for it!

This is how we can send a post with the image using low-level XRPC Client:
```python
from datetime import datetime

from atproto import Client, models


client = Client()
client.login('my-username', 'my-password')

with open('cat.jpg', 'rb') as f:
    img_data = f.read()

    upload = client.com.atproto.repo.upload_blob(img_data)
    images = [models.AppBskyEmbedImages.Image(alt='Img alt', image=upload.blob)]
    embed = models.AppBskyEmbedImages.Main(images=images)

    client.com.atproto.repo.create_record(
        models.ComAtprotoRepoCreateRecord.Data(
            repo=client.me.did,
            collection=models.ids.AppBskyFeedPost,
            record=models.AppBskyFeedPost.Main(
                createdAt=datetime.now().isoformat(), text='Text of the post', embed=embed
            ),
        )
    )
```

I hope you are not scared. May the Force be with you. Good luck!

### Future changes

Things that a want to do soon:
- Use camel_case names of attributes in all models. Will break backward compatibility
- Resolve issues with typehints. There are a lot of reference models with the same set of fields. Now they are completely different types
- Provide autogenerated Record Namespaces with more high-level work with basic operations upon records (CRUD + list records)
