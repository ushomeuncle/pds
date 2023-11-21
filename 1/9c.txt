#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>
#define MASTER 0
int main(int argc, char *argv[]) {
	int numtasks, taskid;
	int value[5], recv[1];
	MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &numtasks);
	MPI_Comm_rank(MPI_COMM_WORLD, &taskid);
	if (taskid == MASTER) {
		printf("Value to be scattered:");
		int i;
		for (i = 0; i < 5; i++) {
			value[i] = i + 1;
			printf("%d ", value[i]);
		}
	}
	MPI_Scatter(value, 1, MPI_INT, recv, 1, MPI_INT, 0, MPI_COMM_WORLD);
	printf("After scattering:");
	printf("process %d value %d\n", taskid, recv[0]);
	MPI_Finalize();
}
