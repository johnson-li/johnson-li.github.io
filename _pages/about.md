---
permalink: /about/
title: "About Me"
layout: splash
header:
  overlay_image: /assets/images/sunset.jpg
  overlay_color: "#000"
  overlay_filter: "0.1"
educations:
  - school: Aalto University
    start_time: 2018-01-01
    end_time: 2018-12-31
    role: Research assistant
    icon: aalto.svg
  - school: Fudan University
    start_time: 2016-09-01
    end_time: 2019-06-01
    role: Master student
    icon: fudan.png
  - school: Fudan University
    start_time: 2012-09-01
    end_time: 2016-06-01
    role: Undergraduate student
    icon: fudan.png
---

Education
===

{% for education in page.educations %}

<div style="display: table;clear: both;content: ; padding-top: 10px; padding-bottom: 10px" height="128" width="128">
    <div style="float: left;width: 30%;">
        <img src="/assets/images/{{ education.icon }}" alt="school icon" height="128" width="128" style="margin: auto;top: 0;left: 0;right: 0;bottom: 0;">
    </div>

    <div style="display: table-cell;float: right;width: 70%;padding-left: 30px;">
        {{ education.school }}
        <br/>
        {{ education.start_time | date: "%b %Y" }} - {{ education.end_time | date: "%b %Y" }} 
        <br/>
        {{ education.role }}
    </div>
</div>

{% endfor %}
