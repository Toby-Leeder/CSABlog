---
layout: post
title: Spotify Api Test 3
description: Tests for the spotify api functions, redirect location
courses: {ToC: {week: 3}}
type: tangibles
---

<button type="button" id="me-button">me!</button>
<button type="button" id="login-button">Profile!</button>
<button type="button" id="playSong-button">Play a song!</button>
<button type="button" id="nextSong-button">Next Song</button>
<button type="button" id="lastSong-button">Previous Song</button>
<button type="button" id="pause-button">Pause</button>
<button type="button" id="play-button">Play</button>


<script type="module">
    let codeVerifier2 = localStorage.getItem('code_verifier');
    const urlParams = new URLSearchParams(window.location.search);
    let code = urlParams.get('code');
    const redirectUri = 'https://toby-leeder.github.io/CSABlog/2023/09/05/Spotify-Api-2.html';
    const clientId = 'a76d4532c6e14dd7bd7393e3fccc1185';

    let body = new URLSearchParams({
        grant_type: 'authorization_code',
        code: code,
        redirect_uri: redirectUri,
        client_id: clientId,
        code_verifier: codeVerifier2
    });

    fetch('https://accounts.spotify.com/api/token', {
        method: 'POST',
        headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: body
    })
        .then(response => {
        if (!response.ok) {
            throw new Error('HTTP status ' + response.status);
        }
        return response.json();
        })
        .then(data => {
            console.log("localStorageIng")
            localStorage.setItem('access_token', data.access_token);
        })
        .catch(error => {
        console.error('Error:', error);
        });

    async function getProfile() {
        var accessToken = localStorage.getItem('access_token');
    
        const response = await fetch('https://api.spotify.com/v1/me/player', {
        headers: {
            Authorization: 'Bearer ' + accessToken
        }
        }).then(response => {
            if (!response.ok) {
                throw new Error('HTTP status ' + response.status);
            }
            return response.json();
        })
        .then(data => {
            // Handle the response data
        })
        .catch(error => {
            console.error('Error:', error);
        });
    }

    async function me() {
        var accessToken = localStorage.getItem('access_token');
    
        const response = await fetch('https://api.spotify.com/v1/me', {
        headers: {
            Authorization: 'Bearer ' + accessToken
        }
        }).then(response => {
            if (!response.ok) {
                throw new Error('HTTP status ' + response.status);
            }
            return response.json();
        })
        .then(data => {
            console.log(data)  
        })
        .catch(error => {
            console.error('Error:', error);
        });
    }

    async function playSong() {
        var accessToken = localStorage.getItem('access_token');
    
        const response = await fetch('https://api.spotify.com/v1/me/player/queue?uri=spotify:track:4cOdK2wGLETKBW3PvgPWqT', {
        method : "POST",
        headers: {
            Authorization: 'Bearer ' + accessToken
        }
        }).then(response => {
            if (!response.ok) {
                throw new Error('HTTP status ' + response.status);
            }
            return response.json();
        })
        .then(data => {
            console.log(data)  
        })
        .catch(error => {
            console.error('Error:', error);
        });
    }

    async function changePlayback(playback, method) {
        var accessToken = localStorage.getItem('access_token');
        const response = await fetch('https://api.spotify.com/v1/me/player/' + playback, {
        method : method,
        headers: {
            Authorization: 'Bearer ' + accessToken
        }
        }).then(response => {
            if (!response.ok) {
                throw new Error('HTTP status ' + response.status);
            }
            return response.json();
        })
        .then(data => {
            console.log(data)  
        })
        .catch(error => {
            console.error('Error:', error);
        });
    }
    document.getElementById('me-button').addEventListener('click', function() { me();}, false);
    document.getElementById('login-button').addEventListener('click', function() { getProfile();}, false);
    document.getElementById('playSong-button').addEventListener('click', function() { playSong();}, false);
    document.getElementById('nextSong-button').addEventListener('click', function() { changePlayback("next", "POST");}, false);
    document.getElementById('lastSong-button').addEventListener('click', function() { changePlayback("previous", "POST");}, false);
    document.getElementById('pause-button').addEventListener('click', function() { changePlayback("pause", "PUT");}, false);
    document.getElementById('play-button').addEventListener('click', function() { changePlayback("play", "PUT");}, false);
</script>