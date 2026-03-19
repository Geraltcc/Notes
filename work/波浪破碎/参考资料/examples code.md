## 1.头文件
```C
#include "navier-stokes/centered.h"     
#include "two-phase.h"
#include "navier-stokes/conserving.h"   // stablility
#include "tension.h"
#include "reduced.h"
#include "view.h"
#include "tag.h"
//#include "navier-stokes/perfs.h"
//#include "profiling.h"
```
## 2.初始化变量
	wave steepness  $ak$
	Bond number     $BO$
	Reynolds number $RE$
```C
double ak = 0.55;
double BO = 1000.;
double RE = 40000.;
```
## 3.加密层数+加密条件
```C
int LEVEL = dimension == 2 ? 9 : 6;
double uemax = 0.005;
```
## 4.水气密度与粘性
```C
#define RATIO (1./850.)
#define MURATIO (17.4e-6/8.9e-4)
```
## 5.stokes波形参数
	wave number $k\_$
	fluid depth $h\_$
	gravity     $g\_$
	Dirac viscous 
```C
#define k_ (2.*pi)
#define h_ 0.5
#define g_ 1.
int DIRAC = 0.;
```
## 6.主函数
```C
int main(int argc, char* argv[])
{
	if (argc > 1)
		LEVEL = atoi (argv[1]);
	else if (argc > 2)
		ak = atof (argv[2]);
	else if (argc > 3)
		BO = atof (argv[3]);
	else if (argc > 4)
		RE = atof (argv[4]);
	else if (argc > 5)
		DIRAC = atof (argv[5]);
	
	// domain setting
	L0 = 1. [0];
	DT = HUGE [0];
	origin (-L0/2., -L0/2., -L0/2.);

	// periodic setting
	periodic (right);
#if dimension > 2
	periodic (front);
#endif

	// property setting
	rho1 = 1.;    // water
	rho2 = RATIO; // air
	mu1 = 1.0/RE;
	mu2 = 1.0/RE*MURATIO;
	f.sigma = 1./(BO*sq(k_));
	G.y = -g_;

	// grid setting
	N = 1 << 6;
	init_grid (N);

	// main loop
	run();
}
```
## 7.stokes波形解
```C
double wave (double x, double y) {
	double a_ = ak / k_;
	double eta1 = a * cos(k_ * x);
	double alpa = 1. / tanh(k_ * h_);
	double eta2 = 1./4. * alpa * (3.*sq(alpa)-1.) * sq(a_) * k_ * cos(2. * k_ * x);
	double eta3 = -3./8.*(cube(alpa)*alpa - 3.*sq(alpa) + 3.)*cube(a_)*sq(k_)*cos(k_*x) + 3./64.*(8.*cube(alpa)*cube(alpa) + (sq(alpa) - 1.)*(sq(alpa) - 1.))*cube(a_)*sq(k_)*cos(3.*k_*x);
	return eta1 + ak*eta2 + sq(ak)*eta3 - y;
}

double eta (double x, double y) {
	double a_ = ak/k_;
	double eta1 = a_*cos(k_*x);
	double alpa = 1./tanh(k_*h_);
	double eta2 = 1./4.*alpa*(3.*sq(alpa) - 1.)*sq(a_)*k_*cos(2.*k_*x);
	double eta3 = -3./8.*(cube(alpa)*alpa - 3.*sq(alpa) + 3.)*cube(a_)*sq(k_)*cos(k_*x) + 3./64.*(8.*cube(alpa)*cube(alpa) + (sq(alpa) - 1.)*(sq(alpa) - 1.))*cube(a_)*sq(k_)*cos(3.*k_*x);
	return eta1 + ak*eta2 + sq(ak)*eta3;
}

double gaus (double y, double yc, double T) {
	double deltaw = sqrt(2.0/RE)/k_;
	double deltaa = sqrt(2.0/RE*MURATIO/RATIO)/k_;
	double r = y - yc;
	return 2.0/(sqrt(2.0*pi*sq(deltaa)) + sqrt(2.0*pi*sq(deltaw))) * (T*exp(-sq(r)/(2.0*sq(deltaw))) + (1.0 - T)*exp(-sq(r)/(2.0*sq(deltaa))));
}
```
## 8.初始化
```C
event init (i = 0)
{
	if (!restore("restart")) {
		do {
			fraction (f, wave(x, y));
			scalar Phi[];
			foreach() {
				double alpa = 1./tanh(k_*h_);
				double a_   = ak/k_;
				double sgma = sqrt(g_*k_*tanh(k_*h_)*(1. + k_*k_*a_*a_*(9./8.*(sq(alpa) - 1.)*(sq(alpa) - 1.) + sq(alpa))));
				double A_   = a_*g_/sgma;
				double phi1 = A_*cosh(k_*(y + h_))/cosh(k_*h_)*sin(k_*x);
				double phi2 = 3.*ak*A_/(8.*alpa)*(sq(alpa) - 1.)*(sq(alpa) - 1.)*cosh(2.0*k_*(y + h_))*sin(2.0*k_*x)/cosh(2.0*k_*h_);
				double phi3 = 1./64.*(sq(alpa) - 1.)*(sq(alpa) + 3.)*(9.*sq(alpa) - 13.)*cosh(3.*k_*(y + h_))/cosh(3.*k_*h_)*a_*a_*k_*k_*A_*sin(3.*k_*x);
				Phi[] = phi1 + ak*phi2 + sq(ak)*phi3;
			}
		    boundary ({Phi});

			if (DIRAC) {
				scalar vort2[];
				scalar psi[];
				foreach() {
					vort2[] = -2.0*gaus(y,wave(x,y)+y,f[])*(Phi[1,0]-Phi[-1,0])/(2.*Delta);
					psi[] = 0.0;
				}
				boundary({vort2, psi});
				psi[top] = dirichlet(0.);
				psi[bottom] = dirichlet(0.);

				poisson (psi, vort2);

				foreach() {
					u.x[] =  (psi[0,1] - psi[0,-1])/(2.*Delta);
					u.y[] = -(psi[1] - psi[-1])/(2.*Delta); 
				}
			} else {
				foreach()
					foreach_dimension()
						u.x[] = (Phi[1] - Phi[-1])/(2.*Delta) * f[];
			}
			boundary ((scalar *){u});			
		} 
#if TREE
		while (adapt_wavelet ({f,u}, (double[]){0.01, uemax, uemax, uemax, uemax}, LEVEL, 5).nf);
#endif
	}
}
```
## 9.后处理
```C
// 耗散率
int dissipation_rate (double* rates)
{
	double rateWater = 0.0;
	double rateAor   = 0.0;
	foreach(reduction (+:rateWater) reduction (+:rateAir)) {
		double dudx = (u.x[1] - u.x[-1])/(2.*Delta);
		double dudy = (u.x[0,1] - u.x[0,-1])/(2.*Delta);
		double dudz = (u.x[0,0,1] - u.x[0,0,-1])/(2.*Delta);
		double dvdx = (u.y[1] - u.y[-1])/(2.*Delta);
		double dvdy = (u.y[0,1] - u.y[0,-1])/(2.*Delta);
		double dvdz = (u.y[0,0,1] - u.y[0,0,-1])/(2.*Delta);
		double dwdx = (u.z[1] - u.z[-1])/(2.*Delta);
		double dwdy = (u.z[0,1] - u.z[0,-1])/(2.*Delta);
		double dwdz = (u.z[0,0,1] - u.z[0,0,-1])/(2.*Delta);   
		// 冗长，可以使用foreach() foreacn_dimension张量代替
		double SDeformxx = dudx;
		double SDeformxy = 0.5 * (dudy + dvdx);
		double SDeformxz = 0.5 * (dudz + dwdx);
		double SDeformyx = SDeformxy;
		double SDeformyy = dvdy;
		double SDeformyz = 0.5 * (dvdz + dwdy);
		double SDeformzx = SDeformxz;
		double SDeformzy = SDeformyz;
		double SDeformzz = dwdz;
		// #define dv() (Delta*cm[])) for 1 dimension
		// #define dv() (sq(Delta)*cm[]) for 2 dimension
		double sqterm = 2.*dv()*(sq(SDeformxx) + sq(SDeformxy) + sq(SDeformxz) + sq(SDeformyx) + sq(SDeformyy) + sq(SDeformyz) + sq(SDeformzx) + sq(SDeformzy) + sq(SDeformzz));
		rateWater += mu1/rho[]*f[]*sqterm; // water
		rateAir   += mu2/rho[]*(1.-f[])*sqterm;  //air
	}
	rates[0] = rateWater;
	rates[1] = rateAir;

	return 0;
}

// 统计破碎液滴和气泡数
#include "tag.h"
event countDropsBubble (i++)
{
	scalar m1[]; // droplets
	scalar m2[]; // bubbles
	foreach() {
		m1[] = f[] > 0.5; // set m true if f[] is close to unity (droplets)
		m2[] = f[] < 0.5; // set m true if f[] is close to zero (bubbles)
	}
	int n1 = tag (m1);
	int n2 = tag (m2);

	double v1[n1]; // droplets
	coord b1[n1]; // droplets

	double v2[n2]; // bubbles
	coord b2[n2]; // bubbles

	// initialize
	for (int j = 0; j < n1; j++)
		v1[j] = b1[j].x = b1[j].y = b1[j].z = 0.0;

	for (int j = 0; j < n2; j++)
		v2[j] = b2[j].x = b2[j].y = b2[j].z = 0.0;

	// calculation
	foreach_leaf() {
		// droplets
		if (m1[] > 0) {
			int j = m1[] - 1;
			v1[j] += dv() * f[];   // increment the volume of the droplet
			coord p = {x, y, z};
			foreach_dimension()
				b1[j].x += dv() * f[] * p.x;
		}

		// bubbles
		if (m2[] > 0) {
			int j = m2[] - 1;
			v2[j] += dv() * (1. - f[]);
			coord p = {x, y, z};
			foreach_dimension()
				b2[j].x += dv() * (1.0 - f[]) * p.x;
		}
	}
#if _MPI
	MPI_Allreduce (MPI_IN_PLACE, v1, n1, MPI_DOUBLE, MPI_SUM, MPI_COMM_WORLD);
	MPI_Allreduce (MPI_IN_PLACE, b1, 3*n1, MPI_DOUBLE, MPI_SUM, MPI_COMM_WORLD);
	MPI_Allreduce (MPI_IN_PLACE, v2, n2, MPI_DOUBLE, MPI_SUM, MPI_COMM_WORLD);
	MPI_Allreduce (MPI_IN_PLACE, b2, 3*n2, MPI_DOUBLE, MPI_SUM, MPI_COMM_WORLD);
#endif

	static FILE * fdrop = fopen("droplets.dat","w");
	static FILE * fbubb = fopen("bubbles.dat","w");

	for (int j=0; j<n1; j++)
	    fprintf (fdrop, "%d %g %d %g %g %g\n", i, t, j, v1[j], b1[j].x/v1[j], b1[j].y/v1[j]);
	for (int j=0; j<n2; j++)
	    fprintf (fbubb, "%d %g %d %g %g %g\n", i, t, j, v2[j], b2[j].x/v2[j], b2[j].y/v2[j]);
}
```