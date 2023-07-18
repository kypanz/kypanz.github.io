---
layout: page
title: About Me
permalink: /about/
---
<style>
        code {
            white-space: pre-wrap;
            background-color: black;
            color: yellowgreen;
            padding: 1px;
            width: 100%;
            display: block;
        }
    </style>


Fullstack developer, in my free time i build things just for fun
also i like practice offensive cybersecurity too :)

<div align="center" style="display:flex;justify-content:center;">
    <img src="https://media.tenor.com/JAZzfZupTTcAAAAS/gil-cat.gif" />
</div>

# About you

This is a good reason of why you need a VPN ( Virtual Private Network ) to keep your information safe.
<code id="visitorLocation">
</code>

  <script>
    fetch('https://ipapi.co/json/')
        .then(response => response.json())
        .then(data => {
            document.getElementById('visitorLocation').textContent = JSON.stringify(data, null, 2)  //`${data.city}, ${data.region}, ${data.country_name}`;
        })
        .catch(error => {
            console.log('Error al obtener la información de ubicación:', error);
        });
  </script>
