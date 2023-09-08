---
layout: post
title: Spotify Api Test 3
description: Tests for the spotify api functions, redirect location
courses: {ToC: {week: 3}}
type: tangibles
---

<button type="button" id="login-button">Click Me!</button>

<script type="module">
    async function getProfile(accessToken) {
        accessToken = localStorage.getItem('access_token');
    
        const response = await fetch('https://api.spotify.com/v1/me', {
        headers: {
            Authorization: 'Bearer ' + accessToken
        }
        });
    
        const data = await response.json();
    }
</script>