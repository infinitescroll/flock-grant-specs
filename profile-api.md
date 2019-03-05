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
const profile = new Profile(ethereumAddress);
```

## Using autark-profile-api in your project

All instance methods support promises and async/await. This documentation uses async/await.

### Determine if a user has a profile or not

This method is a faster way of determining if a user's profile exists (instead of attempting to fetch the profile and getting back `null`)

```js
const hasProfile = await profile.exists();
// > true if the user has a profile
// > false if the user does not have a profile
```

### Get a user's profile

```js
const profile = await profile.get();
// > profile object (example in the next section)
// > null (if no profile exists)
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

```js
{
  "@context": "http://schema.org/Person",
  "@type": "Person",
  "identifier": [
    { // is this better than just putting an ethereum address on the base layer?
      "@context": "https://schema.org/PropertyValue",
      "@type": "PropertyValue",
      "propertyID": "EthereumAddress",
      "value": "<EthereumAddress>"
    },
    { // same Q as above ^^
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
  "affiliation": [ // used for DAO membership
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
