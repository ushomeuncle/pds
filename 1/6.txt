#include <stdio.h>
#include<cuda.h>
#include <stdlib.h>
#include <time.h>
void printArr(int arr[],int n)
{
int i;
for(i=0;i<n;i++)
printf("%d ",arr[i]);
}
__device__ int d_size;
__global__ void partition(int *arr,int *larr,int *harr,int n)
{
int z = blockIdx.x * blockDim.x + threadIdx.x;
d_size = 0;
__syncthreads();
if(z<n)
{
int h = harr[z];
int l = larr[z];
int x = arr[h];
int i = l-1;
int temp;
for(int j=l;j<=h-1;j++)
{
if(arr[j]<=x)
{
i++;
temp = arr[i];
arr[i] = arr[j];
arr[j] = temp;
}
}
temp = arr[i+1];
arr[i+1] = arr[h];
arr[h] = temp;
int p = i+1;
if(p-1>l)
{
int ind = atomicAdd(&d_size,1);
larr[ind] = l;
harr[ind] = p-1;
}
if(p+1<h)
{
int ind = atomicAdd(&d_size,1);
larr[ind] = p+1;
harr[ind] = h;
}
}
}
void quickSort(int arr[],int l,int h)
{
int lstack[h-l+1],hstack[h-l+1];
int top = -1,*d_d,*d_l,*d_h;
lstack[++top] = l;
hstack[top] = h;
cudaMalloc(&d_d,(h-l+1)*sizeof(int));
cudaMemcpy(d_d,arr,(h-l+1)*sizeof(int),cudaMemcpyHostToDevice);
cudaMalloc(&d_l, (h-l+1)*sizeof(int));
cudaMemcpy(d_l, lstack,(h-l+1)*sizeof(int),cudaMemcpyHostToDevice);
cudaMalloc(&d_h, (h-l+1)*sizeof(int));
cudaMemcpy(d_h, hstack,(h-l+1)*sizeof(int),cudaMemcpyHostToDevice);
int n_t = 1,n_b = 1,n_i = 1;
while(n_i>0)
{
partition<<<n_b,n_t>>>( d_d, d_l, d_h, n_i);
int answer;
cudaMemcpyFromSymbol(&answer, d_size, sizeof(int), 0,cudaMemcpyDeviceToHost);
if (answer < 1024)
{
n_t = answer;
}
else
{
n_t = 1024;
n_b = answer/n_t + (answer%n_t==0?0:1);
}
n_i = answer;
cudaMemcpy(arr, d_d,(h-l+ 1)*sizeof(int),cudaMemcpyDeviceToHost);
}
}
int main()
{
int arr[]={0,9,1,8,2,7,3,4,6,5};
int n = sizeof( arr ) / sizeof( *arr );
printf("The input array:\n");
printArr( arr, n );
quickSort( arr, 0, n - 1 );
printf("\nThe output array:\n");
printArr( arr, n );
printf("\n");
return 0;
}