<script>
	import {onMount} from 'svelte';
	import lottery from './lottery';
	import web3 from './web3';

	let accounts = [];
	let manager = "";
	let players = [];
	let balance = 0;
	let onenterpromise = "";
	let inputValue = 0;
	onMount(async()=>{
		accounts = await web3.eth.getAccounts();
		manager = await lottery.methods.manager().call({
		from: accounts[0],
		});
		players = await getPlayers()
		balance = await getBalance();
		balance = web3.utils.fromWei(balance,"ether");
	});

	const getPlayers = async() => {
		const retVal = await lottery.methods.getPlayers().call();
		return retVal;
	}
	const getBalance = async() => {
		const retVal = await web3.eth.getBalance(lottery.options.address);
		return retVal;
	}

	const pickAWinner = async()=>{
		const retVal = await lottery.methods.pickaWinner().send({
			from:accounts[0]
		})
		console.log(retVal);
		players = await getPlayers()
		balance = await getBalance();
		balance = web3.utils.fromWei(balance,"ether");
	}

	const onEnterLottery = async ()  => {
		onenterpromise = onEnterContract(inputValue);
	};

	const onEnterContract = async (ether) => {
		const ret = await lottery.methods.enter().send({
			from:accounts[0],
			value: web3.utils.toWei(ether,"ether")
		});
		console.log(ret);
		if(ret.status)
		{
			players = await getPlayers()
			balance = await getBalance();
			balance = web3.utils.fromWei(balance,"ether");
			return "Transaction Successfull!"
		}
		else{
			return "Transaction Failed!";
		}
	}

</script>

<main>
	<h1>LOTTERY</h1>

	<div class="form">
		<div class="form-container">
			<p>This contract is manage by {manager}.</p>
			<p>There are currently {players.length} people entered, competing to win {balance} ether.</p>
			<button on:click="{pickAWinner}">Pick A Winner</button>
		</div>
		<hr>
		<div class="form-container">	
			<h2>Want to try your luck?</h2>
			<div>
				<form class="input-container" on:submit|preventDefault="{onEnterLottery}">
					<label for="ether-value">Amount of ether to enter</label>
					<input id="ether-value" name="ether_value" type="text" bind:value="{inputValue}" required>
					<button type="submit">Enter</button>
				</form>
			</div>
		</div>
		<hr>

		<div class="form-container">
			{#await onenterpromise}
				<p>Please wait while Processing...</p>
			{:then value} 
				{value}
			{/await}
		</div>
	</div>
</main>

<style>
	h1{
		margin: 0 auto;
		text-align: center;
		padding-bottom: 10px;
	}
	.form{
		border: 1px solid black;
		border-radius: 15px;
		height: 450px;
	}

	.form-container{
		display: flex;
		flex-direction: column;
		row-gap: 15px;
		align-items: center;
		justify-content: center;
		align-content: center;
		text-align: center;
		padding: 10px;
	}

	.input-container{
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	input{
		text-align: center;
		padding: 5px 10px;
		border-radius: 5px;
	}
	p{
		padding: 0;
		margin: 0;
	}

	button{
		padding: 5px 15px;
		background-color:deepskyblue;
		border-radius: 5px;
		font-size: 20px;
	}
</style>