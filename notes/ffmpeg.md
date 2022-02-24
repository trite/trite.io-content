# Unorganized Snippets
 * ffmpeg -ss 04:15 -to 04:20 -i pharah-kill-as-ashe.flv -vf "fps=10,scale=720:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" pharah-kill-as-ashe.gif
 * ffmpeg -ss 2:00:00 -i hanzo-moira-ball.flv hanzo-moira-ball.flv
 * ffmpeg -ss 01:04 -to 01:09 -i hanzo-moira-ball.flv -vf "fps=10,scale=640:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" hanzo-moira-ball.gif
 * ffmpeg -ss 02:09:44 -to 02:09:54 -i nova-drift-chaos.mp4 -c:v libvpx-vp9 -b:v 2M -pass 1 -an -f null /dev/null && ffmpeg -ss 02:09:44 -to 02:09:54 -i nova-drift-chaos.mp4 -c:v libvpx-vp9 -b:v 2M -pass 2 -c:a libopus nova-drift-chaos.webm
 * ffmpeg -ss 37:48 -to 37:56 -i duuce-triple-snipe-end.flv -vf "fps=10,scale=720:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" duuce-triple-snipe-end.gif
 * ffmpeg -ss 37:48 -to 37:56 -i duuce-triple-snipe-end.flv -vf "fps=10,scale=640:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" duuce-triple-snipe-end.gif


# 2 stage method for speeding up flv footage and converting to gif:
Should be a better way, will figure it out later. For now this works:
```bash
ffmpeg -ss 00:14 -i spiro-examples-4.flv -vf "setpts=0.005*PTS" spiro-examples-4.mp4
ffmpeg -i spiro-examples-4.mp4 -loop 0 -vf "fps=15,scale=720:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" spiro-examples-04.gif
```