<script>
  import { fetchMovies } from '../api.js';
  import { onMount } from 'svelte';
  import Search from '../components/Search.svelte';
  import { IMAGE_BASE_URL, BACKDROP_SIZE, POSTER_SIZE } from '../config';
  import Hero from '../components/Hero.svelte';
  import Grid from '../components/Grid.svelte';
  import Thumb from '../components/Thumb.svelte';
  import Spinner from '../components/Spinner.svelte';
  import LoadMoreButton from '../components/LoadMoreButton.svelte';

  let movies = { movies: [] };
  let isLoading;
  let searchTerm = '';
  let error;

  const handleFetchMovies = async (loadMore, searchTerm) => {
    try {
      isLoading = true;
      error = false;
      movies = await fetchMovies(movies, loadMore, searchTerm);
      console.log(movies);
    } catch (error) {
      error = true;
    }
    isLoading = false;
  };

  const handleSearch = event => {
    searchTerm = event.detail.searchText;
    movies.movies = [];
    handleFetchMovies(false, searchTerm);
  };

  const handleLoadMore = () => handleFetchMovies(true, searchTerm);

  onMount(async () => {
    const sessionMovies = window.sessionStorage.getItem('svelte-movies');
    if (sessionMovies) {
      console.log('Grabbing from session storage');
      movies = JSON.parse(sessionMovies);
    } else {
      console.log('Grabbing from API');
      await handleFetchMovies(false, searchTerm);
    }
  });

  $: {
    if (!searchTerm.length) {
      movies.heroImage = {};
      movies.heroImage.backdrop_path = '/pcDc2WJAYGJTTvRSEIpRZwM3Ola.jpg';
      movies.heroImage.original_title = "Zack Snyder's Justice League";
      movies.heroImage.overview =
        "Determined to ensure Superman's ultimate sacrifice was not in vain, Bruce Wayne aligns forces with Diana Prince with plans to recruit a team of metahumans to protect the world from an approaching threat of catastrophic proportions.";
    }
  }

  $: {
    if (movies.movies.length > 0) {
      window.sessionStorage.setItem('svelte-movies', JSON.stringify(movies));
    }
  }
</script>

{#if error}
  <p>Something went wrong</p>
{:else if movies.heroImage && !searchTerm}
  <Hero
    image={`${IMAGE_BASE_URL}${BACKDROP_SIZE}${movies.heroImage.backdrop_path}`}
    title={movies.heroImage.original_title}
    text={movies.heroImage.overview}
  />
{/if}

<Search on:search={handleSearch} />
<Grid header={searchTerm ? 'Search Result' : 'Popular movies'}>
  {#each movies.movies as movie (movie)}
    <Thumb
      clickable
      image={movie.poster_path &&
        IMAGE_BASE_URL + POSTER_SIZE + movie.poster_path}
      movieId={movie.id}
    />
  {/each}
</Grid>

{#if !isLoading && movies.currentPage < movies.totalPages}
  <LoadMoreButton on:loadMore={handleLoadMore}>Load More</LoadMoreButton>>
{/if}
{#if isLoading}
  <Spinner />
{/if}

<style>
</style>
