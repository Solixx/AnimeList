<script setup>
import { ref, onMounted, computed } from 'vue'

const query = ref('')
const my_anime = ref([])
const search_results = ref([])

const my_anime_asc = computed(() => {
	return my_anime.value.sort((a, b) => {
		return a.title.localeCompare(b.title)
	})
})

const searchAnime = () => {
  if(!query.value){
    search_results.value = []
    return
  }

	const url = `https://api.jikan.moe/v4/anime?q=${query.value}`
	fetch(url)
		.then(res => res.json())
		.then(data => {
			search_results.value = data.data
		})
}

const handleInput = (e) => {
	if (!e.target.value) {
		search_results.value = []
	}
}

const addAnime = (anime) => {
	const existingAnime = my_anime.value.find(item => item.id === anime.mal_id);

  if (!existingAnime) {
    my_anime.value.push({
      id: anime.mal_id,
      title: anime.title,
      image: anime.images.jpg.image_url,
      total_episodes: anime.episodes,
      watched_episodes: 0,
      score: anime.score,
      myscore: 0,
    });

    localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
  }
}

const removeAnime = (anime) => {

  const index = my_anime.value.findIndex(item => item.id === anime.id);

  if (index !== -1) {
    my_anime.value.splice(index, 1);
    localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
  }
}

const editMyScore = (anime, e) => {
  if(e.target.value < 0)  anime.myscore = 0
  else if(e.target.value >= 10) anime.myscore = 10
  else  anime.myscore = e.target.value
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const editNumberWatch = (anime, e) => {
  if(e.target.value < 0)  anime.watched_episodes = 0
  else if(e.target.value >= anime.total_episodes) anime.watched_episodes = anime.total_episodes
  else  anime.watched_episodes = e.target.value
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const increaseWatch = (anime) => {
	anime.watched_episodes++
	localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const decreaseWatch = (anime) => {
	anime.watched_episodes--
	localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

onMounted(() => {
	my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || []
})
</script>

<template>
	<main>
		<h1>My Anime Tracker</h1>

		<form @submit.prevent="searchAnime">
			<input type="text" placeholder="Search for an anime..." v-model="query" @input="handleInput" />
			<button type="submit" class="button">Search</button>
		</form>

		<div class="results" v-if="search_results.length > 0">
			<div v-for="anime in search_results" class="result">
				<img :src="anime.images.jpg.image_url" />
				<div class="details">
					<h3>{{ anime.title }}</h3>
					<p :title="anime.synopsis" v-if="anime.synopsis">{{ anime.synopsis.slice(0, 120) }}...</p>
					<span class="flex-1"></span>
					<button @click="addAnime(anime)" class="button">Add to My Anime</button>
				</div>
			</div>
		</div>

		<div class="myanime" v-if="my_anime.length > 0">
			<h2>My Anime</h2>

			<div v-for="anime in my_anime_asc" class="anime">
				<img :src="anime.image" />
				<h3>{{ anime.title }}</h3>
        <input type="number" v-model="anime.myscore"  @blur="editMyScore(anime, $event)">
        <span> / 10</span>
				<div class="flex-1"></div>
        <input type="number" v-model="anime.watched_episodes"  @blur="editNumberWatch(anime, $event)">
				<span class="episodes"> / {{ anime.total_episodes }}</span>
        <div class="anime-buttons">
          <button 
					v-if="(anime.total_episodes !== anime.watched_episodes) && (anime.total_episodes > 0)" 
					@click="increaseWatch(anime)" class="button">+</button>
          <button 
            v-if="anime.watched_episodes > 0"
            @click="decreaseWatch(anime)" class="button">-</button>
          <button 
            @click="removeAnime(anime)" class="button">X</button>
        </div>
			</div>
		</div>
	</main>
</template>

<style scoped>
  *{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Fira sans', sans-serif;
  }

  body{
    background-color: #EEE;
  }

  main{
    margin: 0 auto;
    max-width: 768px;
    padding: 1.5rem;
  }

  h1{
    text-align: center;
    margin-bottom: 1.5rem;
  }

  form{
    display: flex;
    max-width: 480px;
    margin: 0 auto 1.5rem;
  }

  form input{
    appearance: none;
    outline: none;
    border: none;
    background-color: white;
    display: block;
    color: #888;
    font-size: 1.125rem;
    padding: 0.5rem 1rem;
    width: 100%;
  }

  .button{
    appearance: none;
    outline: none;
    border: none;
    cursor: pointer;
    background: none;
    display: block;
    padding: 0.5rem 1rem;
    background-image: linear-gradient(to right, deeppink 50%, darkviolet 50%);
    background-size: 200%;
    color: #FFF;
    font-weight: bold;
    text-transform: uppercase;
    transition: 0.4s;
  }

  .button:hover{
    background-position: right;
  }

  .results{
    background-color: #FFF;
    border-radius: 0.5rem;
    box-shadow: 0 2ox 4px rgba(0, 0, 0, 0.1);
    max-height: 480px;
    overflow-y: scroll;
    margin-bottom: 1.5rem;
  }

  .result{
    display: flex;
    margin: 1rem;
    padding: 1rem;
    transition: 0.4s;
  }

  .result img{
    widows: 100px;
    border-radius: 1rem;
    margin-right: 1rem;
  }

  .details{
    flex: 1 1 0%;
    display: flex;
    align-items: flex-start;
    flex-direction: column;
  }

  .flex-1{
    flex: 1 1 0%;
  }

  .details h3{
    font-size: 1.25rem;
    margin-bottom: 0.5rem;
  }

  .details p{
    font-size: 0.875rem;
    line-height: 1.4;
    margin-bottom: 0.5rem;
  }

  .details .button{
    margin-left: auto;
  }

  .myanime h2{
    color: #888;
    font-weight: 400;
    margin-bottom: 1.5rem;
  }

  .myanime .anime{
    display: flex;
    align-items: center;
    margin-bottom: 1.5rem;
    background-color: #FFF;
    padding: 1rem;
    border-radius: 0.5rem;
    box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
  }

  .anime img{
    widows: 72px;
    height: 72px;
    object-fit: cover;
    border-radius: 1rem;
    margin-right: 1rem;
  }

  .anime h3{
    color: #888;
    font-size: 1.125rem;
    width: auto;
    max-width: 14rem;
  }

  .anime .episodes{
    margin-right: 1rem;
    color: #888;
  }

  .anime input{
    margin-right: 5px;
    appearance: none;
    outline: none;
    border: none;
    background-color: white;
    display: block;
    color: #066fe7;
    font-size: 1rem;
    width: auto;
    max-width: 5rem;
    text-align: right;
  }

  .anime input:focus{
    color: #888;
  }

  .anime input::-webkit-outer-spin-button,
  .anime input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }

  .anime .anime-buttons{
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }

  .anime .button{
    margin-right: 0.5rem;
  }
</style>
