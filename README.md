# Coding for Jellyfin API

I'm writing a thing that connects to Jellyfin API and am sharing some details on how to do it.

Thanks to James Harvey for his helpful article that got me started at: https://jmshrv.com/posts/jellyfin-api/

## Jellyfin API docs
You can find the swagger for your own Jellyfin instance at `http://localhost/api-docs/swagger/index.html`. The documentation there likely differs a bit from the official Jellyfin API Docs: https://api.jellyfin.org/

I also recommend you go look at the source code https://github.com/jellyfin/jellyfin/tree/master/Jellyfin.Api

But again, that's from the `master` branch so it'll likely be newer than your local instance.

## Authentication
To authenticate with the Jellyfin API there are a few different options. The easy one is `/Users/AuthenticateByName`.

This endpoint is POST and takes an authorization header that has 4 pieces of information:
 * Client: The name of your client
 * Version: Version number of your client
 * Device: The name of the device
 * DeviceId: An identifier for the current device

It also takes your username and password as the POST content.

```
const api = axios.create({ baseURL: 'http://localhost:8096' });

const deviceName = 'MyDevice'
const deviceId = 'SomeGuid' // Potentially generate one and save in local storage
const clientName = 'NameOfYourClientApplication'
const version = '0.0.1' // Version of your client application

const response = await api.post(
  '/Users/AuthenticateByName',
  {
    Username: 'bob',
    Pw: 'PlainTextIn2024?'
  },
  {
    headers: {
      // It is critical this header starts with 'MediaBrowser', Jellyfin expects it.
      Authorization: `MediaBrowser Client="${clientName}", Device="${deviceName}", DeviceId="${deviceId}", Version="${version}"`
    }
  }) 

// Grab UserId (needed for many other calls)
const userId = response.data?.User?.Id;
const token = response.data?.AccessToken;

// You can make the header default for future calls with axios like this:
api.defaults.headers.common['Authorization'] = `MediaBrowser Client="${clientName}", Device="${deviceName}", DeviceId="${deviceId}", Version="${version}" Token="${token}"`;
```

# Getting items from the API 
Items in Jellyfin are all of a gigantic basetype called [BaseItemDto](https://github.com/jellyfin/jellyfin/blob/054f42332d8e0c45fb899eeaef982aa0fd549397/MediaBrowser.Model/Dto/BaseItemDto.cs)

You can access instances of this type through endpoints such as `/Users/${userId}/Items`.

I'm building a simple music player so I just want all the music on my server:

```
const itemsResponse = await api.get(`/Users/${userId}/Items`, {
  params: {
    IncludeItemTypes: 'Audio',
    recursive: true,
  },
});

const allSongs = itemResponse.data.Items

```

# Playing music
Again since I'm building a music player, I need a URL to pass to an AudioElement.

To do that I access `/Audio/${item.Id}/universal?ApiKey=${apiToken}`.

Since you cannot pass headers to an audio element src tag. We pass the token using a query parameter.

```
const item = allsongs[0]
audioElement.src = `${api.getUri()}/Audio/${item.Id}/universal?ApiKey=${token}`
```



