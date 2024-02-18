What is WhatsApp ?
WhatsApp is a chat application that provides instant messaging service to its users. It is one of the most used mobile application on the planet, connecting over 2 billion users in 180+countries.Whatsapp is also available on the web.
Requirements
Our System should meet the following requirements
Function Requirements
1. Should support one-on-one chat.
2. Group Chat (Max 100 People)
3. Should support file sharing (image, video, etc)
Non-functional requirements
1. High availability with minimal latency.
2. The System should be scalable and efficient.
Extended requirements
1. Send, Delivered, and Read receipts of the message.
2. Show the last seen time of the users.
3. Push notification.
Estimation and Constrains
Let’s start with estimation and constraints
Traffic
Let us assume we have 10 million daily active users and on average each user sends at least text message 20 per day and message size 01 kb per text message .
 
Daily Text Message Traffic
10 million users × 20 message/users × 1 kb/message = 200 GB /day.
Multimedia message
 Assuming each user sends an average of 5 multimedia messages (image, videos, etc)
 Per day.
 Average multimedia message size: 500 KB.
 Daily Multimedia Message Traffic :
 10 million users × 5 message/users × 500 KB /message =25 TB
Voice Calls
 Assuming each user makes an average of 10 minutes of voice calls per day .
 Voice calls Bitrates: 24kbps
 Daily Voice Calls Traffic:
 10 million users × 10 minutes users × 24kbps × 1/8 to convert to KBps = 375 GB
Video Calls
 Assuming each user makes an average of 10 minutes of voice calls per day .
 Voice calls Bitrates: 500kbps
 Daily Voice Calls Traffic:
 10 million users × 10 minutes users × 500kbps × 1/8 to convert to KBps = 9.375 TB
Total Daily Traffic:
Summing up the daily traffic for text message, multimedia message, and voice calls, and video calls:
200 GB + 25 TB + 375 GB + 9.375 TB = 34.55 TB
This estimation provides an overview of the daily traffic generated by 10 million users engaging in various messaging activities. Keep in mind that these are rough estimates and actual traffic may vary based on user behavior and usage patterns. Continuous and analysis are essential for adjusting resources and infrastructure as user engagement change over time.
Kiloit(kb): A unit of digital information that represents 1,000 bits
Kilobyte (KB): A unit of digital information that represents.
(1,024 bytes = 8,192 bits)
What would be Request Per Second (RPS) for our system?
10 Million Request per day translates into 115.74 requests per second.
 2 Million / 24 hrs × 60 × 60 = 115.74 request / second 
Storage 
Summing up the daily traffic for text message, multimedia message, and voice calls, and video calls:
200 GB + 25 TB + 375 GB + 9.375 TB = 34.55 TB
And for 10 Years, we will require about 12,592.5 TB of storage.
( Total Traffic over 10 years = 34.55 TB × 365 days/years × 10 years ) 
Data model design
This is the general data model which reflects our requirements
Bandwidth
As our system is handling total traffic over 10 years and convert it into megabytes per second (MBps)
Total Traffic over 10 years = 15,592.5 TB
To convert terabytes to Megabytes, we use the fact that 1 terabyte is equal to 1,024 × 1,024 × 1,024 (MB)
Total Traffic in Megabytes = 12,592.5 × 1,024 × 1,024  × 1,024 = 1,36,27,854.4 MB
Average Bandwidth Usage Per Second = Total Traffic in Megabytes 1,36,27,854.4 / 365 × 24 × 60 × 60 =
0.431 MBps .
Therefore, the estimated average bandwidth usage per second over 10 years for the given messaging system with 10 million users is approximately 0.431 megabytes per seconds .
High-level estimate
Here is our high-level estimate:
Types
Estimate
Daily active users ( DAU
10 Million
Request Per Seconds (RPS)
115.74/s
Storage ( Per Day )
34.55 TB
Storage ( 10 years)
12,592.5 TB
Bandwidth
0.431 MBps
 
Data model design
This is the general data model which reflects our requirements

We have the following tables:
Users
This Table will contain a user’s information such as name, phone Number, and other details.
Messages
As the name suggests, this table will store with properties such as type (text, image, video, etc), content, and timestamps for message delivery. The message will also have a corresponding chatID or groupID.
Chats
This table basically represents a private chat between two users and can contain multiple messages.
Users_chats
This table maps users and chats as multiple users can have multiple chats (N:M relationship ) and vice versa.
Groups
This table represents a group made up of multiple users.
What kind of database should we use?
While our data model seems quite relation, we don’t necessarily need to store everything in a single database, as this can limit our stability and quickly become a bottleneck.
We will split the data between different service each having ownership over a particular table. Then we can use a relational database such as PostgreSQL or distrbiuted NoSQL database such as Apache Cassandra for our case .
API design
Let us do a basic API design for our service:
Get all chats or group
This API will get chats or group for a given userID .
 getAll( userID: UUID ) : chat[] | Group[]
Parameters
User ID ( UUID ) : ID of the current user.
 
Returns
Result (chat[] | Group[] ) : All the chats and groups the user is a part of .
Get messages
Get all message for a user given the channelID (chat or group id).
getMessages(userID:  UUID, channelID:  UUID): Message[]
Parameters
User ID (UUID): ID of the current user.
Channel ID ( UUID ): ID of the current user.
Channel ID (UUID): ID of the channel (chat or group) from which message need to be retrieved.
Returns
Message (Message[]): All the message in a given chat or group.
Send message
Send a message from a user to a channel (chat or group ).
 sendMessage(userID: UUID, channeled: UUID, message: Message): Boolean.
Parameters
User ID (UUID): ID of the channel (chat or group) user wants to send a message to.
Message (Message): The message (text, image, video, etc) that the user wants to send.
Returns
Result (boolean ): Represent whether the operation was successful or not .
Join or Leave a channel
Allows the user to join or leave a channel (chat or group )
joinGroup(userID : UUID, channelID: UUID) : Boolean
leaveGroup(userID: UUID, channelID: UUID) : Boolean
Parameters
User ID (UUID) : ID of the current users.
Channel ID (UUID): ID of the channel (chat or group ) the user wants to join or leave.
Returns
Results (boolean ) : Represents whether the operation was successful or not.
High-level design
Now let us do a high-level design of our system.
Architecture
We will be using microservice architecture since it will make it easier to horizontally scale and decouple services .
User Service
This is an HTTP-based that handles user-related concerns such as authentication and user information.
Chat Service
The chat service will use WebSockets and establish connections with the client to handle chat and group message-related functionality. We can also use cache to keep of all the active connections sort of like sessions which will help us determine if the user is online or not.
Notifications Service
This service will simply send push notifications to the users , it will be discussed in details separately .
Presence Service
The presence service will keep track of the last seen status of all users. it will be discussed in details separately.
Media service
This service will handle the media (image,video,file,etc) uploads. It wills be discussed in details separately.
What about inter-service communication and service discovery?
Since our architecture is microservices-based, services will be communication with each other as well.
Generally, REST or HTTP performs well but we can further improve the performance using gPRS which is more lightweight and efficient.
Service discovery is another thing we will have to take into account. We can also use a service mesh that enables managed, observable, and secure communication between individual services.
Note: Learn more about REST,GraphQL,gPRS and how they compare with other.
 
Real-time messaging
How do we efficiently send and receive message? We have two different options:
Pull model
The client can periodically send an HTTP request to server to check if there are any messages. This can be achieved via something like Long polling.
Push Model
The client opens a long-lived connection with the server and once new data is available it will be pushed to the client. We can use WebSockets or Server-Sent-Events (SSE) for this .
The pull model approach is not scalable as will create unnecessary request overhead on our servers and most of the time the response will be empty, thus wasting our resources. To minimize latency, using the push model with WebSockets is a better choice because then we can push data to the client once it’s available without any delay, given the connection is open with the clients ,Also, WebSockets provide full-duplex communication, unlike Server-Sent Events(SSE) which are only unidirectional .
Note: Learn more about Long polling, WebSockets, Server-Sent Events (SSE)
Notifications
Once a message is sent in a chat or a group, we will first check if the recipient is active or not, we can get this information by taking the users active connection and last seen into consideration.
If the recipient is not active, the chat service will add an event to a message queue with additional metadata such as the client’s device platform which will be used to route the notification to the correct platform later on. 
The notification service will then consume the event from the message queue and forward the request to FirebaseCloud Messaging (FCM) or Apple Push Notification Service (APNS) based on the clients device platform (Android, IOS, Web, etc). we can also add support for email and SMS.
Why are we using a message queue?
Since most message queues provide best-effort ordering which ensure that message are generally delivered in the same order as they are sent and that a message is delivered at least once which is an important part of our service functionality .
While this seems like a classic publish subscribe use case, it is actually not as mobile devices and browsers each have their own of handling push notification . Usually ,notification are handle externally via Firebase cloud Messaging (FCM) or Apple Push Notification Service (APNS) unlike message fan-out which we commonly see in backend service. We can use something like Amazon SOS or RabitMQ to support this functionality.
Read Receipts
Handling read receipts can be tricky, for this use case we can wait for sort of Acknowledgment (ACK) from the client to determine if the message was delivered and update the corresponding deliveredAt field. Similarly, we will mark the message as seen once the user opens the chat and update the corresponding seenAt timestamp field.
Design
Now that we have identified core components, let’s do the first draft of our system design.
Detailed design
It’s time to discuss our design decisions in details.
Data partitioning
To scale out our databases we will need to partition our data. Horizontal partitioning can be a good first step. We can use partitions schemas such as:
·         Hash-based Partitioning
·         List-based Partitioning
·         Range-based partitioning
·         Composite Partitioning
The above approach can still cause uneven data and load distribution, we can solve this using consisting Hashing for more details, refer to sharding and consisting Hashing.
Caching
In a messaging application, we have to be careful about using cache as our users expect the latest data, but many users will be requesting the same message especially in a group chat. So,to prevent usage spikes from our resources we can cache older message .
Some group chats can have thousand of message and sending that over the network will be really inefficient; to improve efficiency we can add pagination to our system APIs. This decision will be helpful for users with limited network bandwidth as they won’t have to retrieve old message unless requested.
Which cache eviction policy to use?
We can use solution like Redis or Memcached and cache 20% of the daily traffic but what kind of cache eviction policy would fit our needs?
Least Recently Used (LRU) can be a good policy for our system .In this policy, we discard the least recently used key first.
How to handle cache miss?
Whenever there is a cache miss, our servers can hit the database directly and update with yhe new entries. For more details refer to Caching.
 
Media access and storage
As we know, most of our storage space will be used for storing media files such as images, videos,or other files. Our media service will be handling both access and storage of the user media files.
Content Delivery network (CDN)
Content Delivery Network (CDN) increase content availability and redundancy while reducing bandwidth costs, Generally ,static files such as images, and videos are served from CDN. We can use services like Amazon Cloudfront or cloudflare CDN for this use case.
Notion link site: Link
https://askur.notion.site/askur/Case-Study-WhatsAPP-e631b5e359fc4fd698ec65d4da2f14f6
