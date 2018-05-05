# OCLab7
1.	#include <stdio.h>
2.	#include <sys/types.h>
3.	#include <sys/ipc.h>
4.	#include <sys/shm.h>
5.	#define SEGSIZE 100
6.	main(int argc, char *argv[])
7.	{
8.	key_t key;
9.	int   shmid, cntr;
10.	char  *segptr;
11.	if(argc == 1)
12.	usage();
13.	key = ftok(".", 'S');
14.	if((shmid = shmget(key, SEGSIZE, IPC_CREAT|IPC_EXCL|0666)) == -1) 
15.	{
16.	printf("Shared memory segment exists - opening as client\n");
17.	if((shmid = shmget(key, SEGSIZE, 0)) == -1) 
18.	{
19.	perror("shmget");
20.	exit(1);
21.	}
22.	}
23.	else
24.	{
25.	printf("Creating new shared memory segment\n");
26.	}
27.	if((segptr = shmat(shmid, 0, 0)) == -1)
28.	{
29.	perror("shmat");
30.	exit(1);
31.	}
32.	switch(tolower(argv[1][0]))
33.	{
34.	case 'w': writeshm(shmid, segptr, argv[2]);
35.	break;
36.	case 'r': readshm(shmid, segptr);
37.	break;
38.	case 'd': removeshm(shmid);
39.	break;
40.	case 'm': changemode(shmid, argv[2]);
41.	break;
42.	default: usage();
43.	}
44.	}
45.	writeshm(int shmid, char *segptr, char *text)
46.	{
47.	strcpy(segptr, text);
48.	printf("Done...\n");
49.	}
50.	readshm(int shmid, char *segptr)
51.	{
52.	printf("segptr: %s\n", segptr);
53.	}
54.	removeshm(int shmid)
55.	{
56.	shmctl(shmid, IPC_RMID, 0);
57.	printf("Shared memory segment marked for deletion\n");
58.	}
59.	changemode(int shmid, char *mode) 
60.	{
61.	struct shmid_ds myshmds;
62.	shmctl(shmid, IPC_STAT, &myshmds);
63.	printf("Old permissions were: %o\n", myshmds.shm_perm.mode);
64.	sscanf(mode, "%o", &myshmds.shm_perm.mode);
65.	shmctl(shmid, IPC_SET, &myshmds);
66.	printf("New permissions are : %o\n", myshmds.shm_perm.mode);
67.	}
68.	usage()
69.	{
70.	fprintf(stderr, "shmtool - A utility for tinkering with shared memory\n");
71.	fprintf(stderr, "\nUSAGE:  shmtool (w)rite <text>\n");
72.	fprintf(stderr, "                (r)ead\n");
73.	fprintf(stderr, "                (d)elete\n");
74.	fprintf(stderr, "                (m)ode change <octal mode>\n");
75.	exit(1);
76.	}
