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

This call does not require any signatures

```js
const publicProfile = await profile.getPublic();
```

### Determine if a user has a profile or not

This method is a faster way of determining if a user's profile exists (instead of attempting to fetch the profile and getting back `null`)

```js
const hasProfile = await profile.exists();
// > true if the user has a profile
// > false if the user does not have a profile
```

### Unlock or create a profile (REQUIRED before calling any of the methods below)

Invokes a signature request

```js
// initialize async variables
await profile.unlockOrCreate();
```

### Get private variables

```js
const privateVars = await profile.getPrivate();
```

(we could even add params so that this call only returns private variables you're looking for)

### Setting public fields

```js
const newProfile = await profile.setPublicFields(
  [field1, field2],
  [value1, value2]
);
```

Note, the ordering and length of arrays matter - if you pass in arrays of different lengths, error will get thrown.

This will set field1 to value1 and field2 to value2

### Setting private fields

```js
const newProfile = await profile.setPrivateFields(
  [field1, field2],
  [value1, value2]
);
```

Note, the ordering and length of arrays matter - if you pass in arrays of different lengths, error will get thrown.

This will set field1 to value1 and field2 to value2

### Finding out of a user is already logged in: TBD

This method seems like it will be handy for a nice UX https://github.com/3box/3box-js#Box.isLoggedIn

---

AFter talking with 3box team about data standards, here is our gameplan:

#### STORAGE:

- We’re going to be storing Aragon profile data in the public 3box space
- we’re going to add @context fields for necessary objects
- we’re going to store images in the way 3box currently stores images (+ @context field)
- we’re going to add the ethereum address to an identifier array based off the signature stored in 3box

#### METHODS:

- we need to write a method that takes an image stored in current 3box format and outputs JSON-LD. Eventually this method should be expanded to understand if the image field is formatted according to the current format or the JSON-LD schema.org format and act accordingly
- we need to write a method that takes an ethereum proof from the user’s 3box and reformats it into its schema.org representation

## A profile object

fields we support:

ethereum address + signature
name
email
description
job title
employer
website
location
image
school(s)

```js
{
  "@context": "http://schema.org/",
  "@type": "Person",
  "identifier": [
    { // is this better than just putting an ethereum address on the base layer?
      "@context": "https://schema.org/",
      "@type": "PropertyValue",
      "propertyID": "EthereumAddress",
      "value": "<EthereumAddress>",
      "signature": ""
    }
  ],
  "name": "<LAST NAME>",
  "email": "<EMAIL>",
  "description": "<BIO>",
  "jobTitle": "<JOB TITLE>",
  "worksFor": {
    "@type": "Organization",
    "@context": "http://schema.org/",
    "name": "<NAME OF EMPLOYER>"
  },
  "affiliation": [{
    "@type": "Organization",
    "@context": "http://schema.org/",
    "name": "<NAME OF SCHOOL>"
  }],
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@context": "http://schema.org/",
    "@id": "http://cathscafe.example.com/"
  },
  "homeLocation": "<LOCATION>",
  "image": {
    "@context": "http://schema.org/",
    "@type": "ImageObject",
    "contentUrl": "hash"
  }
}
```

```
{
  "@type": "ImageObject",
  "contentUrl": "<FULL URL PATH TO IMAGE>"
}
```

Extended:

```js
{
  "@context": "http://schema.org/",
  "@type": "Person",
  "identifier": [
    { // is this better than just putting an ethereum address on the base layer?
      "@context": "https://schema.org/PropertyValue",
      "@type": "PropertyValue",
      "propertyID": "EthereumAddress",
      "value": "<EthereumAddress>"
    },
    {
      // we can do each integration as its own property value
      "@context": "https://schema.org/PropertyValue",
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
  /* NOT SURE ABOUT ALUMNI OF VS MEMBER OF VS AFFILIATION (think we just go with affiliation) */
  "alumniOf": [ // used for previous schools and jobs
    {
      "@context": "http://schema.org/Organization",
      "@type": "Organization",
      "address": "<Address>",
      "legalName": "<Name of Organization"
    }
  ],
  // SHOULD MEMBEROF AND AFFILIATION BE THE SAME THING?
  "memberOf": [ // used for current education and jobs
    {
      "@context": "http://schema.org/Organization",
      "@type": "Organization",
      "address": "<Address>",
      "legalName": "<Name of Organization"
    }
  ],
  "affiliation": [ // used for DAO memberships && Github organizations
    {
      "@context": "http://schema.org/Organization",
      "@type": "Organization",
      "legalName": "<Name of Organization",
      "logo": {
        "@context": "http://schema.org/ImageObject",
        "@type": "ImageObject",
        "contentUrl": "<FULL URL PATH TO IMAGE>"
      }
    }
  ],
  "image:" [{
    "@context": "http://schema.org/ImageObject",
    "@type": "ImageObject",
    "contentUrl": "<FULL URL PATH TO IMAGE>"
  }],
  "jobTitle": "<TITLE>", // used for current job title (might be redundant for memberOf)
}
```

How to bridge aragon/3box (or something like this)

```js
class BoxAragonBridge {
  constructor(ethereumAddress, requestSignMessage) {
    this.ethereumAddress = ethereumAddress;
    this.requestSignMessage = requestSignMessage;
  }

  getMethod = method => {
    const methods = {
      personal_sign: async ([message], callback) => {
        this.requestSignMessage(message).subscribe(
          signature => callback(null, { result: signature, error: null }),
          error => callback(error, { error })
        );
      },
    };

    return methods[method];
  };

  sendAsync = async ({ fromAddress, method, params, jsonrpc }, callback) => {
    if (!supportedMethod(method, jsonrpc)) {
      throw new Error('Unsupported sendAsync json rpc method or version');
    }

    if (fromAddress.toLowerCase() !== this.ethereumAddress.toLowerCase()) {
      throw new Error('Address mismatch');
    }

    const handler = this.getMethod(method);
    handler(params, callback);
  };
}
```
