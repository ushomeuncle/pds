#include <stdio.h>
#include <mpi.h>
#define MASTER 0
int main(int argc, char *argv[]) {
	int taskid, numtasks, value;
	MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &numtasks);
	MPI_Comm_rank(MPI_COMM_WORLD, &taskid);
	printf("Before Broadcasting: ");
	printf("Process %d : value %d\n", taskid, value);
	if (taskid == MASTER) {
		value = 100;
	}
	MPI_Bcast(&value, 1, MPI_INT, MASTER, MPI_COMM_WORLD);
	printf("After Broadcasting :");
	printf("Process %d : value %d\n", taskid, value);
	MPI_Finalize();
}

