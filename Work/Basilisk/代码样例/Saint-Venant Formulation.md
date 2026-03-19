```
//#include "grid/cartesian.h"
#include "saint-venant.h"

#define LEVEL 8

event init (t = 0) {
  double a = 1., b = 200;
  foreach()
    h[] = 0.1 + a * exp(- b * (x * x + y * y));
}

event graphs (i++) {
  stats s = statsf (h);
  fprintf (stderr, "%g %g %g\n", t, s.min, s.max);
}

event images (t += 4. / 300) {
  output_ppm (h);
  scalar l[];
  foreach()
    l[] = level;
    static FILE * fp = fopen("grid.ppm", "w");
    output_ppm (l, fp, min =  0, max = 8);
}

event end(t = 4) {
  printf("i = %d t = %g\n", i, t);
}

event adapt (i++) {
  adapt_wavelet ({h}, (double []){4e-3}, maxlevel = LEVEL);
}

int main() {
  origin (-0.5, -0.5);
  init_grid(1 << LEVEL);
  run();
}
```
编译部分：
```
qcc -O2 -Wall test1.c -o test1 -lm
./test1 > out.ppm 2 >log
```
后处理：
```
animate out.ppm   //ImageMagick方法
ffmpeg -framerate 30 -i grid.ppm -c:v libx264 -pix_fmt yuv420p output.mp4   //ffmpeg输出mp4方法

gnuplot   //进入gnuplot对话
>set xlabel 'Time'
>set ylabel 'Depth'
>plot 'log' using 1:2 with lines title 'min' \
      'log' using 1:3 with lines title 'max'
```
输出结果：
![[output.mp4]]![[237ac12f5c3a13f4dafb8d687f30da08.mp4]]