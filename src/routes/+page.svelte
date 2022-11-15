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
	export let suggestionElement;

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
	<header>
		<h1 class="heading">I am bored! 😒</h1>
		<h2 class="subheading">I want to have...</h2>
	</header>

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

						suggestionElement.scrollIntoView({
							behavior: 'smooth' 
						});

						startTiming();
					})();
				}}
			>{activity.name}</button>
		{/each}
	</div>

	<h4 class="suggested-activity" bind:this={suggestionElement}>
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

<style global>
	:global(body) {
		margin: 0;
		padding: 0;
		font-family: Arial, sans-serif;
	}

	::selection {
		background: #ffaa00;
		color: #fff;
	}

	header {
		width: 100%;
		height: 25rem;
		margin-bottom: 5rem;
	}

	.heading {
		margin: 0;
		display: flex;
		justify-content: center;
		align-items: flex-start;
		padding-top: 4rem;
		font-size: 4rem;
		font-weight: 700;
		color: #fff;
		animation: heading-animation 400ms cubic-bezier(.33,1.88,.57,.7);
		background: #ffaa00;
		height: 20rem;
		margin-bottom: 2rem;
	}

	.subheading {
		text-align: center;
		font-size: 2rem;
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
		margin-top: 7rem;
		margin-bottom: 3rem;
	}

	@keyframes heading-animation {
		from {
			transform: translate(10rem, 12rem);
			rotate: 60deg;
			opacity: 0;
		} to {
			transform: translate(0);
			rotate: 0deg;
			opacity: 1;
		}
	}
</style>
