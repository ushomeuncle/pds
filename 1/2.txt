#include <stdio.h>
#include <cuda.h>
const int N = 4; 
const int block_size = 2; // Block size for CUDA threads
__global__ void matrixTranspose(int* input, int* output) {
__shared__ int shared_data[block_size][block_size];
int x = blockIdx.x * block_size + threadIdx.x;
int y = blockIdx.y * block_size + threadIdx.y;

shared_data[threadIdx.y][threadIdx.x] = input[y * N + x];
__syncthreads();

x = blockIdx.y * block_size + threadIdx.x;
y = blockIdx.x * block_size + threadIdx.y;
output[y * N + x] = shared_data[threadIdx.x][threadIdx.y];
}
int main() {
int input_matrix[N * N] = {
1, 2, 3, 4,
5, 6, 7, 8,
9, 10, 11, 12,
13, 14, 15, 16
};
int output_matrix[N * N];
int* d_input_matrix;
int* d_output_matrix;

cudaMalloc((void**)&d_input_matrix, N * N * sizeof(int));
cudaMalloc((void**)&d_output_matrix, N * N * sizeof(int));

cudaMemcpy(d_input_matrix, input_matrix, N * N * sizeof(int), cudaMemcpyHostToDevice);

dim3 grid(N / block_size, N / block_size);
dim3 block(block_size, block_size);

matrixTranspose<<<grid, block>>>(d_input_matrix, d_output_matrix);

cudaMemcpy(output_matrix, d_output_matrix, N * N * sizeof(int), cudaMemcpyDeviceToHost);
printf("Input Matrix:\n");
for (int i = 0; i < N * N; ++i) {
printf("%d\t",input_matrix[i]);
if ((i + 1) % N == 0)
printf("\n");
}
printf("\nOutput Matrix (Transpose):\n");
for (int i = 0; i < N * N; ++i) {
printf("%d\t",output_matrix[i]);
if ((i + 1) % N == 0)
printf("\n");
}

cudaFree(d_input_matrix);
cudaFree(d_output_matrix);
return 0;
}
