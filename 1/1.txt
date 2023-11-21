#include<stdio.h>
#include<cuda.h>
#define N 8
_global_ void dot_product(int *d_a,int *d_b,int *d_c){
int i=threadIdx.x;
if(i<N)
d_c[i]=d_a[i]*d_b[i];
__syncthreads();
for(int s=1;s<blockDim.x;s*=2)
{
int in=2*s*i;
if(in<blockDim.x)
{
d_c[in]=d_c[in]+d[in+s];
}
__syncthreads();
}
}
int main(void)
{
int a[N],b[N],c[N];
int *da,*db,*dc;
int size=N*sizeof(int);
cudaMalloc((void **)&da,size);
cudaMalloc((void **)&db,size);
cudaMalloc((void **)&dc,size);
for(int i=0;i<N;i++)
a[i]=i;
for(int i=0;i<N;i++)
b[i]=i;
cudaMemcpy(da,a,size,cudaMemcpyHostToDevice);
cudaMemcpy(db,b,size,cudaMemcpyHostToDevice);
dot_product<<<1,N>>>(da,db,dc);
cudaMemcpy(c,dc,size,cudaMemcpyDeviceToHost);
for(int i=0;i<N;i++)
printf("\n%d",c[i]);
return 0;
}
