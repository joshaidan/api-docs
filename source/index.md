---
title: API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='https://api.planningcenteronline.com/oauth/applications'>Sign Up for an Authentication Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - people
  - services
  - check_ins
  - giving
  - errors

search: true

# _
---

# Introduction

The Planning Center Online API can be used to interact with all of your organization's data inside Planning Center. In this first iteration of the API the data inside PCO People is available; we are now working towards adding endpoints for some of our other apps (including an all new API for Services).

# Rate limiting

> Requests that are rate limited will return an error like this:

```
{
  "errors": [
    {
      "code": "429",
      "detail": "Rate limit exceeded: 118 of 100 requests per 60 seconds"
    }
  ]
}
```

The API is rate limited to 100 requests per minute per user. You can see our count of your API rate limiting by inspecting the `X-PCO-API-Request-Rate-Count`, `X-PCO-API-Request-Rate-Limit` & `X-PCO-API-Request-Rate-Period` HTTP headers. Requests that exceed that limit will return a HTTP status 429.

# Authentication

There are several ways to authenticate with the API.

## Session Authentication

You (the developer) can browse the API via a browser using your session cookie:

1. Visit [accounts.planningcenteronline.com](https://accounts.planningcenteronline.com) and sign in.
2. Visit [api.planningcenteronline.com/people/v2/people](https://api.planningcenteronline.com/people/v2/people)
   (for example) or other API endpoints.

Browsing the API with your web browser is best experienced with an extension like JSONView, which formats
JSON output in the browser for easier reading.

Note: session auth won't work for building an application for users because of the same-origin policy.

## Personal Access Token

> You can pass your personal access token with `curl` like this:

```
curl -u app_id:secret https://api.planningcenteronline.com/people/v2/people
```

Create a Personal Access Token and use HTTP Basic Auth if you just need access to your own account:

1. Visit [api.planningcenteronline.com/oauth/applications](https://api.planningcenteronline.com/oauth/applications).
2. Create a new "HTTP Basic Application"
3. Pass your token and secret in every request using [HTTP Basic Auth](https://en.wikipedia.org/wiki/Basic_access_authentication).

## OAuth 2.0

If you are distributing your app to multiple churches, you should use [OAuth](https://en.wikipedia.org/wiki/OAuth) version 2.0.

1. Visit [api.planningcenteronline.com/oauth/applications](https://api.planningcenteronline.com/oauth/applications)
   and create an OAuth application token.
2. Make sure your app conforms to the [OAuth 2.0](http://oauth.net/2/) specification.

OAuth access tokens expire after 2 hours from the time they are issued, but we also provide a [refresh token](https://tools.ietf.org/html/rfc6749#section-1.5) you can use to get a new access token at any time, even after the access token expires, without forcing the user to re-authorize.

We will honor refresh tokens for up to 1 year after the date its associated access token was issued. This means that, if your app does not issue a refresh for 1 year, then your user will need to re-authorize.

We have an example Ruby/Sinatra app showing how to obtain and use access tokens and refresh tokens at [github.com/planningcenter/pco_api_oauth_example](https://github.com/planningcenter/pco_api_oauth_example).

### Scopes

Each PCO app is a distinct [OAuth scope](http://tools.ietf.org/html/rfc6749#section-3.3).


| App | Scope|
|-----|------|
| People | people |
| Services | services |
| Check-ins | check_ins |

When authorizing, you can request scopes using the `scope` parameter. The value should be a space-separated list of scopes.

# JSON API

This API is built to conform to the [JSON API](http://jsonapi.org/) 1.0 specification.

Some benefits:

* Excellent documentation is already available [here](http://jsonapi.org/format/) and we do our best to stay true to it.
* Our API returns resources that contain links to related resources, making it easier to "browse" this API and learn about associated endpoints.
* You can include related resources by adding `?include=something` to the URL. This makes grabbing what you need much simpler.

Gotchas:

* Resources are wrapped in an object that looks like this: `{ "data": { "type": "Thing", "id": "1", "attributes": { ... } } }`.
* Creating or updating resources also requires that same structure. Be sure to pass the "type" key and put the resource's attributes _inside_ the "attributes" object.

# Dates & Times

All dates and times conform to the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) standard and look like this:

| value         | format               | example              |
| ------------- | -------------------- | -------------------- |
| date          | YYYY-MM-DD           | 2015-05-05           |
| date and time | YYYY-MM-DDTHH:MM:SSZ | 2015-05-05T22:40:07Z |

<aside class='info'>Times are always returned in UTC.</aside>

# File Uploads

> Here's how to upload a file with curl:

```
curl -u app_id:secret -X POST -F 'file=@/path/to/file.png' https://upload.planningcenteronline.com/v2/files
```

> The response you receive will look like this:

```
{
   "data": [
      {
         "type": "File",
         "attributes": {
            "source_ip":    "70.128.100.145",
            "md5":          "e178604e5083cc23782f34650a49b025",
            "content_type": "image\/png",
            "file_size":    48341,
            "name":         "file.png",
            "expires_at":   "2016-05-23T23:13:32Z"
         },
         "id": "us1-16207df7-b6cc-4abe-ca1a-306c6f7e423d"
      }
   ]
}
```

> To assign the file to an attribute, pass the UUID as a string. For example, here is how to update the Person avatar:

```
curl -u app_id:secret -X PATCH https://api.planningcenteronline.com/people/v2/people/1 -d '{"data":{"attributes":{"avatar":"us1-16207df7-b6cc-4abe-ca1a-306c6f7e423d"}}}'
```

Some endpoints accept file uploads. This is a two-step process:

1.  Upload a file by making a `POST` request to `https://upload.planningcenteronline.com/v2/files`.

    Your request should use `multipart/form-data` encoding and contain a `file` attribute with the file data. (See the sidebar for a `curl` example.)

    The response you get will contain an `id` which is the file UUID.

2.  Pass the UUID as the attribute value to an endpoint that accepts a file.

    The UUID is a string that tells the endpoint which file to associate.

# Libraries

Since our API talks HTTP and JSON, you get to use off-the-shelf components for your language of choice.

Since we love Ruby so much, we went ahead and built a very simple API wrapper library called
[pco_api](https://github.com/planningcenter/pco_api_ruby). `gem install` it, and you'll be building
cool stuff in no time!
