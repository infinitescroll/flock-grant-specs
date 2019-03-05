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
// returns a profile object (example in the next section)
```

### Create a profile for the user

```js
const profileCreated = await profile.create();
// > true if profile successfully created
// > false (and/or error?) if profile is not created
```

### Setting profile information

```js
const newProfile = await profile.modify(profile);
// returns the new profile
```
