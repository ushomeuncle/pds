#include <stdio.h>
#include <mpi.h>
#define N 3
int main(int argc, char** argv) {
	int rank, timestamp = 0;
	int i;
	int round = 1;
	int request[N] = {0};
	int reply[N] = {0};
	int in_critical_section = 0;
	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	while (round > 0) {
		timestamp++;
		request[rank] = timestamp;
		for (i = 0; i < N; i++) {
			if (i != rank) {
				MPI_Send(&request[rank], 1, MPI_INT, i, 0, MPI_COMM_WORLD);
				MPI_Recv(&reply[i], 1, MPI_INT, i, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
			}
		}
		in_critical_section = 1;
		for (i = 0; i < N; i++) {
			if (i != rank) {
				if (request[i] > 0 && (request[i] < request[rank] || (request[i] == request[rank] && i < rank))) {
					in_critical_section = 0;
					break;
				}
			}
		}
		if (in_critical_section) {
			printf("Process %d is in the critical section\n", rank);
			for (i = 0; i < 1000000; i++) {}
			printf("Process %d is exiting the critical section\n", rank);
			request[rank] = 0;
			for (i = 0; i < N; i++) {
				reply[i] = 0;
			}
			in_critical_section = 0;
		}
		for (i = 0; i < N; i++) {
			if (i != rank && reply[i] > 0) {
				MPI_Send(&reply[i], 1, MPI_INT, i, 0, MPI_COMM_WORLD);
			}
		}
		for (i = 0; i < 1000000; i++) {}
		round--;
	}
	MPI_Finalize();
	return 0;
}
