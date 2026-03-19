```
# 转换视频
ffmpeg -framerate 30 -i f_%04d.png -c:v libx264 -pix_fmt yuv420p output.mp4

ffmpeg -framerate 120 -i ux_%04d.png -c:v libx264 -pix_fmt yuv420p k.mp4

ffmpeg -framerate 100 -pattern_type glob -i "ux_*.png" output.mp4

ffmpeg -framerate 10 -pattern_type glob -i 'output/rho_*.png' -c:v libx264 -pix_fmt yuv420p test.mp4
```