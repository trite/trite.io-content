# Unorganized Snippets
 * ffmpeg -ss 04:15 -to 04:20 -i pharah-kill-as-ashe.flv -vf "fps=10,scale=720:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" pharah-kill-as-ashe.gif
 * ffmpeg -ss 2:00:00 -i hanzo-moira-ball.flv hanzo-moira-ball.flv
 * ffmpeg -ss 01:04 -to 01:09 -i hanzo-moira-ball.flv -vf "fps=10,scale=640:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" hanzo-moira-ball.gif
 * ffmpeg -ss 02:09:44 -to 02:09:54 -i nova-drift-chaos.mp4 -c:v libvpx-vp9 -b:v 2M -pass 1 -an -f null /dev/null && ffmpeg -ss 02:09:44 -to 02:09:54 -i nova-drift-chaos.mp4 -c:v libvpx-vp9 -b:v 2M -pass 2 -c:a libopus nova-drift-chaos.webm
 * ffmpeg -ss 37:48 -to 37:56 -i duuce-triple-snipe-end.flv -vf "fps=10,scale=720:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" duuce-triple-snipe-end.gif
 * ffmpeg -ss 37:48 -to 37:56 -i duuce-triple-snipe-end.flv -vf "fps=10,scale=640:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" duuce-triple-snipe-end.gif