#include <stdio.h>
#define RADIUS 2
#define ARRAY_SIZE 8
#define BLOCK_SIZE 128
__constant__ int coef[5] = {1, 1, 1, 1, 1}; // Updated to have 5 elements
__global__ void stencil_1d(int *in, int *out, int size)
{
int tid = threadIdx.x + blockIdx.x * blockDim.x;
if (tid < size)
{
int result = 0;
for (int i = -RADIUS; i <= RADIUS; i++)
{
int idx = tid + i;
if (idx >= 0 && idx < size)
{
result += coef[i + RADIUS] * in[idx];
}
}
out[tid] = result;
}
}
int main() {
int h_input[ARRAY_SIZE] = {5, 2, 1, 9, 2, 3, 6, 1};
int h_output[ARRAY_SIZE];
int *d_input, *d_output;
cudaMalloc((void**)&d_input, ARRAY_SIZE * sizeof(int));
cudaMalloc((void**)&d_output, ARRAY_SIZE * sizeof(int));
cudaMemcpy(d_input, h_input, ARRAY_SIZE * sizeof(int), cudaMemcpyHostToDevice);
int numBlocks = (ARRAY_SIZE + BLOCK_SIZE - 1) / BLOCK_SIZE;
stencil_1d<<<numBlocks, BLOCK_SIZE>>>(d_input, d_output, ARRAY_SIZE);
cudaMemcpy(h_output, d_output, ARRAY_SIZE * sizeof(int), cudaMemcpyDeviceToHost);
cudaFree(d_input);
cudaFree(d_output);
for (int i = 0; i < ARRAY_SIZE; i++) {
printf("%d ", h_output[i]);
}
printf("\n");
return 0;
}