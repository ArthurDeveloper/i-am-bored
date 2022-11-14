<script>
	export let activities = [
		{
			name: 'Fun!',
			apiEquivalent: 'recreational',
			color: '#03a1fc',
		},
		{
			name: 'Learning!',
			apiEquivalent: 'education',
			color: '#f5e90f',
		},
		{
			name: 'Socialization!',
			apiEquivalent: 'social',
			color: '#ffc400',
		},
		{
			name: 'Something to do by myself!',
			apiEquivalent: 'diy',
			color: '#0af529',
		},
		{
			name: 'People to help!',
			apiEquivalent: 'charity',
			color: '#d107fa',
		},
		{
			name: 'A good time in kitchen!',
			apiEquivalent: 'cooking',
			color: '#fc003f',
		},
		{
			name: 'Relaxation!',
			apiEquivalent: 'relaxation',
			color: '#00d6fc',
		},
		{
			name: 'A good song listening!',
			apiEquivalent: 'music',
			color: '#0061bd',
		},
		{
			name: 'Something to be busy about!',
			apiEquivalent: 'busywork',
			color: '#bd5500',
		}
	];

	export let suggestedActivity;

	export let requestsPerSec = 0;
	export let maxRequestsPerSec = 3;
	export let exceededMaxRequestsPerSec = false;

	export let timer;
	export function startTiming() {
		clearTimeout(timer);
		timer = setTimeout(() => {
			requestsPerSec = 0;
			if (exceededMaxRequestsPerSec) exceededMaxRequestsPerSec = false;
		}, exceededMaxRequestsPerSec ? 3000 : 1000);
	}

	export async function requestActivityData(type) {
		requestsPerSec += 1;

		const req = await fetch(`http://www.boredapi.com/api/activity?type=${type}`);
		const data = await req.json();
		
		return data;
	}
</script>

<main>
	<h1>I am bored! 😒</h1>

	<h3>I want to have...</h3>
	<div class="activity-buttons-container">
		{#each activities as activity}
			<button
				class="activity-button"
				style={`background-color: ${activity.color}`}
				on:click={() => {
					(async () => {
						if (requestsPerSec > maxRequestsPerSec) {
							suggestedActivity = 'Hey! Don\'t do that many requests!';
							exceededMaxRequestsPerSec = true;
							return;
						}
						suggestedActivity = 'Loading...';
						const data = await requestActivityData(activity.apiEquivalent);
						
						data.activity = data.activity[0].toLowerCase() + data.activity.slice(1);
						suggestedActivity = `You should ${data.activity}`;

						localStorage.setItem('last-suggestion', suggestedActivity);


						startTiming();
					})();
				}}
			>{activity.name}</button>
		{/each}
	</div>

	<h4 class="suggested-activity">
		{suggestedActivity ?? 
		(
			(
				typeof window !== 'undefined' ? 
					localStorage.getItem('last-suggestion') 
					: ''
			)
		)}
	</h4>
</main>

<style lang="scss">
	* {
		margin: 0;
		padding: 0;
		font-family: Arial, sans-serif;
	}


	.activity-buttons-container {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 5rem;
		width: 100%;
	}

	.activity-button {
		color: #fff;
		border: none;
		width: 30rem;
		height: 7rem;
		font-size: 2rem;
		border-radius: 1rem;
		transition: transform 200ms ease-out;
	}

	.activity-button:hover {
		cursor: pointer;
		transform: scale(1.2);
	}

	.suggested-activity {
		text-align: center;
		font-size: 2rem;
		margin-top: 5rem;
	}
</style>
