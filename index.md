---
layout: page
title: Tone Trixie
image:
  path: /assets/og/og-home.jpg
  width: 1200
  height: 630
---

**Can a $7 codec chip make a piezo-equipped acoustic guitar sound like it was recorded with a $300 studio microphone?**

<img class="hero-image" src="{{ '/episodes/2026-09-pushing-up-daisies/assets/two-brains.jpg' | relative_url }}" alt="Two pedals side by side on a green cutting mat: the original in a wooden gift box printed dankie / THANK YOU, and Tone Trixie, a white box a quarter of the size, hand-lettered in red, with two footswitches, a display window and a knob.">

This is the build log for a real-time guitar pedal that does exactly that — impulse-response convolution turning the harsh, nasal "quack" of an under-saddle pickup into the warm, miked sound of the guitar's body. It started on a Raspberry Pi Pico 2 in a repurposed gift box and became **Tone Trixie**: a Daisy Seed 3 in a hand-lettered box, 3 ms of latency, a looper, a set list on an SD card, and a noise floor below the instruments used to measure it. Battery-powered, built on the bench in Cape Town.

## Build log

<ul class="episode-list">
{%- assign episodes = site.episodes | sort: "date" | reverse -%}
{%- for ep in episodes %}
  <li>
    <a href="{{ ep.url | relative_url }}"><strong>{{ ep.title }}</strong></a><br>
    <small>Episode {{ ep.episode }} · {{ ep.date | date: "%B %Y" }}</small>
    {%- if ep.subtitle %}<br><em>{{ ep.subtitle }}</em>{% endif %}
  </li>
{%- endfor %}
</ul>

---

Source, schematics, and firmware: [github.com/dylangmiles/daisy-tone-trixter](https://github.com/dylangmiles/daisy-tone-trixter) · the original Pico build: [github.com/dylangmiles/pico-tone-trixter](https://github.com/dylangmiles/pico-tone-trixter)