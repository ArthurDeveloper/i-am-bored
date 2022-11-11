<script>
	export let activities = [
		{
			name: 'Fun!',
			apiEquivalent: 'recreational'
		},
		{
			name: 'Learning!',
			apiEquivalent: 'education'
		},
		{
			name: 'Socialization!',
			apiEquivalent: 'social'
		},
		{
			name: 'Something to do by myself!',
			apiEquivalent: 'diy'
		},
		{
			name: 'People to help!',
			apiEquivalent: 'charity',
		},
		{
			name: 'A good time in kitchen!',
			apiEquivalent: 'cooking'
		},
		{
			name: 'Relaxation!',
			apiEquivalent: 'relaxation'
		},
		{
			name: 'A good song listening!',
			apiEquivalent: 'music',
		},
		{
			name: 'Something to be busy about!',
			apiEquivalent: 'busywork',
		}
	];

	export let suggestedActivity = '';

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
		const data = req.json();
		return data;
	}
</script>

<main>
	<h1>I am bored! 😒</h1>

	<h3>I want to have...</h3>
	<div>
		{#each activities as activity}
			<button on:click={() => {
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
				
					startTiming();
				})();
			}}>{activity.name}</button>
		{/each}
	</div>

	<span>{suggestedActivity}</span>
</main>
