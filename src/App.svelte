<script>

	import {identity,  configure, updateLikeStatus, getLikesForPost, getDiamondsForPost, getSinglePost, sendDiamonds, submitPost} from 'deso-protocol';
	import { onMount } from 'svelte';	
	import { writable } from 'svelte/store';

	let userStore = writable(null);
	let loggedIn = false;
	let publicKey =null;
	let testPost = 'aaebf5cdc74423356f53ce865ed88b0ee41bba8c9d496e86b9e32c3135fa140a';
	let reply = '';
	let responseMessage = null;
	let testPostBody = '';
	onMount ((count) => {
		configure({
			spendingLimitOptions: {
				// NOTE: this value is in Deso nanos, so 1 Deso * 1e9
				GlobalDESOLimit: 1 * 1e9, // 1 Deso
				// Map of transaction type to the number of times this derived key is
				// allowed to perform this operation on behalf of the owner public key
				TransactionCountLimitMap: {
				BASIC_TRANSFER: 100, // 2 basic transfer transactions are authorized
				SUBMIT_POST: 10,
				LIKE: 100,
				FOLLOW: 100
				}
			}
		})		
		identity.subscribe((state) => {
		const event = state.event;
		switch(event){
			case 'LOGOUT_END':
				userStore.set({});
				loggedIn = false;

			break;
			case 'LOGIN_END':
				if(state.currentUser){
					let currentUser = state.currentUser;
					userStore.set(currentUser);
					loggedIn = true;
					publicKey =currentUser.publicKey;

				
				}				
			break;			

		}

		});
	});

	const sendPostHandler = async () =>{
		// get param values
		let senderPublicKeyBase58Check = publicKey;	
		let postBody = testPostBody;

		const params = {
			UpdaterPublicKeyBase58Check: senderPublicKeyBase58Check,
			BodyObj: {
				Body: postBody,
				ImageURLs: [],
				VideoURLs: [],
			}
		};
		console.log(params);
		const txInfo = await submitPost(params);		
		console.log(txInfo);
	}

	const sendLikeHandler = async () =>{

		// get param values
		let postHashHex = testPost;
		let readerPublicKeyBase58Check = publicKey;


		sendLike(readerPublicKeyBase58Check, testPost, false)

	}

	const userHasLikedHandler = async () =>{
		let myPublicKey = publicKey;
		let postHashHex = testPost;

		let likedStatus = await userHasLiked(myPublicKey,postHashHex);
		responseMessage = 'Your key '+myPublicKey+' has liked post '+postHashHex;
	}

	const sendDiamondHandler = async () =>{
		// get param values
		let postHashHex = testPost;
		let readerPublicKeyBase58Check = publicKey;
		sendDiamond(readerPublicKeyBase58Check, postHashHex);
	}

	const getLikesHandler = async () =>{

	}

	const getDiamondsForPostHandler = async () =>{
		let myPublicKey = publicKey;
		let postHashHex = testPost;
		let dimondStaus = await userHasDiamonded(myPublicKey, postHashHex);
		if(dimondStaus){
			console.log('You have sent diamonds to this post');
			console.log(dimondStaus);
			responseMessage = 'Your key '+myPublicKey+' has NOT sent diamonds to this post '+postHashHex;

		} else {
			console.log('You have not sent diamonds to this post');
			console.log(dimondStaus);
		responseMessage = 'Your key '+myPublicKey+' has disent diamonds to this post '+postHashHex;
		}
	}

	const sendReplyHandler = async () =>{
		let myPublicKey = publicKey;
		let postHashHex = testPost;
		let postBody = testPostBody;

		const params = {
			ParentStakeID: postHashHex,
			UpdaterPublicKeyBase58Check: myPublicKey,
			BodyObj: {
				Body: postBody,
				ImageURLs: [],
				VideoURLs: [],
			}
		};
		console.log(params);
		const txInfo = await submitPost(params);		
		console.log(txInfo);
	}
	const userHasLiked = async (myPublicKey, passedPostHash) =>{

		let res = await getLikes(passedPostHash);
		let likers = res.Likers;
		let result = likers.some(profile => profile.PublicKeyBase58Check === myPublicKey);
		return result;
	}

	const sendLike=async(readerPublicKeyBase58Check, postHashHex, unlikeIfLiked)=>{

		let liked = await userHasLiked(readerPublicKeyBase58Check, postHashHex);
		let isUnlike=false;
		if(liked){
			if(unlikeIfLiked){
				isUnlike = true
			}
		};

		const likeParams = {
			LikedPostHashHex: postHashHex,
			ReaderPublicKeyBase58Check:readerPublicKeyBase58Check,
			IsUnlike: isUnlike
		};
		const options = {
		// optional fields
		};

		updateLikeStatus(likeParams, options)
		.then(response => {
			console.log(response);
			responseMessage = JSON.stringify(response);
		})
		.catch(error => {
			console.log('error recieved...')
			console.error(error);
			responseMessage = JSON.stringify(error);

		});

	}
	const getLikes = (passedPostHash)=>{
		let readerPublicKeyBase58Check = publicKey;
		let postHashHex = testPost;
		if(passedPostHash){
			postHashHex = passedPostHash;
		};
		const params = {
			PostHashHex: postHashHex,
			Limit: 100000,
			Offset: 0
		};

		// Call the getLikesForPost function
		return getLikesForPost(params)
	
	}




	const sendDiamond=async(readerPublicKeyBase58Check, passedPostHash)=>{
		let dimondStaus = await userHasDiamonded(readerPublicKeyBase58Check, passedPostHash);
		if(dimondStaus){
			console.log('You have sent diamonds to this post');
			console.log(dimondStaus);
		} else {
			console.log('You have not sent diamonds to this post');
			console.log(dimondStaus);		
			const receiverPublicKey = await getPostAuthor(passedPostHash);
			const senderPublicKey = readerPublicKeyBase58Check;
			const diamondLevel = 1; // Diamond level can be from 1 to 6

			const params = {
				SenderPublicKeyBase58Check: senderPublicKey,
				ReceiverPublicKeyBase58Check: receiverPublicKey,
				DiamondLevel: diamondLevel,
				DiamondPostHashHex: passedPostHash
			};

			console.log('params',params);
			
			sendDiamonds(params).then((response) => {
				console.log(response);
				responseMessage = JSON.stringify(response);

			}).catch(error => {
				console.error(error);
				responseMessage = JSON.stringify(error);

			});
		}
	}

	const getPostAuthor = async(passedPostHashHex) =>{
		const params = {PostHashHex:passedPostHashHex};
		let res = await getSinglePost(params);
		return res.PostFound.ProfileEntryResponse.PublicKeyBase58Check;
	}

	const getDiamonds  = async (passedPostHash) =>{

		const params = {
			PostHashHex: passedPostHash,
			Limit: 100000,
			Offset: 0
		};

		// Call the getLikesForPost function
		return getDiamondsForPost(params)
	}

	const userHasDiamonded = async (myPublicKey, passedPostHash) =>{
		let res = await getDiamonds(passedPostHash);
		let diamondSenders = res.DiamondSenders;
		let result = diamondSenders.some(profile => profile.PublicKeyBase58Check === myPublicKey);
		return result;
	}

	const sendReply = async()=>{

	}

	const sendReplyOnce = async()=>{

	}	

	const handleLoginClick=async()=>{
		await identity.login();
	}

	const handleLogoutClick=async()=>{
		await identity.logout();
	}
</script>
<main>
	{#if !loggedIn}	
		<h2>Login</h2>
		<div>
		<button id="login" on:click={handleLoginClick}>Login With DeSo</button>	
		</div>
	{:else}	
	<div>
		<h2>Logout</h2>
		<button id="login" on:click={handleLogoutClick}>Log out</button>
		</div>
	{/if}
	{#if responseMessage!=null}	
		<section>
			<div>{responseMessage}</div>
		</section>
	{/if}	
	<section>
	<form id="like-form" on:submit|preventDefault={sendLikeHandler}>
		<h2>Send Like</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex" bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<button type="submit" id="send-like">Like</button>
	</form>
	<form id="diamond-form" on:submit|preventDefault={sendDiamondHandler}>
		<h2>Send Diamond</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"   bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<button type="submit"  id="send-diamond">Diamond</button>
	</form>	
	</section>
	<section>
	<form id="get-likes-form" on:submit|preventDefault={getLikesHandler}>
		<h2>Get Likes For Post</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<button  type="submit"  id="get-likes">Get Likes</button>
	</form>		
	<form id="has-liked-form" on:submit|preventDefault={userHasLikedHandler}>
		<h2>Have I liked This Post?</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<button  type="submit"  id="get-likes">Get Likes</button>
	</form>		
	<form id="get-diamonds-form" on:submit|preventDefault={getDiamondsForPostHandler}>
		<h2>Get Diamonds For Post</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<button  type="submit"  id="get-diamonds">Get Diamonds</button>
	</form>
	<form id="has-diamonded-form" on:submit|preventDefault={getLikesHandler}>
		<h2>Have I liked This Post?</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<button  type="submit"  id="get-likes">Get Likes</button>
	</form>		
	<form id="get-blogs" on:submit|preventDefault={sendReplyHandler}>
		<h2>Get Blog Posts For User</h2>
		<label>userPk</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<button  type="submit"  id="get-blogs">Get Blogs</button>
	</form>
	</section>
	<section>
		<form id="post-form" on:submit|preventDefault={sendPostHandler}>
			<h2>Send Post</h2>
			<label>postHashHex</label>
			<input type="text" name="postHashHex"  bind:value="{testPost}"/>
			<label>publicKey</label>
			<input type="text" name="publicKey"  bind:value="{publicKey}"/>
			<label>message</label>
			<input type="text" name="message" bind:value="{testPostBody}"/>		
			<button  type="submit"  id="send-post">Post</button>
		</form>		
		<form id="reply-form" on:submit|preventDefault={sendReplyHandler}>
			<h2>Send reply</h2>
			<label>Reply To PostHashHex</label>
			<input type="text" name="postHashHex"  bind:value="{testPost}"/>
			<label>publicKey</label>
			<input type="text" name="publicKey"  bind:value="{publicKey}"/>
			<label>message</label>
			<input type="text" name="message" bind:value="{testPostBody}"/>		
			<button  type="submit"  id="send-reply">Reply</button>
		</form>
	</section>
</main>

<style>
	main {
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}
	form {
		border: solid 1px grey;
		max-width: 20em;
		padding: 1em;
		margin-right: 1em;
		float: left;
	}

	section{ clear: left;}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>