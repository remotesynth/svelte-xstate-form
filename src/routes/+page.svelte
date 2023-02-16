<script>
	import Form from './Form.svelte';
	import ProgressBar from './ProgressBar.svelte';
	import { createMachine, interpret, assign } from 'xstate';

	let fullName, quest, favorite_color;
	let formData = {};
	const formMachine = createMachine({
		id: 'bridgekeeper-multi-step-form',
		context: {
			fullName: '',
			quest: '',
			favorite_color: ''
		},
		predictableActionArguments: true,
		initial: 'answering questions',
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
								target: 'validateQuest',
								actions: [
									assign({
										quest: (context, event) => {
											if (quest) return quest;
										}
									})
								]
							}
						}
					},
					validateQuest: {
						on: {
							'': [
								{
									target: 'enteringFavoriteColor',
									cond: (context) => context.quest !== undefined && context.quest.length > 0
								},
								{
									target: '#bridgekeeper-multi-step-form.thrownIntoVolcano'
								}
							]
						}
					},
					enteringFavoriteColor: {
						on: {
							CONFIRM_FAVORITE_COLOR: {
								target: 'validateFavoriteColor',
								actions: [
									assign({
										favorite_color: (context, event) => {
											if (favorite_color) return favorite_color;
										}
									})
								]
							}
						}
					},
					validateFavoriteColor: {
						on: {
							'': [
								{
									target: 'allAnswered',
									cond: (context) =>
										context.favorite_color !== undefined && context.favorite_color.length > 0
								},
								{
									target: '#bridgekeeper-multi-step-form.thrownIntoVolcano'
								}
							]
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
							'': [
								{
									target: 'enteringQuest',
									cond: (context) => context.fullName !== undefined && context.fullName.length > 0
								},
								{
									target: '#bridgekeeper-multi-step-form.thrownIntoVolcano'
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
			console.log('answering questions:' + active_state);
		} else {
			console.log(state.value);
		}
	});
	active_state = service.initialState;
	service.start();

	let steps = [
			{ title: 'Name', event: 'CONFIRM_NAME' },
			{ title: 'Quest', event: 'CONFIRM_QUEST' },
			{ title: 'Color', event: 'CONFIRM_FAVORITE_COLOR' },
			{ title: 'Confirmation', event: '' }
		],
		currentActive = 0;

	const handleProgress = (stepIncrement) => {
		service.send(steps[currentActive].event);
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
