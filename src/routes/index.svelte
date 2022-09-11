<script context="module">
	export function load({fetch}) {
		return fetch('https://svelte-course-f0c66-default-rtdb.firebaseio.com/meetups.json')
		.then(res => {
			if (!res.ok) {
				throw new Error('Fetching meetups failed, please try again later!');
			}
			return res.json();
		})
		.then(data => {
			const fetchedMeetups = [];
			for (const key in data) {
				fetchedMeetups.push({
				...data[key],
				id: key
				});
			}
			fetchedMeetups.reverse();
			return {
				props: {fetchedMeetups}
			};
			/*
			setTimeout(() => {
				isLoading = false;
				meetups.seetupMeetup(loadedMeetups.reverse());
			}, 1000);
			*/
		})
		.catch(err => {
			isLoading = false;
			console.log(err);
			error(500, 'Could not fetch meetups!');
		});
		
	}
</script>

<script>
	import EditMeetup from "../Meetups/EditMeetup.svelte";
	import meetups from '../Meetups/meetups-store.js';
	import LoadingSpinner from '../UI/LoadingSpinner.svelte';
	import Error from '../UI/Error.svelte';
	import MeetupItem from "../Meetups/MeetupItem.svelte";
	import MeetupFilter from '../Meetups/MeetupFilter.svelte';
	import Button from '../UI/Button.svelte';
	import { onMount, onDestroy } from 'svelte';
	import { scale } from 'svelte/transition';
	import { flip } from 'svelte/animate'; 

	export let fetchedMeetups;
  
	let editMode;
	let editedId;
	let isLoading;
	let error;
	let loadedMeetups = [];
	let unsubscribe;

  let favsOnly = false;

  $: filteredMeetups = favsOnly ? loadedMeetups.filter(m => m.isFavorite) : loadedMeetups;
  
  function setFilter(event) {
    favsOnly = event.detail === 1; 
  }
  
	onMount(() => {
		unsubscribe = meetups.subscribe(items => {
			loadedMeetups = items; 
		});
		meetups.seetupMeetup(fetchedMeetups); 
	});

	onDestroy(() => {
		if (unsubscribe) {
			unsubscribe();
		}
	});
  
	function savedMeetup(event) {
	  editMode = null;
	  editedId = null;
	}
  
	function cancelEdit() {
	  editMode = null;
	  editedId = null;
	}
  
	function startEdit(event) {
	  editMode = 'edit';
	  editedId = event.detail;
	}

	function startAdd() {
		editMode = 'edit';
	}
  
	function clearError() {
	  error = null;
	}
  </script>
  
  <style>
	main {
	  margin-top: 5rem;
	}
	#meetups {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 1rem;
  }

  #meetup-control {
    margin: 1rem;
    display: flex;
    justify-content: space-between;
  }

  #no-meetups {
    margin: 1rem;
  }

  @media (min-width: 768px) {
    #meetups {
      grid-template-columns: repeat(2, 1fr);
    }
  }
  </style>

	<svelte:head>
		<title>Home</title>
		<meta name="description" content="Svelte demo app" />
	</svelte:head>
  
	{#if error}
		<Error message={error.message} on:cancel={clearError} /> 
  	{/if} 

  <main>
	  {#if editMode === 'edit'}
		<EditMeetup id={editedId} on:save={savedMeetup} on:cancel={cancelEdit} />
	  {/if}
	  {#if isLoading}
		<LoadingSpinner />
	  {:else}
	  <section id="meetup-control">
		<MeetupFilter on:select={setFilter} />
		<Button on:click={startAdd}>New Meetup</Button>
	  </section>
	  
	  {#if filteredMeetups.length === 0}
		<p id="no-meetups">No meetups found, you can start adding some.</p>
	  {/if}  
	  <section id="meetups">
		{#each filteredMeetups as meetup (meetup.id)}
		  <div transition:scale animate:flip={{duration: 300}}> 
			<MeetupItem 
			  id={meetup.id}
			  title={meetup.title} 
			  subtitle={meetup.subtitle}
			  description={meetup.description}
			  imageUrl={meetup.imageUrl}
			  email={meetup.contactEmail}
			  address={meetup.address}
			  isFav={meetup.isFavorite}
			  on:edit={startEdit} />
		  </div>
		{/each}
	  </section>
	  {/if}
  </main>
  




