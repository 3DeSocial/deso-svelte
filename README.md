## Functions in App.svelte

### `configure`
This function is used to set the spending limit options and transaction count limit map for various operations like basic transfer, submit post, like, and follow.

**Parameters:**
- An object with `GlobalDESOLimit` and `TransactionCountLimitMap` as properties.

```javascript
configure({
  spendingLimitOptions: {
    GlobalDESOLimit: 1 * 1e9, // 1 Deso
    TransactionCountLimitMap: {
      BASIC_TRANSFER: 100,
      SUBMIT_POST: 10,
      LIKE: 100,
      FOLLOW: 100
    }
  }
});
```

### `identity.subscribe`
This function is used to subscribe to the identity object and handle login and logout events.

**Parameters:**
- A state object. Uses the event property of the state to handle different cases.

```javascript
identity.subscribe((state) => {
  const event = state.event;
  // handle different cases based on event
});
```

### `userHasLiked`
This function is used to check if the user has liked a particular post.

**Parameters:**
- User's public key
- Post hash

```javascript
let result = userHasLiked(myPublicKey, postHashHex);
```

### `sendLike`
This function is used to send a like to a post.

**Parameters:**
- Reader's public key
- Post hash
- Boolean flag to unlike if already liked

```javascript
sendLike(readerPublicKeyBase58Check, postHashHex, unlikeIfLiked);
```

### `getLikes`
This function is used to get the likes for a post.

**Parameters:**
- Post hash

```javascript
let res = getLikes(postHashHex);
```

### `sendDiamond`
This function is used to send a diamond to a post.

**Parameters:**
- Reader's public key
- Post hash

```javascript
sendDiamond(readerPublicKeyBase58Check, postHashHex);
```

### `sendPost`
This function is used to send a post.

**Parameters:**
- Sender's public key
- Post body

```javascript
sendPost(senderPublicKeyBase58Check, postBody);
```

The functions `getLikesHandler`, `getDiamondsForPostHandler`, and `sendReplyHandler` are currently empty and do not perform any operations.
