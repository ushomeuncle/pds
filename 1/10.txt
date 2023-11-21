#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>
#define NUM_MACHINES 5
#define TIME_DAEMON 0
int main(int argc, char **argv) {
	int rank, size;
	int i;
	MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &size);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	if (size != NUM_MACHINES) {
		printf("This program should be run with %d processes.\n", NUM_MACHINES);
		MPI_Finalize();
		return 1;
	}
	srand(time(NULL) + rank);
	int local_time = rand() % 100;
	if (rank == TIME_DAEMON) {
		int total_time = local_time;
		for (i = 1; i < NUM_MACHINES; i++) {
			int machine_time;
			MPI_Recv(&machine_time, 1, MPI_INT, i, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
			total_time += machine_time;
		}
		int avg_time = total_time / NUM_MACHINES;
		for (i = 1; i < NUM_MACHINES; i++) {
			MPI_Send(&avg_time, 1, MPI_INT, i, 0, MPI_COMM_WORLD);
		}
		printf("Time daemon synchronized clocks. Original time: %d, Average time: %d\n", local_time, avg_time);
	}
	else {
		printf("Machine %d's original time: %d\n", rank, local_time);
		MPI_Send(&local_time, 1, MPI_INT, TIME_DAEMON, 0, MPI_COMM_WORLD);
		int avg_time;
		MPI_Recv(&avg_time, 1, MPI_INT, TIME_DAEMON, 0, MPI_COMM_WORLD,
		MPI_STATUS_IGNORE);
		local_time = avg_time;
		printf("Machine %d synchronized its clock. New time: %d\n", rank, local_time);
	}
	MPI_Finalize();
	return 0;
}

