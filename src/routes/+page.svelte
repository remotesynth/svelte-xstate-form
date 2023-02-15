<script>
	import Form from './Form.svelte';
	import ProgressBar from './ProgressBar.svelte';
	import { createMachine, interpret, assign } from 'xstate';

	let fullName, quest, favorite_color;
	let formData = {};
	const formMachine = createMachine({
		id: 'Bridgekeeper Multi-step Form',
		predictableActionArguments: true,
		initial: 'answering questions',
		context: {
			fullName: '',
			quest: '',
			color: ''
		},
		states: {
			thrownIntoVolcano: {
				type: 'final'
			},
			'answering questions': {
				initial: 'enteringName',
				states: {
					enteringQuest: {
						on: {
							CONFIRM_QUEST: {
								target: 'enteringFavoriteColor',
								actions: [
									assign({
										name: (context, event) => {
											if (quest) return quest;
										}
									})
								],
								// you cannot get to this without entering a quest
								cond: (context) => context.quest.length > 0
							}
						}
					},
					enteringFavoriteColor: {
						on: {
							CONFIRM_FAVORITE_COLOR: {
								target: 'allAnswered',
								actions: [
									assign({
										name: (context, event) => {
											if (favorite_color) return favorite_color;
										}
									})
								],
								// you cannot get to this without entering a color
								cond: (context) => context.favorite_color.length > 0
							}
						}
					},
					enteringName: {
						on: {
							CONFIRM_NAME: {
								target: 'validateName',
								actions: [
									assign({
										fullName: (context, event) => {
											if (fullName) return fullName;
										}
									})
								]
							}
						}
					},
					validateName: {
						on: {
							always: [
								{
									target: 'enteringQuest',
									cond: (context) => context.fullName.length > 0
								}
							]
						}
					},
					allAnswered: {
						type: 'final'
					}
				},
				on: {
					'*': {
						target: 'thrownIntoVolcano',
						description: 'Any unexpected event (wrong answers)'
					}
				},
				onDone: {
					target: 'youMayPass'
				}
			},
			youMayPass: {
				type: 'final'
			}
		}
	});

	let progressBar, active_state;
	const service = interpret(formMachine).onTransition((state) => {
		if (state.matches('answering questions')) {
			active_state = state.value['answering questions'];
			if (progressBar) progressBar.handleProgress(1);
			console.log(active_state);
		} else {
			console.log(state.value);
		}
	});
	active_state = service.initialState;
	service.start();

	let steps = [
			{ title: 'Name', event: 'CONFIRM_NAME' },
			{ title: 'Quest', event: 'CONFIRM_QUEST' },
			{ title: 'Color', event: 'CONFIRM_COLOR' },
			{ title: 'Confirmation', event: '' }
		],
		currentActive = 1;

	const handleProgress = (stepIncrement) => {
		service.send(steps[currentActive - 1].event);
	};
</script>

<ProgressBar {steps} bind:currentActive bind:this={progressBar} />

<Form {active_state} bind:fullName bind:quest bind:favorite_color />

<div class="step-button">
	<button class="btn" on:click={() => handleProgress(-1)} disabled={currentActive == 1}>Prev</button
	>
	<button class="btn" on:click={() => handleProgress(+1)} disabled={currentActive == steps.length}
		>Next</button
	>
</div>

<style>
	.btn {
		background-color: #3498db;
		color: #fff;
		border: 0;
		border-radius: 6px;
		cursor: pointer;
		font-family: inherit;
		padding: 8px 30px;
		margin: 5px;
		font-size: 14px;
	}

	.btn:active {
		transform: scale(0.98);
	}

	.btn:focus {
		outline: 0;
	}

	.btn:disabled {
		background-color: #e0e0e0;
		cursor: not-allowed;
	}

	.step-button {
		margin-top: 1rem;
		text-align: center;
	}
</style>
