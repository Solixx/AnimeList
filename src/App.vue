<script setup>
import { ref, onMounted, computed } from "vue";

const query = ref("");
const my_anime = ref([]);
const search_results = ref([]);

const my_anime_asc = computed(() => {
	return my_anime.value.sort((a, b) => {
		return a.title.localeCompare(b.title);
	});
});

const searchAnime = () => {
	if (!query.value) {
		search_results.value = [];
		return;
	}

	const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;
	fetch(url)
		.then((res) => res.json())
		.then((data) => {
			search_results.value = data.data;
		});
};

const handleInput = (e) => {
	if (!e.target.value) {
		search_results.value = [];
	}
};

const addAnime = (anime) => {
	const existingAnime = my_anime.value.find((item) => item.id === anime.mal_id);

	if (!existingAnime) {
		my_anime.value.push({
			id: anime.mal_id,
			title: anime.title,
			image: anime.images.jpg.image_url,
			total_episodes: anime.episodes,
			watched_episodes: 0,
			score: anime.score,
			myscore: 0,
			status: 0	// 0 -> watching, 1 -> completed, 2 -> onHold, 3 -> dropped, 4 -> planToWatch
		});

		localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
	}
};

const removeAnime = (anime) => {
	const index = my_anime.value.findIndex((item) => item.id === anime.id);

	if (index !== -1) {
		my_anime.value.splice(index, 1);
		localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
	}
};

const editMyScore = (anime, e) => {
	if (e.target.value < 0) anime.myscore = 0;
	else if (e.target.value >= 10) anime.myscore = 10;
	else anime.myscore = e.target.value;
	localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

const editNumberWatch = (anime, e) => {
	if (e.target.value < 0) anime.watched_episodes = 0;
	else if (e.target.value >= anime.total_episodes)
		anime.watched_episodes = anime.total_episodes;
	else anime.watched_episodes = Math.round(e.target.value);
	localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

const increaseWatch = (anime) => {
	anime.watched_episodes++;
	localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

const decreaseWatch = (anime) => {
	anime.watched_episodes--;
	localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

const changeStats = (anime) => {
	if(anime.status == 4)	anime.status = 0;
	else					anime.status++;

	if(anime.status == 1)	anime.watched_episodes = anime.total_episodes

	localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
}

onMounted(() => {
	my_anime.value = JSON.parse(localStorage.getItem("my_anime")) || [];
});
</script>
<template>
	<main>
		<h1>My Anime Tracker</h1>

		<form @submit.prevent="searchAnime">
			<input type="text" placeholder="Search for an anime..." v-model="query" @input="searchAnime" />
			<button type="submit" class="button">Search</button>
		</form>

		<div class="results" v-if="search_results.length > 0">
			<div v-for="anime in search_results" class="result">
				<img :src="anime.images.jpg.image_url" />
				<div class="details">
					<h3>{{ anime.title }}</h3>
					<p :title="anime.synopsis" v-if="anime.synopsis">
						{{ anime.synopsis.slice(0, 120) }}...
					</p>
					<span class="flex-1"></span>
					<button @click="addAnime(anime)" class="button">
						Add to My Anime
					</button>
				</div>
			</div>
		</div>

		<div class="myanime" v-if="my_anime.length > 0">
			<h2>My Anime</h2>

			<div v-for="anime in my_anime_asc">
				<div class="anime animeWatching" v-if="anime.status == 0">
					<div class="changeStats" @click="changeStats(anime)"></div>
					<img :src="anime.image" />
					<h3>{{ anime.title }}</h3>
					<input type="number" v-model="anime.myscore" @blur="editMyScore(anime, $event)" />
					<span class="rate"> / 10</span>
					<div class="flex-1"></div>
					<input type="number" v-model="anime.watched_episodes" @blur="editNumberWatch(anime, $event)" />
					<span class="episodes"> / {{ anime.total_episodes }}</span>
					<div class="anime-buttons">
						<button v-if="anime.total_episodes !== anime.watched_episodes &&
							anime.total_episodes > 0
							" @click="increaseWatch(anime)" class="button">
							+
						</button>
						<button v-if="anime.watched_episodes > 0" @click="decreaseWatch(anime)" class="button">
							-
						</button>
						<button @click="removeAnime(anime)" class="button">X</button>
					</div>
				</div>
				<div class="anime animeCompleted" v-else-if="anime.status == 1">
					<div class="changeStats" @click="changeStats(anime)"></div>
					<img :src="anime.image" />
					<h3>{{ anime.title }}</h3>
					<input type="number" v-model="anime.myscore" @blur="editMyScore(anime, $event)" />
					<span class="rate"> / 10</span>
					<div class="flex-1"></div>
					<input type="number" :value="anime.watched_episodes" disabled style="color: #888;" />
					<span class="episodes"> / {{ anime.total_episodes }}</span>
					<div class="anime-buttons">
						<button @click="removeAnime(anime)" class="button">X</button>
					</div>
				</div>
				<div class="anime animeOnhold" v-else-if="anime.status == 2">
					<div class="changeStats" @click="changeStats(anime)"></div>
					<img :src="anime.image" />
					<h3>{{ anime.title }}</h3>
					<input type="number" v-model="anime.myscore" @blur="editMyScore(anime, $event)" />
					<span class="rate"> / 10</span>
					<div class="flex-1"></div>
					<input type="number" v-model="anime.watched_episodes" @blur="editNumberWatch(anime, $event)" />
					<span class="episodes"> / {{ anime.total_episodes }}</span>
					<div class="anime-buttons">
						<button v-if="anime.total_episodes !== anime.watched_episodes &&
							anime.total_episodes > 0
							" @click="increaseWatch(anime)" class="button">
							+
						</button>
						<button v-if="anime.watched_episodes > 0" @click="decreaseWatch(anime)" class="button">
							-
						</button>
						<button @click="removeAnime(anime)" class="button">X</button>
					</div>
				</div>
				<div class="anime animeDropped" v-else-if="anime.status == 3">
					<div class="changeStats" @click="changeStats(anime)"></div>
					<img :src="anime.image" />
					<h3>{{ anime.title }}</h3>
					<input type="number" v-model="anime.myscore" @blur="editMyScore(anime, $event)" />
					<span class="rate"> / 10</span>
					<div class="flex-1"></div>
					<input type="number" v-model="anime.watched_episodes" @blur="editNumberWatch(anime, $event)" />
					<span class="episodes"> / {{ anime.total_episodes }}</span>
					<div class="anime-buttons">
						<button v-if="anime.total_episodes !== anime.watched_episodes &&
							anime.total_episodes > 0
							" @click="increaseWatch(anime)" class="button">
							+
						</button>
						<button v-if="anime.watched_episodes > 0" @click="decreaseWatch(anime)" class="button">
							-
						</button>
						<button @click="removeAnime(anime)" class="button">X</button>
					</div>
				</div>
				<div class="anime animePlantowatch" v-else>
					<div class="changeStats" @click="changeStats(anime)"></div>
					<img :src="anime.image" />
					<h3>{{ anime.title }}</h3>
					<input type="number" value="0" disabled style="color: #888;" />
					<span class="rate"> / 10</span>
					<div class="flex-1"></div>
					<input type="number" value="0" disabled style="color: #888;" />
					<span class="episodes"> / {{ anime.total_episodes }}</span>
					<div class="anime-buttons">
						<button @click="removeAnime(anime)" class="button">X</button>
					</div>
				</div>
			</div>
		</div>
	</main>
</template>

<style scoped></style>
