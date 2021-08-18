---
layout: single
author_profile: true
title: Micro Blog
permalink: /micro/
---

<div class="container row">
    {% for step in site.steps %}
    <div class="item">
        <i class="vertical-line"></i>
        <h2 class="item-date">{{ step.date | date: '%m/%Y' }}{% if step.enddate %} - {{ step.enddate | date: '%m/%Y' }}{% endif %}</h2>
        <div class="card-panel">
            <h3 class="card-title">
                {{ step.title }}
            </h3>
            <p>
                {{ step.content }}
            </p>
        </div>
    </div>
    {% endfor %}
    <div class="last-item">
        <i class="vertical-line"></i>
        
    </div>
    </div>
