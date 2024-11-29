<script>
  import { onMount } from "svelte";
  import { writable } from "svelte/store";
  import "../app.css";

  let audio;
  let isPlaying = writable(false);
  let currentTime = writable(0);
  let duration = writable(0);
  let volume = writable(1);

  let playlist = [
    { src: "/music/Mariya.mp3", title: "Mariya", artist: "Zohid", image: "https://i1.sndcdn.com/artworks-QFD1znlHN6BTcBhw-Xfip3g-t500x500.jpg" },
    { src: "/music/Sogintirma.mp3", title: "Sog'intirma", artist: "Meline Madmusayeva", image: "https://cdn-images.dzcdn.net/images/artist/4a50c33e01ad70274d99c228d74ac9a4/1900x1900-000000-80-0-0.jpg" },
    { src: "/music/GozlerQara.mp3", title: "Gozler Qara", artist: "Orxan", image: "https://i.ytimg.com/vi/vBVwRxdlcqk/maxresdefault.jpg" },
  ];

  let selectedSong = playlist[0];

  onMount(() => {
    loadSong(selectedSong);
  });

  function loadSong(song) {
    if (audio) {
      audio.pause();
      isPlaying.set(false);
    }

    audio = new Audio(song.src);
    currentTime.set(0);
    duration.set(0);

    audio.volume = $volume;

    audio.ontimeupdate = () => {
      currentTime.set(audio.currentTime);
    };

    audio.onloadedmetadata = () => {
      duration.set(audio.duration);
    };

    audio.onended = () => {
      nextSong(); 
    };
  }

  function togglePlay() {
    if (audio.paused) {
      audio.play();
      isPlaying.set(true);
    } else {
      audio.pause();
      isPlaying.set(false);
    }
  }

  function selectSong(song) {
    selectedSong = song;
    loadSong(song);
    audio.play();  
    isPlaying.set(true);
  }

  function seek(time) {
    audio.currentTime = time;
    currentTime.set(time);
  }

  function changeVolume(e) {
    audio.volume = e.target.value;
    volume.set(audio.volume);
  }

  function skip(seconds) {
    seek(audio.currentTime + seconds);
  }

  function formatTime(time) {
    const minutes = Math.floor(time / 60);
    const seconds = Math.floor(time % 60).toString().padStart(2, "0");
    return `${minutes}:${seconds}`;
  }

  function nextSong() {
    const currentIndex = playlist.indexOf(selectedSong);
    const nextIndex = (currentIndex + 1) % playlist.length; // Davriy o'tish
    selectSong(playlist[nextIndex]);
  }

  function prevSong() {
    const currentIndex = playlist.indexOf(selectedSong);
    const prevIndex = (currentIndex - 1 + playlist.length) % playlist.length; // Davriy o'tish
    selectSong(playlist[prevIndex]);
  }
</script>

<div class="player bg-gray-100 p-5 rounded-lg shadow-lg max-w-md mx-auto text-center">
  <div class="playlist mb-5">
    <h3 class="text-lg font-semibold mb-3">Playlist</h3>
    <ul class="list-none p-0">
      {#each playlist as song}
        <li on:click={() => selectSong(song)} class="p-3 cursor-pointer border-b border-gray-300 hover:bg-green-500 hover:text-white transition-colors {song === selectedSong ? 'bg-green-500 text-white' : ''}">
          <span>{song.title} - {song.artist}</span>
        </li>
      {/each}
    </ul>
  </div>

  <div class="song-info">
    <div class="album-art mb-4">
      <img src={selectedSong.image} alt="Album Art" class="w-full h-80 object-fill rounded-lg" />
    </div>
    <h2 class="text-2xl font-bold mb-1">{selectedSong.title}</h2>
    <p class="text-gray-600">{selectedSong.artist}</p>
  </div>

  <div class="controls flex justify-between items-center my-5">
    <button on:click={prevSong} class="text-xl text-gray-600 hover:text-gray-800">
      <i class="fa-solid fa-backward"></i> Oldingi
    </button>
    <button on:click={() => skip(-15)} class="text-xl text-gray-600 hover:text-gray-800">
      <i class="fa-solid fa-backward-step"></i>
    </button>
    <button class="play-btn bg-green-500 text-white px-4 py-3 flex justify-center items-center rounded-full text-3xl" on:click={togglePlay}>
      <i class="fa-solid fa-{$isPlaying ? 'pause' : 'play'}"></i>
    </button>
    <button on:click={() => skip(15)} class="text-xl text-gray-600 hover:text-gray-800">
      <i class="fa-solid fa-forward-step"></i>
    </button>
    <button on:click={nextSong} class="text-xl text-gray-600 hover:text-gray-800">
      Keyingi <i class="fa-solid fa-forward"></i>
    </button>
  </div>

  <div class="progress">
    <input
      type="range"
      min="0"
      max={$duration}
      step="0.1"
      bind:value={$currentTime}
      on:change={(e) => seek(e.target.value)}
      on:input={(e) => seek(e.target.value)}
      class="w-full"
    />
    <div class="time flex justify-between text-sm text-gray-500 mt-2">
      <span>{formatTime($currentTime)}</span>
      <span>{formatTime($duration)}</span>
    </div>
  </div>

  <div class="volume flex items-center gap-3 mt-5">
    <i class="fa-solid fa-volume-high text-gray-600"></i>
    <input
      type="range"
      min="0"
      max="1"
      step="0.01"
      bind:value={$volume}
      on:input={changeVolume}
      class="flex-grow"
    />
  </div>
</div>
