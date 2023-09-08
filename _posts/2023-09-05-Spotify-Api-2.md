---
layout: post
title: Spotify Api Test 3
description: Tests for the spotify api functions, redirect location
courses: {ToC: {week: 3}}
type: tangibles
---

<button type="button" id="login-button">Click Me!</button>

<script type="module">
    let codeVerifier2 = localStorage.getItem('code_verifier');
    let code = urlParams.get('code');

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
        localStorage.setItem('access_token', data.access_token);
        })
        .catch(error => {
        console.error('Error:', error);
        });

    async function getProfile(accessToken) {
        accessToken = localStorage.getItem('access_token');
    
        const response = await fetch('https://api.spotify.com/v1/me', {
        headers: {
            Authorization: 'Bearer ' + accessToken
        }
        });
    
        const data = await response.json();
        console.log(data);
    }
    document.getElementById('login-button').addEventListener('click', function() { getProfile();}, false);
</script>