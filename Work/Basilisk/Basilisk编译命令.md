```
qcc -autolink -g -Wall -Wno-unused-function -pipe -D_FORTIFY_SOURCE=2 -O3 -fopenmp -o test demo2.c -L$BASILISK/gl -I$BASILISK -lglutils -lfb_osmesa -lGLU -lOSMesa -lm

qcc -autolink -g -Wall -Wno-unused-function -pipe -D_FORTIFY_SOURCE=2 -O3 -fopenmp -o test flat-test.c -L$BASILISK/gl -I$BASILISK -lglutils -lfb_osmesa -lGLU -lOSMesa -lm

CC99='mpicc -std=c99' qcc -disable-dimensions -autolink -Wall -Wno-unused-function -pipe -D_FORTIFY_SOURCE=2 -O3  -D_MPI=1 -o test 2dtest.c -L$BASILISK/gl -I$BASILISK -lglutils -lm

CC99='mpicc -std=c99 -D_GNU_SOURCE=1 -g' qcc -g -autolink -Wall -Wno-unused-function -pipe -D_FORTIFY_SOURCE=2 -O3 -D_MPI=1 -o test test.c -L$BASILISK/gl -I$BASILISK -lglutils -lfb_osmesa -lGLU -lOSMesa -lm

mpicc -Wall -std=c99 

mpirun -np 1 ./test2

tar -zxvf 解压

qcc -O2 -Wall bump.c -o bump -lm
```
