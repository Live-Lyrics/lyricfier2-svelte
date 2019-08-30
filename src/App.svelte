<script>
	import debounce from 'lodash/debounce';

	let song = {
		title: '',
		artist: '',
		artUrl: '',
		lyric: '',
		source: ''
	}

	let app = null;
	function update() {
		fetch('/status')
			.then(
				function (response) {
					if (response.status !== 200) {
						return;
					}
					response.json().then(function (data) {
							if (JSON.stringify(song) !== JSON.stringify(data)) {
								song = data.song;
							}
					});
				}
			)
			.catch(function (err) {
			});
	}
	const debouncedUpdate = debounce(update, 1000);
	function setup() {
		update();
		const conn = new WebSocket("ws://" + document.location.host + "/ws");
		conn.onclose = function (evt) {
			console.log('Connection error', evt)
		};
		conn.onmessage = function (evt) {
			debouncedUpdate();
		};
	}
	document.addEventListener('DOMContentLoaded', setup, false);

</script>

<style>
	.full-vertical-flex {
		display: flex;
		flex-direction: column;
		height: 100vh;
	}

	.header {
		position: relative;
		overflow: hidden;
		min-height: 8rem;
		box-shadow: 0 0.25rem 1rem rgba(0, 0, 0, 0.1);
	}

	.header::before {
		content: '';
		position: absolute;
		bottom: 0;
		left: 0;
		height: 0.0625rem;
		width: 100%;
		background: black;
		opacity: 0.1;
	}

	.header__background {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		opacity: 0.25;
		background-size: cover;
		background-position: center center;
		-webkit-filter: blur(10px);
		transform: translate3d(0, 0, 0);
		z-index: -1;
	}

	.header__album-art {
		float: left;
		width: 8rem;
		height: 8rem;
		margin-right: 1rem;
	}

	.header h3,
	.header h4 {
		font-weight: 300;
		margin: 0 1rem 0 0;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.header h3 {
		font-size: 1.5rem;
		margin-top: 1rem;
		color: currentColor;
	}

	.header h4 {
		color: currentColor;
	}

	.lyrics {
		white-space: pre-wrap;
		flex: 1;
		overflow: auto;
		padding: 1rem;
		line-height: 1.75;
		position: relative;
	}

	.lyrics:first-letter {
		text-transform: capitalize;
	}

	.credits-source {
		padding: 1rem;
		font-size: .65em;
	}

	.credits-source a {
		color: #03A9F4;
	}

	.not-playing-view {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;
		text-align: center;
		padding: 0 2rem;
	}

	.not-playing-view img {
		max-width: 16rem;
	}

	.not-playing-view h3 {
		font-size: 2rem;
		font-weight: 300;
	}

	.not-playing-view p {
		max-width: 80%;
		font-size: 0.85rem;
		line-height: 1.75;
		margin-bottom: 2rem;
	}


	::-webkit-scrollbar {
		width: 0.5rem;
		height: 0.5rem;
	}

	::-webkit-scrollbar-track {
		background: transparent;
	}

	::-webkit-scrollbar-thumb {
		border-radius: 0.25rem;
		background: transparent;
	}

	:hover ::-webkit-scrollbar-thumb {
		background: rgba(0, 0, 0, 0.15);
	}

	a {
		outline: none;
	}
</style>



{#if song.title}
	<div v-if="song" class="full-vertical-flex">
		<header class="header" >
			{#if song.artUrl}
                <span  class="header__background" style="background-image: url({song.artUrl});"></span>
                <img   class="header__album-art" src={song.artUrl} alt={song.artist}>
            {:else}
                <span class="header__background" style="background-image: url(/static/img/icon.png);"></span>
                <img  class="header__album-art" style="padding: 5px; box-sizing: border-box;" src="/static/img/icon.png" alt={song.artist} />
            {/if}
            <h3>{ song.title }</h3>
            <h4>{ song.artist }</h4>
        </header>
		<div  id="lyricBox" class="lyrics">
			{ song.lyric }
		</div>
		<div class="credits-source">Source:
            <a rel="noopener" target="_blank" href={ song.source } >
                { song.source }
            </a>
        </div>
	</div>
{:else}
	<div class="full-vertical-flex">
		<div class="not-playing-view">
              <img src="/static/img/waves.svg" alt="Waveform">
              <h3>Looking for a Song on Spotify</h3>
              <p>Connecting...</p>
          </div>
	</div>
{/if}