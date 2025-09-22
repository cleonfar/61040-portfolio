**Concept Questions**

 1. The concept may want multiple contexts because users would want unique strings in a given context but likely would be fine with generated nonces matching in different concepts. In the context of the URL shortening app it is important that the nonces generated are unique to prevent a short URL from being associated with multiple different long URLs, because short URLs are supposed to redirect to at most 1 long URL in the URL shortening concept.  

 2. It must store the used strings so that it knows what nonces have been generated and avoid generating a nonce that is already in use, which would not be desirable in the context of UrlShortening.  

 3. One obvious advantage from a user's perspective would be that the shortened Url is easier to remember because it is a common word. It also makes it easier to share verbally with others because a word is easier to communicate than a random string. A disadvantage for users could be that if the words are generated randomly, they will not have any meaning to the user and not be as memorable as it would seem, or if they are not randomly generated there could be issues with desired keywords being taken because the nonces are limited to simple dictionary words. To modify the NonceGeneration concept to reflect this, I would change the principle and purpose to mention the unique strings are simple dictionary words, and alter the generate action to pick and return an unused simple word from a dictionary.  


**Synchronization Questions**

 1. Because generating the Nonce to go with a shortUruBase does not rely on the targetUrl, so it is not necessary to include it in the first sync, while in the second sync registering does require both the shortUrlBase and the targetUrl, so both have to be provided.  

 2. I assume this convention is not used in every case for clarity. While in some cases it can be nice to avoid uneccessary writing, there can be cases where the omission could make the specification unclear to the reader and cause confusion.  

 3. I believe that the first two have request included because those are calls to access some piece of state in the concept, while the third uses calls to actions in the concepts.  

 4. If I'm understanding correctly I think the generate sync might not be necessary if alternate domain names were not supported. If alternate domain names are not supported then all nonce's will be generated in the same context, so it's not necessary to have the sync between the request for the shortUrlBase and the nonce generation.  

 5. **sync** delete  
 **when** ExpiringResource.System.expireResourse() : (resource)  
 **then** UrlShortening.delete(resource)


 **Extending the Design**

 1. 
 **concept** ShorteningUsage  
 **purpose** Count the uses of a shortening  
 **principle** When a shortening is accessed the usage count for that shortening is incremented by 1  
 **state**  
a set of Shortenings with  
 a user  
 a count  
 a shortUrl  
**actions**  
addShortening (shortUrl: string, user: User)
**requires** No shortening with the given shortUrl in the set of shortenings  
**effect** adds a shortening to the set of shortenings with the given user and shortUrl and an initial count of 0  
increment (shortUrl: string)  
**requires** The shortUrl is the in set of shortenings  
**effect** Increments the count of the given shortening by 1  
getCount (shortUrl: string, user: User): (count: Int)  
**requires** The shortUrl is in the set of shortenings and the user is the one associated with the shortUrl  
**effect** Returns the count of the shortening with the given shortUrl  
delete (shortUrl: string, user: User)  
**requires** The shortUrl is in the set of shortenings and the user is the one associated with the shortUrl  
**effect** Removes the shortening from the set  

 **concept** VerifyUser  
 **purpose** Verify a user's identity  
 **principle** When a user verification is needed, returns true if the user being verified is the current user  
 **state**  
 None
 **actions**
 verify (validUser: User, currentUser: User): (allowed: Boolean)
 **requires** None  
 **effect** Returns true if the users are the same and false otherwise
  
2. 
**sync** addShortening  
**when** UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase, targetUrl): (shortUrl: string)  
**then** ShorteningUsage.addShortening(shortUrl, user)  

**sync** increment  
**when** UrlShortening.lookup (shortUrl: string): (targetUrl: string)  
**then** ShorteningUsage.increment(shortUrl: string)  

**sync** verify  
**when** ShorteningUsage.getCount (shortUrl: string, user: User): (targetUrl: string)  
**then** VerifyUser.verify(currentUser, validUser)  

3. To allow users to choose their own short URLs, you could probably remove the register sync and instead have users directly register their own UrlShortening.  

The word as nonce strategy would just require an alteration to the NonceGeneration.generate action so that it picks words from a dictionary.  

This would just take a change to the state of the ShorteningUsage concept so that it holds a set of target Urls with a count and a set of shortenings associated with the target Urls, and some alterations to the parameters the actions take to reflect this change.  

This would just require a change to the NonceGeneration.generate action so that the nonces it generates fit some requirements that would make it difficult to guess.  

I suppose a change to the ShorteningUsage concept could allow for an optional phone number or email as a alternative to a registered user and allow for access to analytics by adding a confirmation by text or email. Alternatively, it could be argued this is not a desirable feature because it could add security risks.