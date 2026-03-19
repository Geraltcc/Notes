采样
```
LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libprofiler.so CPUPROFILE=./my_prof.out ./test
```
结果：PDF
```
google-pprof --pdf ./your_program ./my_prof.out > profile.pdf
```
结果：火焰图
```
pprof -http=:8080 ./test ./my_prof.out
```
web
```
http://localhost:8080
```
