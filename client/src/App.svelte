<script>
	import axios from "axios";
	import Transaction from "./components/Transaction.svelte";
	import SummaryCard from "./components/SummaryCard.svelte";
	import Loading from "./components/Loading.svelte";
	import { onMount } from "svelte";
	import {transactions,sortedTransactions, balance, income, expense} from './store.js'

	let input = 0;
	let typeOfTransaction = "+";
	let loading

	$: disabled = !input;

	onMount(async () => {
			loading = true
		const { data } = await axios.get(
			"/api/transactions"
		);
		$transactions = data;
			loading = false
	});
	const addTransaction = async () => {
			loading = true
		const transaction = {
			date: new Date().getTime(),
			value: typeOfTransaction === "+" ? input : input * -1,
		};
		const response = await axios.post(
			"/api/transactions",
			transaction
		);
		$transactions = [response.data, ...$transactions];
		input = 0;
			loading = false
	};

	const removeTransaction = async (id) => {
			loading = true
		const response = await axios.delete(
			`/api/transactions/${id}`
		);
		if (response.data.id === id) {
			$transactions = $transactions.filter((t) => t._id !== id);
		}
			loading = false
	};
</script>

<style>
	.app {
		margin: 4px auto;
		max-width: 500px;
	}
</style>

<div class="app container">
	<div class="field has-addons">
		<p class="control">
			<span class="select">
				<select bind:value={typeOfTransaction}>
					<option value="+">+</option>
					<option value="-">-</option>
				</select>
			</span>
		</p>
		<p class="control is-expanded">
			<input
				class="input"
				type="number"
				bind:value={input}
				placeholder="Amount of money" />
		</p>
		<p class="control">
			<button class="button" on:click={addTransaction} {disabled}>
				Save
			</button>
		</p>
	</div>
	{#if loading}
		<Loading />
	{/if}
	{#if $transactions.length > 0}
	<SummaryCard mode="balance" value={$balance} />

	<div class="columns">
		<div class="column">
			<SummaryCard mode="income" value={$income} />
		</div>
		<div class="column">
			<SummaryCard mode="expense" value={$expense} />
		</div>
	</div>
	<hr />
	{#each $sortedTransactions as transaction (transaction._id)}
		<Transaction {transaction} {removeTransaction} />
	{/each} 
	{:else if !loading}
		<div class="notification is-info is-light">
			Add Your First Transaction
		</div>
	{/if}
</div>
