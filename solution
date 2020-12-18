#include <stdio.h>
#include <pthread.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <unistd.h>
#include <sys/stat.h>
#include <fcntl.h>


struct mymsg{
	long mtype;
	char mtext;
};

int i = 0;
int msg_id;
double work = 0;

#define L 10
#define T 0.1

#define h 1
#define t 0.01

#define N 16
#define LEN 10      //L/h
#define ALL_LEN 11  //LEN+1
#define TIME 10    //T/t
#define COEFF 0.005  //t * (0.5 / h)
#define size 256

struct input{
	double * before;
	double * now;
	int n;
	int left;
	int right;
	};

void * function(void * data){
	clock_t start = clock();
	struct input *spread = (struct input*)data;
	double *spread_before = spread->before;
	double *spread_now = spread->now;
	int n = spread->n;
	int left = spread->left;
	int right = spread->right;
	*spread_now = 1;

	for (int i = left; i <= right; i++){
		*(spread_now+i) = 0.5 * ( *(spread_before+i+1) + *(spread_before+i-1) ) - COEFF * ( *(spread_before+i+1) -  *(spread_before+i-1) );
	}
	struct mymsg done;
	done.mtype = n+1;
	done.mtext = '0';
	if (msgsnd (msg_id, &done, 0, 0) == -1){
		printf("send error1\n");
		exit(errno);
	}
	clock_t finish = clock();
	work += (finish - start);
	pthread_exit((void *)NULL);
	}
			

int main(){

	char path[size];
	if (getcwd(path, size) == NULL) {
		printf("getcwd error\n");
			exit(errno);
	}

	sprintf(path, "%s/for_yulya.txt", path);
	if ( open(path, O_CREAT|0666) == -1){
		printf("create error\n");
		exit(errno);
	}
	key_t msg_key;
        if ( (msg_key = ftok(path , 0)) == -1){
		printf("ftok error\n");
		exit (errno);
       	}

       	if ( (msg_id =  msgget(msg_key, IPC_CREAT|0666)) == -1){
		printf("msgget error\n");
		exit(errno);
       	}

        double *spread_now = (double *) malloc (ALL_LEN * sizeof(double));
	double *spread_before = (double *) malloc (ALL_LEN * sizeof(double));
	*spread_before = 1;
	for (int i = 1; i < ALL_LEN; i++){
		*(spread_before + i) = 0;
	}

	pthread_t thread_id[N];
	struct input data[N];

		for (int i = 0; i < ALL_LEN; i++)
			printf("%f ", *(spread_before+i) );
		printf("\n");
		for(int i=0; i < TIME ; i++){
			for (int j = 0; j < N; j++){
				data[j].before = spread_before;
				data[j].now = spread_now;
				data[j].n = j;
				data[j].left = j*LEN/N + 1;
				data[j].right = (j+1)*LEN/N;
				if( pthread_create( &thread_id[j], (pthread_attr_t *)NULL, function , (void*) (&data[j])) != 0) {
					printf ("create error\n");
					return errno;
				}
			}
			for (int j = 0; j < N; j++){
				if ( pthread_join (thread_id[j], (void *) NULL) == -1 ){
					printf("join error\n");
					exit (errno);
				}
			}
			struct mymsg done;
			for (int j = 0; j < N; j++){
				if ( msgrcv (msg_id, &done, 0, 0, 0) == -1){
					printf("get error2\n");
					exit(errno);
				}
			}
			/*for (int i = 0; i < ALL_LEN; i++){
				*(spread_before+i) = *(spread_now+i);
			}*/
			memcpy(spread_before, spread_now, ALL_LEN * sizeof(double));
			for (int i = 0; i < ALL_LEN; i++){
				printf("%f ", *(spread_now+i));
			}
			printf("\n");
		}
		//printf("%f\n", work/N/TIME);
		free(spread_now);
		free(spread_before);
		if ( msgctl(msg_id, IPC_RMID, NULL) == -1){
				printf("remove queue error\n");
				exit(errno);
		}
		if ( remove(path) == -1){
				printf("remove file error\n");
				exit(errno);
		}
	
}
