#include <stdio.h>
#include <mpi.h>
#include <stdlib.h>
#define MASTER 0
int main(int argc, char *argv[]) {
	int numtasks, length, taskid;
	char hostname[MPI_MAX_PROCESSOR_NAME];
	int *recv, i;
	MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &numtasks);
	MPI_Comm_rank(MPI_COMM_WORLD, &taskid);
	MPI_Get_processor_name(hostname, &length);
	i = taskid + 1;
	printf("Before gathering:");
	printf("process %d value %d\n", taskid, i);
	if (taskid == MASTER) {
		recv = (int *)malloc(numtasks * sizeof(int));
	}
	MPI_Gather(&i, 1, MPI_INT, recv, 1, MPI_INT, MASTER, MPI_COMM_WORLD);
	if (taskid == MASTER) {
		printf("After gathering:");
		printf("process %d", taskid);
		int j;
		for (j = 0; j < numtasks; j++) {
			printf("%d ", recv[j]);
		}
	}
	MPI_Finalize();
	free(recv);
}
