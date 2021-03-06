# Signed Backend for NodeJS and AWS

The backend is built using AWS Lambda, DynamoDB, S3, and SQS.

## Deployment

## Lambda
We use the same Lambda for all API requests and SQS messages to minimize Lambda cold starts. The Lambda contains an internal router that invokes a module that corresponds to the request type

## DynamoDB structure
The 'signed' table is defined as:
- primary key: PK string
- sort key: SK string

Here is the list of all used combinations of PK/SK
- all-sources / source-[address] - sourceJSON
- posts-from-[address] / post-[createdAt]-[address]-[id] - postJSON, replies count, likes count, reposts count
- hash-[hash] / post-[createdAt]-[address]-[id] - type, size, mime-type, username, uploadedAt
- tag-[tag-name-base64] / post-[createdAt]-[address]-[id] 
- reply-to-[post-hash] / post-[createdAt]-[address]-[id] - replies to a particular post version
- like-[post-hash] / post-[createdAt]-[address]-[id] - likes of a particular post version
- repost-[post-hash] / post-[createdAt]-[address]-[id] - reposts of a particular post version
- all-users / user-[username]
- inbox-[address] / post-[createdAt]-[address]-[id] - postJSON, sourceJSON

## Internal DB API
- putItem(Item)
- getItem(PK, SK)
- getItems(PK, fromSK, toSK)
  - implements paging internally 
- delete([PK, SK]) - deletes in batches of 25

## Internal S3 asset store API
- putItem(content, address, createdAt, type, mime-type, username) returns hash

# Public API

- getPosts for a particular timestamp range (createdAt)
  - of several source addresses 
   - combine results of getItems(post-address, fromSK, toSK) 
  - OR with a particular tag
   - combine results of getItems(tag-name, fromSK, toSK)
  - OR replies to a specific post hash
   - combine results of getItems(reply-to-hash, fromSK, toSK)
- getSources by a list of addresses
- getDefaultSources

# Inbox API
- addPost(post, source)

# Private API
- addPost (post, source)
- uploadAsset
- registerUser
- login flow
