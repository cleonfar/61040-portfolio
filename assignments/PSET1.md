**Exercise 1**

  1.  One invariant is the count number from the requests + the sum of the count numbers from the purchases will always total to the total count of the addItem actions on the item corresponding to that request. The purchase action is affected by this and prevents purchasing more than the count in requests, and will decrement the request count by the amount added to the purchase count to maintain this.

        Another invariant is that all purchases must correspond to a request. The purchase action checks to make sure that the 
    item they are attempting to purchase is a requested item. This one seems more important to me because it prevents unrequested things from being purchased. 

  2.  The removeItem action could break the second invariant from above because if the item being removed has already been purchased then there is end up being purchases for an item that is not requested anymore. One fix for this would be to prevent removing and item that already has purchases.

  3.  The spec definitely allows the registry to be opened and closed repeatedly. I suppose one reason for this could be for convenience for the user, if they accidently open the registry at the wrong time, they may want to be able to close it and reopen it later rather than being forced to completely remake and open a new registry. 

  4.  Possibly. From the user's perspective the difference between a closed registry and a deleted registry probably doesn't matter, but if no registries are ever deleted it's possible there could end up being a storage issue with a lot of dead registries clogging up space somewhere.

  5.  Registry owners will probably be querying the count of purchases that has been made to track progress, while gift givers will query the count of requests on items to identify what still need to be bought.

  6.  Maybe add a flag to the set of registries that indicated whether or not the purchases are hidden or not, and if they are hidden prevent the owner from being able to view the purchases. Also add actions to toggle the flag back and forth so the owner can still get access to the purchases if they need it.

  7.  Generic types are preferable because it allows a layer of abstraction so that the concept does not have to know the exact way that something is implemented, just that it represents what it needs to represent.


**Exercise 2**

  concept: PasswordAuthentication  
  purpose: limit access to known users  
  principle: after a user registers with a username and a password,  
    they can authenticate with that same username and password  
    and be treated each time as the same user  
  state  
    a set of Users  
    Each user has: a username, a password, a secret token, and a confirm flag  

  Actions:  

    register (username: String, password: String): (user: User)  
      Requires: The user is not in the set of users and no user other user has the given username  
      Effects: Adds the user to the set of users with that username and password. Also generates a secret token for that user  and emails it to them. Sets the user's confirm flag to false.  
    confirm (username: String, secret token: string):  
      Requires: The username is in the set of users. The secret token is equal to the secret token of that user. The confirm  flag is false.  
      Effects: Sets the user's confirm flag to true.  
    authenticate (username: String, password: String): (user: User)  
      Requires: There is a user in the set with a matching username and password  
      Effect: Authenticates the sender of this action as the corresponding user  
  3.  It must hold that all usernames are unique. This is preserved by preventing new users from registering with a username  that has already been used.  


**Exercise 3**

  concept: PersonalAccessToken
  purpose: To allow a person access to full or partial resources allowed to that user depending on the token.  
  principle: A user sets up a personal access token with certain permissions.  
    They can then use that token to access the resources that that token has permissions for.  
  state:  
    A set of users  
    each user has an associated username, token, and permissions  
  Actions:  
    register  (username: String, permissions: String): (user: User, token: String)
      Requires: The user is not in the set of users and no user other user has the given username  
      Effects: Adds the user to the set of users with that username and permission level.  
      Also generates a token a returns it to the user.
    authenticate (username: String, token: String): (user: User)  
      Requires: There is a user in the set with a matching username and token  
      Effect: Authenticates the sender of this action as the corresponding user  

**Exercise 4**

  concept: TrackBillableHours
  purpose: To allow a user to track the billable hours of a client.  
  principle: After a user sets up a project, they can start and end work sessions  
  within the project to track time spent and work done.
    They can then use that token to access the resources that that token has permissions for.  
  state:  
    A set of projects with:  
    A client name  
    An hour count  
    An hourly price  
    A set of sessions  

    A set of sessions with:  
    An active flag
    A start time  
    An end time  
    A description of work to be done  

  actions  
    startProject  (name: String, client: String, rate): (project: Project)
      Requires: Project name is unique
      Effects: Adds a project to the set of projects with the given name, client, and rate
    startSession (name: String, description: String):  (session: Session)
      Requires: There is a project with the given name and the description is unique to the project    
      Effect:   Creates a session with the given description, the current time as start time, and the active flag set to true. Adds the session to the set of sessions.  
    endSession (name: string, description: string)  
      Requires: The session with the given description is active  
      Effects:  Sets the session's active flag to false and sets the end time to the current time  
    editSession: (name: String, description: String, startTime: string, endtime: string):
      Requires: There exists a closed session with the given description and a start and end time.
      Effects adjusts the session's start and end time to the give times.  
  
  concept: ConferenceRoomBooking
  purpose: To allow users to book a time to use a conference room.  
  principle: A user checks a conference room for available time slots.  
  state:  
    A set of conference rooms with:  
    A range of bookable hours  
    A set of booked times  
  
    A set of booked times with:  
    The start and end time of the booking    
    The name of the person who made the booking  

  actions  
    addRoom  (roomName: string, openTime: time, closeTime: time): (room: Room)
      Requires: There is no room with this name.     
      Effects: Adds a room to the set of bookable rooms with bookable times with the range of the open and close times.
    bookRoom (roomName: string, username: String, startTime: time, endTime: time): (booking: Booking)  
      Requires: The room has no booked times that overlap with the start and end time   
      Effect: Adds a booking to the set of booking in that room with the given start time, end time, and user.  
  
  
  
  concept: URLShortener
  purpose: To allow users create a short easily shareable link that redirects to the original longer URL.  
  principle: After a user submits a long URL, a short url that redirects to the link is generated and given to the user    
  state:  
    A set of links with:    
    Original URLS
    Short URLS
  actions  
    Create URL(originalURL: String): (shortURL: String)  
      Requires:   
      Effect: Creates a unique short URL that redirects to the orginal URL  