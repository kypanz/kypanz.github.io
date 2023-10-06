---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: Projects
---

<style>

.card {
    margin: 5px;
    background: black;
    width:40%;
    height:auto;
    padding: 20px;
    border-radius: 10px;
}

.btn {
    background:red;
    width:100%;
    display:block;
    text-align:center;
    padding:5px;
    color:white !important;
}

.btn:hover {
    background: greenyellow;
    color:black !important;
}

.box {
    display:flex;
    align-items:center;
    justify-content:center;
}

img.post-project {
    display:block;
    border: 5px solid greenyellow;
    width:100%;
}


.title {
    color: red;
}

section#box-cards {
    width: 100%;
    display: flex;
    justify-content:center;
}

</style>


<section id="box-cards">
{% for project in site.data.projects %}
<div class="card">
    <div class="box">
        <img class="post-project" src="{{ project.image_url }}" />
    </div>
    <p class="title"> Description </p>
    <p> {{ project.description }} </p>
    <div class="box">
        <a href="{{ project.link }}" class="btn"> Take a Look </a>
    </div>
</div>

{% endfor %}
<div/>
