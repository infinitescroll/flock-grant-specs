# Profile API Documentation

The profile API will be required by the autark/profiles app and used to get, create, and modify a user's profile

## Setup

### Install autark-profile-api

```
$ npm install autark-profile-api
```

### Import autark-profile-api into your project

```js
import Profile from 'autark-profile-api';
```

## Usage

### Create a new profile instance, passing an ethereum address

```js
const profile = new Profile(ethereumAddress, web3Provider);
```

## Using autark-profile-api in your project

All instance methods support promises and async/await. This documentation uses async/await.

### Get a public profile (does not require opening box first)

_NOTE_ - if the box has been unlocked, this `get` call will also return the private profile variables. If the box has not yet been unlocked, we're going to get the public profile

```js
const publicProfile = await profile.get();
```

### Determine if a user has a profile or not

This method is a faster way of determining if a user's profile exists (instead of attempting to fetch the profile and getting back `null`)

```js
const hasProfile = await profile.exists();
// > true if the user has a profile
// > false if the user does not have a profile
```

### Unlock the profile (REQUIRED before calling any of the methods below)

Invokes a signature request

```js
// initialize async variables
await profile.unlock();
```

### Create a profile for the user

```js
const profileCreated = await profile.create();
// > true if profile successfully created
// > false (and/or error?) if profile is not created
```

### Modifying the entire profile at once (dangerous)

You might use this method once, after initial profile save

```js
const newProfile = await profile.modify(profile);
// > profile object (example in the next section)
```

### Setting specific fields in a profile (safe)

```js
const newProfile = await profile.set(field, value);
```

## A profile object

Initial

```js
{
  "@context": "http://schema.org/",
  "@type": "Person",
  "identifier": [
    { // is this better than just putting an ethereum address on the base layer?
      "@context": "https://schema.org/",
      "@type": "PropertyValue",
      "propertyID": "EthereumAddress",
      "value": "<EthereumAddress>"
    }
  ],
  "familyName": "<LAST NAME>",
  "givenName": "<FIRST NAME>",
  "email": "<EMAIL>",
  "description": "<BIO>",
  "image:" [{
    "@context": "http://schema.org/",
    "@type": "ImageObject",
    "contentUrl": "<FULL URL PATH TO IMAGE>"
  }]
}
```

Extended:

```js
{
  "@context": "http://schema.org/",
  "@type": "Person",
  "identifier": [
    { // is this better than just putting an ethereum address on the base layer?
      "@context": "https://schema.org/",
      "@type": "PropertyValue",
      "propertyID": "EthereumAddress",
      "value": "<EthereumAddress>"
    },
    {
      // we can do each integration as its own property value
      "@context": "https://schema.org/",
      "@type": "PropertyValue",
      "propertyID": "Twitter",
      "value": "<Twitter Handle>",
      "valueReference": "<DID>"
    },
  ],
  "familyName": "<LAST NAME>",
  "givenName": "<FIRST NAME>",
  "email": "<EMAIL>",
  "description": "<BIO>",
  "alumniOf": [ // used for previous schools and jobs
    {
      "@context": "http://schema.org/",
      "@type": "Organization",
      "address": "<Address>",
      "legalName": "<Name of Organization"
    }
  ],
  // SHOULD MEMBEROF AND AFFILIATION BE THE SAME THING?
  "memberOf": [ // used for current education and jobs
    {
      "@context": "http://schema.org/",
      "@type": "Organization",
      "address": "<Address>",
      "legalName": "<Name of Organization"
    }
  ],
  "affiliation": [ // used for DAO memberships && Github organizations
    {
      "@context": "http://schema.org/",
      "@type": "Organization",
      "legalName": "<Name of Organization",
      "logo": {
        "@context": "http://schema.org/",
        "@type": "ImageObject",
        "contentUrl": "<FULL URL PATH TO IMAGE>"
      }
    }
  ],
  "image:" [{
    "@context": "http://schema.org/",
    "@type": "ImageObject",
    "contentUrl": "<FULL URL PATH TO IMAGE>"
  }],
  "jobTitle": "<TITLE>", // used for current job title (might be redundant for memberOf)
}
```

1. format this data

```js
const artist = {
  '@context': 'schema.org/Person',
  '@type': 'Person',
  givenName: 'Jon',
};
```

2. put this dag to IPFS (on the browser) and get back cid
3. with the hash, we add back in the cid

```js
const artist = {
  '@context': 'schema.org/Person',
  '@type': 'Person',
  givenName: 'Jon',
  ipld: '<hash>',
};
```

4. we put this into 3box

const discussionThread = {
"@id": 'localhost:3000/discussions/<cid>'
posts: [
{
"@id": 'localhost:3000/discussions/<cid>'
},
{
"@id": 'localhost:3000/discussions/<cid>'
}
]
}

const discussionPost = {
"@id": 'localhost:3000/discussions/<cid>'
}

```js
class ethereumAdapter {
  async sendAsync(args, callback) {
    if (args.method !== 'personal_sign') throw new Error('unsupported method');

    try {
      const sig = await Aragon.sign(args.ethereumAddress, args.dataToSign);
      return sig.data;
    } catch (err) {
      throw new Error(err);
    }
  }
}
```
