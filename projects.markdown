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
        <img class="post-project" src="https://github.com/kypanz/blockchain-transactions-filter-in-console-with-charts/assets/37570367/59b012f1-84a3-43fb-b7e6-abbc1cd8d022" />
    </div>
    <p class="title"> Description </p>
    <p> Blockchain transactions filter by balance in real time </p>
    <div class="box">
        <a href="/projects/blockchain-filter-console" class="btn"> Take a Look </a>
    </div>
</div>

{% endfor %}
<div/>
