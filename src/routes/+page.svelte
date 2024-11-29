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
    {
      src: "/music/Mariya.mp3",
      title: "Mariya",
      artist: "Zohid",
      image:
        "https://i1.sndcdn.com/artworks-QFD1znlHN6BTcBhw-Xfip3g-t500x500.jpg",
    },
    {
      src: "/music/Sogintirma.mp3",
      title: "Sog'intirma",
      artist: "Meline Madmusayeva",
      image:
        "https://cdn-images.dzcdn.net/images/artist/4a50c33e01ad70274d99c228d74ac9a4/1900x1900-000000-80-0-0.jpg",
    },
    {
      src: "/music/GozlerQara.mp3",
      title: "Gozler Qara",
      artist: "Orxan",
      image: "https://i.ytimg.com/vi/vBVwRxdlcqk/maxresdefault.jpg",
    },
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
    if (selectedSong === song && !$isPlaying) {
      // Resume playback if clicking the same song that's paused
      togglePlay();
    } else {
      selectedSong = song;
      loadSong(song);
      audio.play();
      isPlaying.set(true);
    }
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
    let newTime = audio.currentTime + seconds;
    if (newTime < 0) newTime = 0; // Prevent negative time
    if (newTime > audio.duration) newTime = audio.duration; // Prevent exceeding duration
    seek(newTime);
  }

  function formatTime(time) {
    const minutes = Math.floor(time / 60);
    const seconds = Math.floor(time % 60).toString().padStart(2, "0");
    return `${minutes}:${seconds}`;
  }

  function nextSong() {
    const currentIndex = playlist.indexOf(selectedSong);
    const nextIndex = (currentIndex + 1) % playlist.length;
    selectSong(playlist[nextIndex]);
  }

  function prevSong() {
    const currentIndex = playlist.indexOf(selectedSong);
    const prevIndex = (currentIndex - 1 + playlist.length) % playlist.length;
    selectSong(playlist[prevIndex]);
  }
</script>

<div class="bg-gray-900 text-white min-h-screen flex items-center justify-center">
  <div class="player bg-gray-800 p-6 rounded-lg shadow-lg max-w-lg w-full">
    <!-- Playlist -->
    <div class="playlist bg-gray-800 p-5 rounded-lg shadow-md mb-6">
      <h3 class="text-lg font-semibold text-gray-200 mb-4">Playlist</h3>
      <ul class="space-y-3">
        {#each playlist as song}
          <li
            on:click={() => selectSong(song)}
            class="flex items-center gap-3 p-3 rounded-md transition-colors duration-200 cursor-pointer 
                  {song === selectedSong ? 'bg-green-500 text-white' : 'bg-gray-700 text-gray-300 hover:bg-gray-600 hover:text-white'}"
          >
            <!-- Song Thumbnail -->
            <img
              src={song.image}
              alt="{song.title}"
              class="w-12 h-12 object-cover rounded-lg shadow-md"
            />

            <!-- Song Details -->
            <div class="flex-1">
              <p class="text-sm font-semibold">{song.title}</p>
              <p class="text-xs text-gray-400">{song.artist}</p>
            </div>

            <!-- Play/Pause Icon -->
            <div class="text-lg">
              {#if song === selectedSong && $isPlaying}
                <i class="fas fa-pause-circle"></i>
              {:else}
                <i class="fas fa-play-circle"></i>
              {/if}
            </div>
          </li>
        {/each}
      </ul>
    </div>

    <!-- Now Playing -->
    <div class="song-info text-center mb-6">
      <div class="album-art mb-4">
        <img
          src={selectedSong.image}
          alt="Album Art"
          class="w-48 h-48 object-cover rounded-lg mx-auto"
        />
      </div>
      <h2 class="text-2xl font-bold">{selectedSong.title}</h2>
      <p class="text-gray-400">{selectedSong.artist}</p>
    </div>

    <!-- Playback Controls -->
    <div class="controls flex justify-between items-center my-5">
      <button
        on:click={prevSong}
        class="text-xl text-gray-300 hover:text-white"
      >
        <i class="fas fa-backward"></i>
      </button>

      <button
        on:click={() => skip(-15)}
        class="text-lg text-gray-300 hover:text-white"
      >
        <i class="fas fa-reply"></i> -15s
      </button>

      <button
        class="play-btn bg-green-500 text-white px-4 py-3 flex justify-center items-center rounded-full text-3xl"
        on:click={togglePlay}
      >
        <i class="fas fa-{$isPlaying ? 'pause' : 'play'}"></i>
      </button>

      <button
        on:click={() => skip(15)}
        class="text-lg text-gray-300 hover:text-white"
      >
        +15s <i class="fas fa-share"></i>
      </button>

      <button
        on:click={nextSong}
        class="text-xl text-gray-300 hover:text-white"
      >
        <i class="fas fa-forward"></i>
      </button>
    </div>

    <!-- Progress Bar -->
    <div class="progress">
      <input
        type="range"
        min="0"
        max={$duration}
        step="0.1"
        bind:value={$currentTime}
        on:input={(e) => seek(e.target.value)}
        class="w-full"
      />
      <div class="time flex justify-between text-sm text-gray-400 mt-2">
        <span>{formatTime($currentTime)}</span>
        <span>{formatTime($duration)}</span>
      </div>
    </div>

    <!-- Volume Control -->
    <div class="volume flex items-center gap-3 mt-5">
      <i class="fas fa-volume-high text-gray-300"></i>
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
</div>
