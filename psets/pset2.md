# Pset 2

## Concept Questions

1. The contexts are so that there are sets of strings that are separate for each use case of NonceGeneration (so that it can be used across multiple other concepts). In the URL shortening app, a context is the concept of URL shortening, since all of the shortened URLs have to be unique. Having other contexts allows a string in one context (like URL shortening) to be used in a different context that's completely different from URL shortening, allowing the concept of NonceGeneration to be reusable.
2. NonceGeneration has to store sets of used strings to make sure none of the old strings are reused when generating a new one. The abstraction function maps the set of used strings to a number for each string that is equal to the counter at the time that the string was added.
3. The advantage is that it is easier to remember and type out for the user, but the disadvantage is that a word may already be taken when the user tries to register it (and unique words could run out depending on how much this needs to scale).

## Synchronization Questions

1. When generating a unique shortened URL (nonce), we don't need the targetUrl (the long version), but it is needed to register the match between the short and long URLs in order.
2. This convention isn't used in every case since sometimes a names are necessary for clarity when the variable name is different from the argument or is tied to a variable being returned by or being used as an argument to a different action.
3. The user doesn't need to request for a url to be shortened for the expiry to be set for a url (it's done by the system, when the new short url is being registered).
4. `shortUrlBase` should be hardcoded to be "bit.ly" in the syncs.
5.

```
sync expireShortUrl
when ExpiringResource.expireResource(): (resource)
then UrlShortening.delete(resource)
```

## Extending the Design

1.

```
concept ShorteningAnalytics [ShortUrl]
purpose count number of accesses for a shortened URL
principle each short url has associated access count that is incremented whenever it is accessed, and this count can be retrieved at any time
state
    a set of UrlToAnalytics with
        a short url String
        an access count Number
actions
    register(shortUrl: String)
        requires short url is not already registered (exists in UrlToAnalytics)
        effects adds shortUrl initialized with count number 0
    increment(shortUrl: String)
        requires the short url exists
        effect add 1 to the short url's access count number
    getCount(shortUrl: String): (count: Number)
        reuires the short url exists
        effect returns the access count number associated with the short url
```

```
concept UserOwnership [ShortUrl]
purpose track which user owns each short URL
principle after a user registers a shortened URL, they become the owner of the shortened url
state
    a set of ShortUrlToUser with
        a short url String
        a userId String
actions
    register(shortUrl: String, userId: String)
        requires short url does not already exist in ShortUrlToUser
        effect adds ShortUrl with the associated userId
    getURLGivenUserId(shortUrl: String, userId: String): (url: String)
        requires short url exists
        effects returns shortUrl if userId is owner of short url (associated with the short url in ShortUrlToUser) otherwise fails
```

2.

```
sync analyticsOnRegister
when UrlShortening.register(): (shortUrl)
then ShorteningAnalytics.register(shortUrl)
     UserOwnership.register(shortUrl, userId)
```

```
sync analyticsOnLookup
when UrlShortening.lookup(shortUrl): (targetUrl)
then ShorteningAnalytics.increment(shortUrl)
```

```
sync userRequestsAnalytics
when Request.getAnalytics(shortUrl, userId)
     UserOwnership.getURLGivenUserId(shortUrl, userId): (url)
then
    ShorteningAnalytics.getCount(shortUrl: url): (count)
```

3.

- Allowing users to choose their own short URLs;
  - Add argument to UrlShortening.register and sync with the short URL to validate that the short URL has not already been used
- Using the “word as nonce” strategy to generate more memorable short URLs;
  - Change implementation of NonceGeneration's generation action to pick a random word instead of generating a random string
- Including the target URL in analytics, so that lookups of 0 different short URLs can be grouped together when they refer to the same target URL;
  - Add a mapping of long urls to lists of short urls to the state of ShorteningAnalytics and add a new action so that when a short url is looked up, it and all the other urls in the same list have their access counts summed
- Generate short URLs that are not easily guessed;
  - Change implementation of NonceGeneration's generation action to generate completely random alphanumeric strings that cannot be words
- Supporting reporting of analytics to creators of short URLs who have not registered as user.
  - This is not recommended since it violates the privacy of the owner of the short URL
