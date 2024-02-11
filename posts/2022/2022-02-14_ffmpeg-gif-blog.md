---
layout: post
title: FFmpeg and GIF-ifying Videos
created: 2022-02-14
---

Needed to convert some video files to a format that would play nicely with blog posting. I'm also taking my sweet time designing the site, so these posts are likely to just live in github for a good bit.

# Options
There are quite a few tools out there for converting videos, and a great many aim to be quite user friendly. Unfortunately using any site to convert files has a few problems:
 * Don't know what happens to files once uploaded - not a huge worry here, but could be in the future.
 * Bandwidth/time costs - one of the clips here is only 5 seconds, but the original footage is over 2 hours. Trimming footage is likely going to be necessary before uploading.
 * **Harder to automate** <- Biggest factor for me.

Using a tool like [FFmpeg](https://www.ffmpeg.org/) sounds much better.

# Installing FFmpeg
Using an Ubuntu WSL instance for this, install is as easy as expected:

```bash
$ sudo apt update
$ sudo apt install ffmpeg
$ ffmpeg
ffmpeg version 4.2.4-1ubuntu0.1 Copyright (c) 2000-2020 the FFmpeg developers
```

Version 5 came out a few weeks ago, but 4.x will be fine for this use case.

# Methods/Resources
Not going into depth on this here, but these sources have some great info worth reading through (imo):
 * Command examples and a few notes: https://superuser.com/questions/556029/how-do-i-convert-a-video-to-gif-using-ffmpeg-with-reasonable-quality
 * Great getting started info: https://engineering.giphy.com/how-to-make-gifs-with-ffmpeg/
 * Deep dive into some of the issues with gif encoding, focusing on color since gifs have some significant limitaitons regarding color: http://blog.pkh.me/p/21-high-quality-gif-with-ffmpeg.html

The solution on the `superuser.com` page seems to generate the same file as when using the 2-pass method described in the `blog.phk.me` link above. As such here's that script slightly modified for this case:

```bash
#!/bin/sh

palette="/tmp/palette.png"

filters="fps=10,scale=320:-1:flags=lanczos"

start_time=02:09:44
end_time=02:09:49

ffmpeg -ss $start_time -to $end_time -i "$1" -vf "$filters,palettegen" -y $palette
ffmpeg -ss $start_time -to $end_time -i "$1" -i $palette -lavfi "$filters [x]; [x][1:v] paletteuse" -y "$2"
```

Which is then run like so:
```bash
$ ./transcoding.sh nova-drift-chaos.mp4 nova-drift-chaos-dual-pass.gif
```

Next it is time to do the single-pass version, again modified for our case:

```bash
$ ffmpeg -ss 02:09:44 -to 02:09:49 -i nova-drift-chaos.mp4 -vf "fps=10,scale=320:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" nova-drift-chaos-single-pass.gif
```

The 2 resulting files have the exact same filesize and the `cmp` command finds no difference between them:

```bash
$ ls -la *pass.gif
-rwxrwxrwx 1 trite trite 1583169 Feb 15 10:32 nova-drift-chaos-dual-pass.gif
-rwxrwxrwx 1 trite trite 1583169 Feb 15 10:29 nova-drift-chaos-single-pass.gif
$ cmp nova-drift-chaos-dual-pass.gif output.gif
nova-drift-chaos-dual-pass.gif output.gif differ: byte 17, line 1
$ cmp nova-drift-chaos-dual-pass.gif nova-drift-chaos-single-pass.gif
$
```

# Result
Might end up working with FFmpeg more frequently. If so I want to spend some more time picking apart the filter options. In the meantime this is probably going to see a lot of use:

```
ffmpeg -ss {start_time} -to {end_time} -i {input_file} -vf "fps=10,scale=320:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" {output_file}
```

Some notes on the timestamps: `-ss [timestamp]` and `-to [timestamp]` are position-specific flags to provide a starting and ending point in footage
 * If placed before `-i` these affect how much input footage is processed, if placed after they simply affect which parts are written to disk.
 * Syntax options for the timestamps can be found here: http://ffmpeg.org/ffmpeg-utils.html#time-duration-syntax - will be using `HH:MM:SS` in this case.