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
![{{ education.school }}](/assets/images/{{ education.icon }}){: height="128px" width="128px"} 

<div>
{{ education.school }}

{{ education.start_time | date: "%b %Y" }} - {{ education.end_time | date: "%b %Y" }} 

{{ education.role }}
</div>

{% endfor %}
