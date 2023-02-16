<script>
	import InputField from './InputField.svelte';
	export let active_state;

	export let fullName, quest, favorite_color;

	const handleSubmit = () => {
		console.log('Your form data => ', fullName);
	};
</script>

<form class="form-container" on:submit={handleSubmit}>
	{#if active_state === 'enteringName'}
		<InputField label={'Name'} bind:value={fullName} />
	{:else if active_state === 'enteringQuest'}
		<InputField label={'Quest'} bind:value={quest} />
	{:else if active_state === 'enteringFavoriteColor'}
		<InputField label={'Favorite Color'} bind:value={favorite_color} />
	{:else if active_state === 'youMayPass'}
		<div class="message">
			<h2>You may pass!</h2>
			<button class="btn submit">Finish </button>
		</div>
	{:else if active_state === 'thrownIntoVolcano'}
		<div class="message">
			<h2>You have been thrown into the volcano!</h2>
		</div>
	{/if}
</form>

<style>
	.form-container {
		background-color: #fff;
		border-radius: 10px;
		box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1), 0 6px 6px rgba(0, 0, 0, 0.1);
		padding: 50px 20px;
		text-align: center;
		max-width: 100%;
		width: 350px;
	}
	.btn {
		color: white;
		padding: 0.5rem 0;
		margin-top: 0.5rem;
		display: inline-block;
		width: 100%;
		border-radius: 0.25rem;
		cursor: pointer;
	}
	.submit {
		background: linear-gradient(to bottom, #44c767 5%, #50b01c 100%);
		background-color: #44c767;
	}
	.submit:hover {
		background: linear-gradient(to bottom, #50b01c 5%, #44c767 100%);
		background-color: #50b01c;
	}
	.message {
		text-align: center;
	}
</style>
