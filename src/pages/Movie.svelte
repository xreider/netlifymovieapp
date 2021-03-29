<script>
  import { fetchMovie } from '../api.js';
  import Actor from '../components/Actor.svelte';
  import Grid from '../components/Grid.svelte';
  import MovieInfoBar from '../components/MovieInfoBar.svelte';
  import MovieInfo from '../components/MovieInfo.svelte';
  import Spinner from '../components/Spinner.svelte';
  import Navigation from '../components/Navigation.svelte';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';

  export let params;

  let isLoading;
  let error;
  let movie;

  const handleFetchMovie = async () => {
    try {
      isLoading = true;
      error = false;
      movie = await fetchMovie(params.id);
      console.log(movie);
    } catch (err) {
      error = true;
    }
    isLoading = false;
  };

  onMount(async () => {
    const localMovie = window.localStorage.getItem(params.id);
    if (localMovie) {
      console.log('Grab movie from localStorage');
      movie = JSON.parse(localMovie);
    } else {
      console.log('Grab movie from API');
      handleFetchMovie();
    }
  });

  $: {
    if (movie) {
      window.localStorage.setItem(params.id, JSON.stringify(movie));
    }
  }
</script>

{#if error}
  <p>Something went wrong</p>
{:else if movie}
  <div transition:fade={{ duration: 300 }}>
    <Navigation movie={movie.original_title} />
    <MovieInfo {movie} />
    <MovieInfoBar
      time={movie.runtime}
      budget={movie.budget}
      revenue={movie.revenue}
    />
    <Grid header="Actors">
      {#each movie.actors as actor}
        <Actor {actor} />
      {/each}
    </Grid>
  </div>
{/if}

{#if isLoading}
  <Spinner />
{/if}

<style>
</style>
