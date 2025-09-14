# Pset 1

## Exercise 1 Questions

1. For any request, the total number of purchases for that item can never exceed the request’s count. A purchase in a registry is associated with an existing request in that registry. It is more important that a purchase that does not exceed the requested amount since this affects user's purchases and may cause them to overbuy an item. The purchase action is most affected by it, and it preserves this invariant by requiring that the count of an item to be purchased has an active request that has at least that count.
2. You can add negative numbers of items through addItem which can cause users to purchase negative amounts of an item if the number of items requested is more negative, and you can fix this by adding a requirement that count is positive.
3. Yes, a registry can be repeatedly opened and closed since this may be useful to the user if they keep changing their mind about whether to make their registry public.
4. In practice, this would not affect the main operational principle (purchase/request workflow still works), but if the user makes many registries that they no longer wnat, they may want to delete them.
5. Queries for all open or closed registries of a user, and queries for all purchases by a user.
6. Add an additional flag called "hidden" to each Registry that, when set to true, hides purchases for that registry.
7. This allows for more flexibility and accuracy when specifying these types. For example, an issue may occur if specifying users/items by traits rather than unique codes results in not being able to uniquely identify each user or item. Additionally, forcing all users and items to have specific traits may cause issues in having to handle edge cases where an item or user is missing a trait, for example.

## Exercise 2 Questions

1.

```
state
    a set of Users with
        a Username
        a Password
```

2.

```
actions
    register (username: String, password: String): (user: User)
        requires User with that username does not exist
        effects create a new User with username and password and returns that newly created user

    authenticate (username: String, password: String): (user: User)
        requires User with that username exists
        effects returns User associated with username if password matches, otherwise fails
```

3. Each username must be unique, and this is preserved by registration (adding a User to the state), since registration only succeeds if the username is not already existing.
4.

```
state
    a set of Users with
        a Username
        a Password
        a confirmed Flag
        a secret Token
actions
    register (username: String, password: String, email: String): (token: String)
        requires User with that username does not exist
        effects generates and returns a secret token and emails it to user's email via a sync; create a new User with username and password and the secret token

    confirm (username: String, token: String): (user: User)
        requires User with that username exists and has confirmed flag set to false
        effects if token matches the token of the user with username, sets that User's confirmed flag to true and returns the user, otherwise it remains false and confirmation fails

    authenticate (username: String, password: String): (user: User)
        requires User with that username exists and has confirmed flag set to true
        effects returns User associated with username if password matches, otherwise fails
```

## Exercise 3

```
concept PersonalAccessToken[User]
purpose authenticate users to GitHub via revokeable tokens as an alternative to using passwords
principle user names a new token, gives it an expiration date and sets the token's scopes to define what resources the token has access to; then the user generates the token and copies it to be used; the token can later be deleted from the list of the user's personal access tokens
state
a set of Tokens with
    a string Name
    an expiration Date
    a set of Scopes
actions
    createToken(user: User, name: String, expiration: Date, scope: [String]): (token: Token)
        requires user exists and token with same name does not exist
        effects generates token with given name, expiration date, and scopes, adds to user's Tokens, return token

    authenticate(username: String, token: Token, scope: String): (user: User)
        requires user exists and token exists for the user
        effects return user if token is found in user's Tokens AND the current date is before the expiration date AND the scope is in the token's list of scopes, otherwise fails

    deleteToken(user: User, token: Token)
        requires token is in the user's Tokens
        effects removes token
```

PersonalAccessTokens differ in that you can specify more granular levels of permissions (via scopes) than a Password. I think the GitHub page could be improved by specifying the scopes that regular PasswordAuthentication gives access to vs the scopes that the PersonalAccessToken gives access to.

## Exercise 4

### URL Shortener

```
concept URLShortener[URL]

purpose given a URL, provides a shortened version of it that redirects to the original URL

principle user provides a long URL for a shortened URL that will redirect to the long URL and optionally provides the shortened URL suffix; URL shortener provides shortened URL (and supplies suffix if user did not provide)

state
    a set of ShortenedURLs with
        a string LongURL

actions
    shortenURL(longURL: String, suffix: String): (shortenedURL: String)
        requires a valid, nonempty URL
        effects if suffix is nonempty, generates and returns a shortened URL with this suffix if the resulting shortened URL does not already exist, otherwise generates its own suffix for a shortened URL (that does not already exist) and returns it; adds shortenedURL

    redirectURL(shortenedURL: String): (longURL: String)
        requires shortenedURL exists and has associated LongURL
        effects returns longURL associated with shortenedURL and redirects shortenedURL to this longURL
```

### Billable Hours Tracking

```
concept BillableHoursTracker[Client, Employee, Project]

purpose tracks hours worked by employees on projects

principle user provides a long URL for a shortened URL that will redirect to the long URL and optionally provides the shortened URL suffix; URL shortener provides shortened URL (and supplies suffix if user did not provide)

state
    a set of ShortenedURLs with
        a string LongURL

actions
    shortenURL(longURL: String, suffix: String): (shortenedURL: String)
        requires a valid, nonempty URL
        effects if suffix is nonempty, generates and returns a shortened URL with this suffix if the resulting shortened URL does not already exist, otherwise generates its own suffix for a shortened URL (that does not already exist) and returns it; adds shortenedURL

    redirectURL(shortenedURL: String): (longURL: String)
        requires shortenedURL exists and has associated LongURL
        effects returns longURL associated with shortenedURL and redirects shortenedURL to this longURL
```

```
concept BillableHoursTracker[Employee, Project]

purpose tracks hours employees worked on projects

principle an employee starts and ends work sessions on a project, automatically ending a session after; the tracker records them and totals time for billing

state
    a set of Sessions with
        an Employee
        a Project
        a string Description
        a startTime
        an endTime
        an autoCloseTime

actions
    startSession(employee: Employee, project: Project, description: String, autoCloseTime: Date): (session: Session)
        requires project exists and employee exists, description should be nonnull, and autoCloseTime is after current time
        effects creates a new session with employee, project, description, auto close time; startTime set to now, endTime = null; returns this session

    endSession(session: Session, endTime: Date)
        requires session exists and endTime is null
        effects sets session's endTime to endTime

    autoClose(session: Session)
        requires session exists, endTime is null, and autoCloseTime is nonnull
        effects sets session's endTime to its autoCloseTime

    totalHours(project: Project): (hours: Number)
        requires project exists
        effects returns sum of (endTime - startTime) across all completed sessions for this project
```

### Conference Room Booking

```
concept RoomBooking[Room, User]

purpose allocate rooms for meetings without time conflicts

principle a user reserves a room for a time interval; the system prevents overlapping bookings for the same room

state
    a set of Bookings with
        a room Room
        a user User
        a startTime Time
        an endTime Time

actions
    reserveRoom(user: User, room: Room, startTime: Time, endTime: Time): (booking: Booking)
        requires room exists and no existing booking for room overlaps [startTime, endTime)
        effects creates a booking with these details and adds it, otherwise fails to add booking

    cancelBooking(user: User, booking: Booking)
        requires booking exists and is owned by user
        effects removes booking from set of bookings

    listBookings(room: Room): (bookings: Set of Bookings)
        requires room exists
        effects returns all bookings for this room
```
