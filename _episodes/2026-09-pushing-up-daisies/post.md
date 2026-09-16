---
title: "Pushing Up Daisies"
subtitle: "The episode I skipped. Between the front-end fight and the copper, the whole computer changed — and every number with it."
episode: 7
date: 2026-09-16
permalink: /episodes/2026-09-pushing-up-daisies/
image:
  path: /assets/og/og-episode-7.jpg
  width: 1200
  height: 630
---

Episode 5 ended with one sentence about a new brain on the bench. Episode 6 opened as if it had always been there. Somewhere in between, the pedal stopped being a Raspberry Pi Pico with a codec module bolted on and became something else, and I never wrote that down — partly because it happened fast, and partly because the hum was a better story. But it was the biggest single decision in this project, and it touched every measurement I've ever put on this log. So: the missing episode.

![Two pedals side by side on a green cutting mat. On the left, a wooden gift box the size of a large cigar box, printed "dankie · THANK YOU · Enkosi kakhulu", with two footswitches and a small display window let into its lid and a hand-written "Tone Trixter" on red tape. On the right, a white plastic box about a quarter of the size, hand-lettered "Tone Trixie" in red, with two footswitches, a display window and a black knob. A steel rule runs along the front edge of the mat.](assets/two-brains.jpg)

---

## What the Pico Was Up Against

The first five episodes were built on a Pico 2 and a seven-dollar ES8388 codec module, and I want to be fair to that combination: it played an open mic. It ran a 512-tap impulse response in an interrupt handler and made a piezo sound like a microphone. Nothing about it was a mistake.

But it was a stack of workarounds, and I'd stopped noticing.

The Pico is a superb general-purpose microcontroller. It is not an audio module, and for a few specific reasons it was never going to win against something built to be one. Those reasons are worth naming, because none of them is "too slow."

The Pico has no audio hardware, so the codec was driven by bit-banging I²S with the chip's programmable I/O — which works, and which cost me most of April when the framing came out rotated by a few bit-clocks and no setting of the codec's registers would produce a clean sine. The fix involved parking a state machine on a particular clock edge and shifting every sample left by one bit to absorb the residual. I was proud of it. It was also the kind of thing that shouldn't have needed doing.

The converter itself had a floor of about −46 dBFS with the input shorted, on the breadboard — which sounds bad, and was, but the number that mattered was subtler: everything I did to the front end had to be judged against that floor, and by the rematch the buffer I was arguing about sat only three decibels above the noise of the things measuring it. Two whole episodes of careful A/B, with three decibels of room to see anything in.

Then there was the power. The Pico wanted 5 volts, so a 9-volt battery went through a small switching regulator, and a 9-volt alkaline sags under load to the point where that regulator drops out — in seconds. The pedal died on stage in my head several times before I worked out why it died on the bench. And a slip of a scope probe across the 5-volt rail, in May, killed a Pico outright.

Each of those got solved. The list of them is the point.

## The Seed

The Daisy Seed 3 is a small module from Electrosmith: an STM32H750 — a Cortex-M7 at 480 MHz — with 64 MB of SDRAM on the board, a USB-C port, and a real 32-bit audio codec whose datasheet noise floor is about thirty decibels below the ES8388's. It accepts anything from 4 to 16 volts directly, so the regulator and its failure mode disappear. It's thirty dollars.

I ordered three in mid-August, through a package forwarder, because Electrosmith doesn't sell through the catalogue distributors and the European warehouse was out of stock. Three, because there's no walking into a shop in Cape Town for a fourth.

What I was buying, really, was one thing: a converter quiet enough that the *front end* would become the limit. Everything the last two episodes had argued about — the JFET, the op-amp, the bias point, the 9-volt rail — would finally be the thing setting how quiet the pedal could be, instead of one contributor among several.

And what carried over was more than I'd expected. The op-amp buffer from Episode 5 went in ahead of the Daisy unchanged — the Seed's input is line-level and the pickup is a high-impedance piezo, so a buffer is still mandatory, and that one had just won its exam. The DSP chain ported almost line for line. The footswitches, the display, the encoder, the SD card: same parts, new pins.

## Two Boards

![The old pedal open, seen from above: a wooden box lined with copper foil, a small OLED and a knob in the lid's edge at the top, two panel jacks in the side walls, and in the middle a white plastic tray holding a perfboard with the Pico and the codec module under a dense loom of coloured jumper wires. Two footswitches at the bottom edge.](assets/v1-open.jpg)

![The new pedal open, seen from above, both halves of a white plastic box side by side on the cutting mat. The left half holds a perfboard with the Daisy Seed lying across it, two panel jacks along its top edge, a small blue microSD breakout at the top-left corner, and two red footswitches with short leads at the bottom. The right half is empty apart from a DC socket, with the battery leads crossing to it. A steel rule runs along the bottom edge.](assets/daisy-open.jpg)

Same job, both of them. The wooden box is the original; the white one is the Daisy build the day after it first played, before any copper went into it. A large part of the difference in size is the wiring: the Pico build grew by accretion, one module and one jumper at a time, and the Daisy board was laid out on a single 18-by-45-hole perfboard from a drawing before a part was placed.

The drawing mattered for one reason above others. The Seed's datasheet says its analogue and digital grounds are *not* joined on the module — you join them, once, outside it, or you get noise "or even damage." So the board has a ground star: one link between the two ground rails, at the point where power comes in. That single link was missing from the first version of the layout. I found it by reading the datasheet again, not by measuring, which is the cheaper way round.

Bring-up took an afternoon on the fifth of September, and the lesson from it is worth more than the board. "The menu is sluggish" survived four rounds of fixes, because it was three faults stacked: the library's quadrature decoder can't decode this encoder, so I wrote my own; the encoder was being read from the audio callback, too slowly to see a fast turn; and one of its legs had a dry joint from the build. Each fix helped a little and nothing settled it — and that pattern, where every fix helps slightly, turned out to be the signal to stop fixing and start measuring. What ended it was a mechanical test: pulling on the shaft changed the readings, and no software fault can do that.

The same afternoon produced a second class of problem: numbers that were lying. The microcontroller's C library omits floating-point printing unless you ask for it, so every `%f` on the console had been printing an empty string; and the microsecond clock wraps every twenty-one seconds, which turned every long timing into arithmetic fiction. Both fixed in a line. Both would have sent me chasing ghosts for a week if the bring-up had happened in a different order.

It played the next day. Guitar in, chain, IR, backing track, tuner, menu. Sixth of September.

## What Changed, Measured

![A scorecard of four rows, each with two horizontal bars: a hatched bar for the Pico V1 above a solid bar for the Daisy. Usable dynamic range: 51 dB against 96 dB. Latency in to out: 7.1 ms against 3.06 ms. IR length on one core: 512 taps against 2048 taps. Memory for loops: 0.5 MB against 64 MB. A footnote notes V1 reached 2048 taps only by splitting the IR across both of its cores.](assets/before-after.svg)

The dynamic range row is Episode 6's story — 51 to at least 96 — and I won't retell it, except to underline the part that belongs here: the electronics had been thirty decibels quieter from the first day the Seed ran. The copper only let me see it.

The latency row is new, and I like the method more than the number.

Every latency figure I'd quoted before came from scope cursors: a burst in on one channel, the burst out on the other, read the offset by eye. Good to a few hundred microseconds. This time the audio interface did the work. It played a click out of its monitor output, and that click went two ways at once — straight back into one input, and through the pedal into the other. Both inputs are sampled by the same converter clock, so the pedal's delay is simply the number of samples between the two recordings of the same click. Twenty clicks, cross-correlated:

![Two traces of a single click, a few milliseconds wide. The dashed trace, labelled "direct — the interface hearing the click itself", is a sharp spike at 0 ms with a short ring. The solid trace, labelled "through the pedal, bypass — same click, same clock", is the same shape starting at 3.06 ms. A double-headed arrow between the two onsets is marked "147 samples = 3.06 ms" and, beneath it, "two 64-sample blocks + the converter's 19".](assets/latency.svg)

**147 samples. 3.06 milliseconds.** All twenty clicks read the same to the sample — no jitter at all — and, more usefully, it read the same in every mode I could put the pedal into: bypass, the compressor, the 2048-tap impulse response, and song mode with the looper recording underneath. The IR and the looper both do their work inside the same audio callback, so neither costs a block. The old pedal was 7.1 ms; the new one is 147 samples, which is two 64-sample blocks plus nineteen samples of the converter's own filters, and there is nothing hiding anywhere.

The IR row is where the extra horsepower shows plainly. The Pico ran 512 taps on one core and needed its second core to reach 2048. The Daisy runs 2048 inline at 16 % average and a 44 % peak — and with eight looper layers summing on top of it and a metronome ticking, 48 %. Half the machine is still spare at the heaviest thing the pedal can currently do.

The memory row is the one that changed what the pedal *is*. Sixty-four megabytes is eight layers of sixty seconds each, held as plain audio, which is how there's now a looper in it — arm a layer with the foot, play, and the loop starts on the first note and ends one beat after the last one. That didn't exist as a possibility on 520 kilobytes. Nor did leaving the SD card in the box: hold the encoder while plugging in the USB and the Daisy mounts as a disk, so presets, impulse responses and backing tracks get edited on the laptop and the card never comes out. I mention both not because they're the subject of this episode but because they fell out of a decision made for the noise floor.

## What Didn't Change

The front end. Every measurement above was through the same op-amp buffer that won Episode 5, on the same 9 volts, unchanged. The Seed made it *visible*; it didn't make it better.

And one thing I'd believed since May turned out not to be about either board. The pedal had always seemed to lose a little bass — I'd measured a roll-off around 200 Hz on the old build and filed it as a coupling-capacitor problem to fix later. This week, with the two-channel rig, I could separate the pedal from the test fixture for the first time, and the pedal alone is flat to below 40 Hz. The roll-off was the *bench dongle*: the little box that mimics a piezo's source capacitance was built with a one-nanofarad capacitor, and the actual pickup on the guitar, when I finally put a meter across it, reads twenty-four. Every bass figure I'd ever taken through that dongle was measuring the dongle. The pickup into the pedal corners at about 6 Hz. There was no shoulder.

I'm including that because it's the same lesson as the hum, from the other side: the instrument you measure with is part of the circuit.

## What It Cost

A month, roughly — from the order on the twelfth of August to a playing pedal on the sixth of September — and most of it was the board, not the code. A new pin map, a new drawing, a perfboard cut to size, a ground star that wasn't in the first draft.

Three Seeds instead of one, via Florida.

And a new set of library quirks in place of the old ones. The Pico's were about bit-banging a codec; the Daisy's are about a large vendor library that does most things and a few things wrong. The encoder decoder. The silent floating-point printing. And one that took a bisection with a serial console to find: the USB port died every time the processor went to sleep between audio blocks, because a clock the USB core needs is gated in sleep by default and the library doesn't ungate it. One line, once found. A day, finding it.

None of which changes the sum. The old pedal was a stack of workarounds that played. The new one is a board that does what the datasheet says and measures where the instruments run out.

![The two pedals from the front, jack rows facing the camera. The wooden box on the left has a red "Tone Trixter" label on its front face and an "IN" tag by a single jack. The white box on the right has three jacks along its front edge labelled by hand "IN", "POWER" and "OUT", with the footswitches and knob on the face above.](assets/jack-rows.jpg)

The white one has its own name on it now, which is probably the honest way to say what this episode is about: it isn't the same pedal with a faster chip. It's the pedal the first one was working out how to be.

The next board is the one I've been putting off since April: a printed circuit board, with a ground plane, designed for the two problems that are actually left — a faint tick from the display bus once a second, and the path the battery current takes home. Small problems. It took changing brains to make them the biggest ones.

---

Source, schematics, and firmware: [github.com/dylangmiles/daisy-tone-trixter](https://github.com/dylangmiles/daisy-tone-trixter)
