# Discussion API Documentation

The discussion API will be a node-express-SQL database that stores information on IPFS and resolves data in a similar way to IPLD

## Naming

A "discussion" is composed of 2 parts:

1. an ethereum transaction hash that represents the vote
2. a series of posts about the ethereum transaction hash

We've decided to format (1) as a schema.org [Conversation](https://schema.org/Conversation) and (2) as schema.org [Comment](https://schema.org/Comment)

## Usage

### Authentication

The following steps are required in order to POST discussion data:

1. ask the user to sign {message - TBD}
2. capture the signature
3. send signature, {message}, and ethereum address used to sign to `/auth` route (see section on Routes for more info)
4. backend will ECRecover the signature, and ensure the user owns the signed ethereum address
5. backend will generate a JWT for the user and send it back to the client
6. User can attach the JWT token to as a header to POST requests under the Authorization header: `{ Authorization: Bearer <token> }`
7. Auth token will expire after {time - TBD}

ANOTHER OPTION (need research to determine if this is as secure):

1. ask the user to sign {message - TBD}
2. User signs message
3. Cache signature, message, and address used to sign in aragon cache
4. When client sends `POST` request to server, send the signature, message, and address used to sign in the payload or headers (tbd)
5. Server valides signature
6. Server completes POST request

If this is just as secure ^^ then we remove the steps of using JWT auth

### Routes

##### Get Conversation about an ethereum transaction

`GET` `/api/v0/conversation/<txHash>`

This will return a conversation with all its comments:

```js
{
  "@id": '<endpoint>/api/v0/conversation/<txHash>',
  "@context": 'http://schema.org/',
  "@type": 'Conversation',
  about: <Thing>,
  author: {
    "@context": "http://schema.org/",
    "@type": "Person",
    "identifier": [
      {
        "@context": "https://schema.org/",
        "@type": "PropertyValue",
        "propertyID": "EthereumAddress",
        "value": "<EthereumAddress>"
      }
    ]
  },
  commentCount: <Integer>,
  comment: [
    {
      "@id": '<endpoint>/api/v0/conversation/<txHash>/comment/<cid>',
      "@context": 'http://schema.org/',
      "@type": 'Comment',
      parentItem: '<endpoint>/api/v0/conversation/<txHash>',
      author: {
        "@context": "http://schema.org/",
        "@type": "Person",
        "identifier": [
          {
            "@context": "https://schema.org/",
            "@type": "PropertyValue",
            "propertyID": "EthereumAddress",
            "value": "<EthereumAddress>"
          }
        ]
      },
      dateModified: <Date>,
      datePublished: <Date>,
      mentions: [
        {
          "@context": "http://schema.org/",
          "@type": "Person",
          "identifier": [
            {
              "@context": "https://schema.org/",
              "@type": "PropertyValue",
              "propertyID": "EthereumAddress",
              "value": "<EthereumAddress>"
            }
          ]
        },
        {
          "@context": "http://schema.org/",
          "@type": "Person",
          "identifier": [
            {
              "@context": "https://schema.org/",
              "@type": "PropertyValue",
              "propertyID": "EthereumAddress",
              "value": "<EthereumAddress>"
            }
          ]
        },
      ],
      text: <String>
    },
    { ... },
    { ... }
  ],
  expires: <Date>,
  inLanguage: 'EN',
}
```

Q: Do we want to include pagination query parameters at the start or later on?

##### Get Comment from a Conversation

`GET` `/api/v0/conversation/<txHash>/comment/cid`

returns a comment:

```js
{
  "@id": '<endpoint>/api/v0/conversation/<txHash>/comment/<cid>',
  "@context": 'http://schema.org/',
  "@type": 'Comment',
  parentItem: '<endpoint>/api/v0/conversation/<txHash>',
  author: <Person>,
  dateModified: <Date>,
  datePublished: <Date>,
  mentions: [<Person>, <Person>],
  text: <String>
},
```

##### POST Comment to a Conversation

Following the auth steps at the beginning of this document, an Authorization header or signature will need to be included in every post request to validate signatures

```js
{
  "@id": '<endpoint>/api/v0/conversation/<txHash>/comment/<cid>',
  "@context": 'http://schema.org/',
  "@type": 'Comment',
  parentItem: '<endpoint>/api/v0/conversation/<txHash>',
  author: <Person>,
  mentions: [<Person>, <Person>], // keep this out for v1
  text: <String>
},
```

Later on to allow for reddit style deeply nested replies, `parentItem` can be another `Comment`. For now we're keeping `parentItem` as a Conversation and allowing only 1 level of nesting
