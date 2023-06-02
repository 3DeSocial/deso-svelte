<script>

	import {identity,  configure, updateLikeStatus, getLikesForPost, getDiamondsForPost } from 'deso-protocol';
	import { onMount } from 'svelte';	
	import { writable } from 'svelte/store';

	let userStore = writable(null);
	let loggedIn = false;
	let publicKey =null;
	let testPost = 'd4c4b7a26a7fdedb83cdb4902815f9bc82047c87381fa2027820b22604fe09c6';
	let reply = '';
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
		//	UNLIKE: 100,
			FOLLOW: 100
		//	UNFOLLOW: 100			
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
		//{(testPost)?testPost:''
		//{(publicKey)?publicKey:''}
		});
	});

	const userHasLiked = async (myPublicKey, passedPostHash) =>{

		let keyToCheck = publicKey;
		if(myPublicKey){
			keyToCheck = myPublicKey
		}

		let postHashHex = testPost;
		if(passedPostHash){
			postHashHex = passedPostHash;
		};		
		let res = await getLikes(postHashHex);
		let likers = res.Likers;
		console.log(likers);

		let result = likers.some(profile => profile.PublicKeyBase58Check === keyToCheck);
		return result;
	}

	const sendLike=async()=>{

		// Define the postHashHex and readerPublicKeyBase58Check parameters
		let postHashHex = testPost;
		let readerPublicKeyBase58Check = publicKey;
		const params = {
			PostHashHex: testPost
		};

		let liked = await userHasLiked(readerPublicKeyBase58Check, postHashHex);
		let isUnlike=false;
		if(liked){
			console.log('You have liked this post, unliking it now..');
			isUnlike = true
		};

		const likeParams = {
			LikedPostHashHex: testPost,
			ReaderPublicKeyBase58Check:publicKey,
			IsUnlike: isUnlike
		};
		const options = {
		// optional fields
		};

		updateLikeStatus(likeParams, options)
		.then(response => {
			console.log(response);
		})
		.catch(error => {
			console.log('error recieved...')
			console.error(error);
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



	const userHasDiamonded = () =>{

	}

	const sendDiamond=async()=>{
		console.log(`sendDiamond testPost: ${testPost}, publicKey: ${publicKey}`);
		const params = {
			PostHashHex: testPost
		};
		getDiamondsForPost(params)
		.then(diamonds => console.log(diamonds))
		.catch(error => console.error(error));

	}

	const sendReply=async()=>{
		console.log(`testPost: ${testPost}, publicKey: ${publicKey}`);
	}

	const handleLoginClick=async()=>{
		await identity.login();
		
	/*	await identity.requestPermissions({
    TransactionCountLimitMap: {
      SUBMIT_POST: 10,
	  LIKE: 1000,
	  FOLLOW: 1000
    },
  });*/
	
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
	<section>
	<form id="like-form" on:submit|preventDefault={sendLike}>
		<h2>Send Like</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex" bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<button type="submit" id="send-like">Like</button>
	</form>
	<form id="diamond-form" on:submit|preventDefault={sendDiamond}>
		<h2>Send Diamond</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"   bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<button type="submit"  id="send-diamond">Diamond</button>
	</form>	
	<form id="reply-form" on:submit|preventDefault={sendReply}>
		<h2>Send reply</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<label>message</label>
		<input type="text" name="message" bind:value="{reply}"/>		
		<button  type="submit"  id="send-reply">Reply</button>
	</form>
	</section>
	<section>
	<form id="get-likes-form" on:submit|preventDefault={getLikes}>
		<h2>Get Likes For Post</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<button  type="submit"  id="get-likes">Get Likes</button>
	</form>		
	<form id="has-liked-form" on:submit|preventDefault={userHasLiked}>
		<h2>Have I liked This Post?</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<button  type="submit"  id="get-likes">Get Likes</button>
	</form>		
	<form id="get-diamonds-form" on:submit|preventDefault={sendReply}>
		<h2>Get Diamonds For Post</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<button  type="submit"  id="get-diamonds">Get Diamonds</button>
	</form>
	<form id="has-diamonded-form" on:submit|preventDefault={getLikes}>
		<h2>Have I liked This Post?</h2>
		<label>postHashHex</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<button  type="submit"  id="get-likes">Get Likes</button>
	</form>		
	<form id="get-blogs" on:submit|preventDefault={sendReply}>
		<h2>Get Blog Posts For User</h2>
		<label>userPk</label>
		<input type="text" name="postHashHex"  bind:value="{testPost}"/>
		<label>publicKey</label>
		<input type="text" name="publicKey"  bind:value="{publicKey}"/>
		<button  type="submit"  id="get-blogs">Get Blogs</button>
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