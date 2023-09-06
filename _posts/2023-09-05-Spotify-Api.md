---
layout: post
title: Spotify Api Test
description: Tests for the spotify api functions, attempts to add to queue
courses: {ToC: {week: 2}}
type: tangibles
---

<button type="button" id="login-button">Click Me!</button>

<script type="module">
    function generateRandomString(length) {
        let text = '';
        let possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    
        for (let i = 0; i < length; i++) {
        text += possible.charAt(Math.floor(Math.random() * possible.length));
        }
        return text;
    }
    
    async function generateCodeChallenge(codeVerifier) {
        function base64encode(string) {
        return btoa(String.fromCharCode.apply(null, new Uint8Array(string)))
            .replace(/\+/g, '-')
            .replace(/\//g, '_')
            .replace(/=+$/, '');
        }
    
        const encoder = new TextEncoder();
        const data = encoder.encode(codeVerifier);
        const digest = await window.crypto.subtle.digest('SHA-256', data);
    
        return base64encode(digest);
    }
    
    const clientId = 'a76d4532c6e14dd7bd7393e3fccc1185';
    const redirectUri = 'http://localhost:3000';
    
    let codeVerifier = generateRandomString(128);
    
    generateCodeChallenge(codeVerifier).then(codeChallenge => {
        let state = generateRandomString(16);
        let scope = 'user-read-private user-read-email';
    
        localStorage.setItem('code_verifier', codeVerifier);
    
        let args = new URLSearchParams({
        response_type: 'code',
        client_id: clientId,
        scope: scope,
        redirect_uri: redirectUri,
        state: state,
        code_challenge_method: 'S256',
        code_challenge: codeChallenge
        });
    
        window.location = 'https://accounts.spotify.com/authorize?' + args;
    });
    
    const urlParams = new URLSearchParams(window.location.search);
    let code = urlParams.get('code');
    
    let codeVerifier2 = localStorage.getItem('code_verifier');
    
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
    }

    document.getElementById('login-button').addEventListener('click', redirectToSpotifyAuthorizeEndpoint, false);

</script>