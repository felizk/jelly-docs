# Coding for Jellyfin API

I'm writing a thing that connects to Jellyfin API and am sharing some details on how to do it.

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
const response = await api.post(
  '/Users/AuthenticateByName',
  {
    Username: 'bob',
    Pw: 'PlainTextIn2024?'
  },
  {
    headers: {
      // It is critical this header starts with 'MediaBrowser', Jellyfin expects it.
      Authorization: 'MediaBrowser Client="MyClient", Device="MyDevice", DeviceId="SomeGuid", Version="0.0.1"'
    }
  }) 

  // Grab UserId (needed for many other calls)
  const userId = response.data?.User?.Id;
  const token = response.data['AccessToken'];
  this.api.defaults.headers.common['Authorization'] = this.makeAuthString();

```
