<script>
    import { getAllFollowedArtists, getAllArtistsTracks } from "$lib/spotifyApi/requests";
    import { token } from "$lib/stores/authenication.js";
    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();
    let loading = false;

    async function loadTracks(){
        loading = true;
        let artists = await getAllFollowedArtists("https://api.spotify.com/v1/me/following?type=artist&limit=50", $token)

        localStorage.setItem('artists', JSON.stringify(artists));
        console.log("Saved artists in local storage");

        let tracks = [];
        for (const artist of artists) {
            let artistTracks = await getAllArtistsTracks("https://api.spotify.com/v1/artists/" + artist.id + "/top-tracks?market=gb", $token)

            for (const track of artistTracks) {
                track.artist = {
                    name: artist.name,
                    href: artist.href,
                    id: artist.id,
                    uri: artist.uri
                };
                tracks.push(track);
            }
        }

        localStorage.setItem('tracks', JSON.stringify(tracks));

        console.log("Saved tracks in local storage");

        dispatch('tracksSaved');
        loading = false;
    }
</script>

<div class="container mx-auto px-12 text-center flex flex-col gap-4">
    <p class="font-bold">Connected to your Spotify account.</p>
    <p>Beat the Intro currently only uses artist's you have 'favourited' with a green heart. If you have not favourited any (or many) artists, go into your spotify and add a few now.</p>
    <p>When you're done. Click the button below to sync your tracks with Beat the Intro.</p>
    {#if loading}
        <div class="flex justify-center">
            <progress class="progress w-56"></progress>
        </div>
    {:else}
        <button class="btn" on:click="{() => loadTracks()}">Sync Tracks</button>
    {/if}
</div>