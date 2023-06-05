---
title: Уведомления
description: Receiving notifications about any actions with the bot on BotiCord
---

# Notifications

There are two ways to receive notifications on Boticord: WebSocket and HTTPs WebHooks. More about them:

## WebSocket

[WebSockets](https://ru.wikipedia.org/wiki/WebSocket) are a basic way to receive notifications on actions with bots.

### How to connect

In short:

1. `wss://gateway.boticord.top/websocket/`;
2. client hello: `{"event":"auth","data":{"token":"token from user settings"}}`;
3. server hello: `{"event":"hello","data":{"id":"connection id"}}`;
4. server notification:

```json
{
  "event": "notify",
  "data": {
    "type": number
    event
    types,
    "payload": any
    json
    data
    or
    null,
    "affected": string
    affected
    resource
    id,
    "user": string
    id
    of
    the
    user
    who
    triggered
    this
    notification,
    "happened": number
    unix
    timestamp
    ms
  }
}
```

In detail:

1. Your program (client) needs to instantiate a connection to `wss://gateway.boticord.top/websocket/`;
2. Client must send a valid JSON message: `{"event":"auth","data":{"token":"..."}}`. Field `token` must contain a token, which you can get in bot's owner settings on website. `https://boticord.top/me` In settings a little bit below you must generate a new token.
3. After the client send a message to server, server will reply with `{"event":"hello","data":{"id":"your client's special id (not a bot, not a server.)"}`
4. If server doesn't respond or sends something else, try again later or fix the error reported to you by the server (usually the wrong token or no token was provided at all)
5. Server will send notifications in the following format:

```json
{
  "event": "notify",
  "data": {
    "type": number, one of types of events. (check NotifyTypes),
    "payload": any additional data in the JSON (such as the content of a comment), or null,
    "affected": string, id of the resource that the event is about, for example, id of a comment,
    "user": string, member id of user that triggered the notification,
    "happened": number, the UNIX timestamp in milliseconds when the event occurred
  }
}
```

#### How do disconnections happen

If the token in bot's settings was deleted or regenerated, server will teminate the WebSocket connection with the exit code `1006`. In any other situation disconnecting is not performed, however client can be safely disconnected at any time. If connection was lost, your client must try to reconnect until the connection is fixed. You must take the global rate-limit of 15 attempts/10s into consideration. After that you have to repeat authorization.

## HTTPs WebHooks

Also, Boticord can send notifications through HTTPs, by sending POST requests to the address that you have provided.

### Preparing your server

In short:

1. Open port 80 for HTTP or 443 for HTTPS, for subnet `2a06:98c0::/29` (AS132892 - Cloudflare, Inc);
2. Check if IPV6 works;
3. Receive POST requests at the address specified in bot settings (API tab);
4. Check the correctness of requests with token from bot's settings, which will be passed in the headers.

In detail:

1. BotiCord uses [IPV6](https://habr.com/ru/company/droider/blog/568778/) addresses of subnet `2a06:98c0::/29` (AS132892 - Cloudflare, Inc) for sending notifications. This means that any IPV4 addresses **do not work**. You will not be able to receive notifications if you try to do so via direct IPV4. This includes, for example, your PC's and IPV4 issued by your ISP, as well the cloud servers provided with IPV4. If you have a cloud server, then you most likely have a domain or the ability to connect to a tunnel broker. In case you use Cloudflare for your domain, just create a subdomain or a transform rule or create an endpoint in your API, if you have it.
2. You have to create a server which will be able to receive our POST requests and work 24/7. Also, your server must respond with the 200 status code, if the request was processed successfully. Your server must not be slow or the notification will not reach your server. An attempt to send a notification is only made once. If your server was unable to receive a notification, it will not be sent again. Notifications are sent to both WebSocket and webhook.
3. We have added the ability to add additional headers to our POST requests to your servers. You can use it, for example, to authorize our server in [Cloudflare Access](https://www.cloudflare.com/products/zero-trust/access/). You can also add special headers with unique values to verify that the request was administered by us. By default, we expect you to check the validity of the request, using the token we pass along with the request in `X-Boticord-Token` header.
4. **We don't recommend using HTTP instead of HTTPS for server, that will receive the notifications.** If you use Cloudflare make sure that the connections between Cloudflare and your server is secure and goes through the 443 port. ***We pass the bot's token in the `X-Boticord-Token` header as a confirmation that the request was administered by us. ***Not only is the token used for receiving notifications, it is also used for changing your bot's settings in BotiCord.****** Take the security of your token seriously.
5. You must open port 80 for HTTP or 443 for HTTPS, for subnet `2a06:98c0::/29` in order to receive notifications. You can check that whether requests are coming through using the button in the settings. It will send a POST request to the specified address with a test notification.
6. Receive requests and check their validity with token from bot's settings on BotiCord which comes with the notification:

```txt
Content-Type: application/json
X-Boticord-Token: bot's token
X-Boticord-Affected: the identifier of the affected resource
... your headers
```

request body:

```json
{
  "type": integer, one of events type, see NotifyTypes,
  "payload": any extra params in JSON (for example, comment content), or null, or property is empty,
  "affected": string, id of the resource that the event is about, for example, id of comment,
  "user": id, id of user that triggered the notification,
  "happened": integer, UNIX timestamp in milliseconds when the event occured
}
```

## NotifyTypes

TBA
