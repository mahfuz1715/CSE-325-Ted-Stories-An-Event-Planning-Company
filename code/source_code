#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <sys/types.h>
#include <time.h>
#include <string.h>

#define MAX_EVENTS 20
#define MAX_PLANNERS 5
#define MAX_RESOURCES 3
#define MAX_IDEAS 5

typedef struct {
    char log_buffer[4096];
    int used_proposal_spots[MAX_RESOURCES];
    int used_decorations[MAX_RESOURCES];
    int used_av_equipment[MAX_RESOURCES];
    int idea_in_progress;
    int idea_counter;
} SharedData;

union semun {
    int val;
    struct semid_ds *buf;
    unsigned short *array;
};

void sem_wait(int semid, int sem_num) {
    struct sembuf sb = {sem_num, -1, 0};
    semop(semid, &sb, 1);
}

void sem_signal(int semid, int sem_num) {
    struct sembuf sb = {sem_num, 1, 0};
    semop(semid, &sb, 1);
}

void write_log(SharedData *shared, const char *msg) {
    printf("%s", msg);
    strcat(shared->log_buffer, msg);
}

void share_idea(SharedData *shared, int semid, int planner_id) {
    sem_wait(semid, 4);
    
    shared->idea_in_progress = 1;
    char* ideas[MAX_IDEAS] = {
        "We could create a memory lane with photos from their childhood",
        "What if we surprise them with a flash mob?",
        "We could recreate their first date!",
        "A traditional nakshi kantha quilt signing ceremony for the couple",
        "A floating lantern ceremony on a pond with terracotta diyas"
    };
    
    char msg[256];
    int idea_index = shared->idea_counter % MAX_IDEAS;
    sprintf(msg, "Planner %d shares a spontaneous idea: %s\n", planner_id, ideas[idea_index]);
    write_log(shared, msg);
    
    shared->idea_counter++;
    sleep(1);
    shared->idea_in_progress = 0;
    
    sem_signal(semid, 4);
}

void planner_process(SharedData *shared, int semid, int planner_id) {
    char msg[256];
    sprintf(msg, "Event Planner %d is ready for new assignments!\n", planner_id);
    write_log(shared, msg);

    while (1) {
        sem_wait(semid, 0);
        
        sprintf(msg, "Planner %d is assigned to a new event!\n", planner_id);
        write_log(shared, msg);
        
        if (rand() % 2) {
            share_idea(shared, semid, planner_id);
        }
        
        sem_wait(semid, 1);
        int spot_index = -1;
        for (int i = 0; i < MAX_RESOURCES; i++) {
            if (!shared->used_proposal_spots[i]) {
                shared->used_proposal_spots[i] = 1;
                spot_index = i;
                break;
            }
        }
        sprintf(msg, "Planner %d reserves proposal spot: %s\n", 
                planner_id, 
                spot_index == 0 ? "Empire State Building" :
                spot_index == 1 ? "Central Park Bow Bridge" : "Brooklyn Heights Promenade");
        write_log(shared, msg);
        
        sem_wait(semid, 2);
        int decor_index = -1;
        for (int i = 0; i < MAX_RESOURCES; i++) {
            if (!shared->used_decorations[i]) {
                shared->used_decorations[i] = 1;
                decor_index = i;
                break;
            }
        }
        sprintf(msg, "Planner %d chooses decoration theme: %s\n", 
                planner_id, 
                decor_index == 0 ? "Vintage Romance" :
                decor_index == 1 ? "Modern Elegance" : "Rustic Charm");
        write_log(shared, msg);
        
        sem_wait(semid, 3);
        int av_index = -1;
        for (int i = 0; i < MAX_RESOURCES; i++) {
            if (!shared->used_av_equipment[i]) {
                shared->used_av_equipment[i] = 1;
                av_index = i;
                break;
            }
        }
        sprintf(msg, "Planner %d reserves AV equipment: %s\n", 
                planner_id, 
                av_index == 0 ? "Projector & Screen" :
                av_index == 1 ? "Sound System" : "Lighting Setup");
        write_log(shared, msg);
        
        sprintf(msg, "Planner %d is working on the event...\n", planner_id);
        write_log(shared, msg);
        sleep(2 + rand() % 3);
        
        shared->used_proposal_spots[spot_index] = 0;
        sem_signal(semid, 1);
        shared->used_decorations[decor_index] = 0;
        sem_signal(semid, 2);
        shared->used_av_equipment[av_index] = 0;
        sem_signal(semid, 3);
        
        sprintf(msg, "Planner %d has completed the event!\n", planner_id);
        write_log(shared, msg);
        
        sem_signal(semid, 0);
    }
}

int main() {
    printf("=== Ted's Stories Event Planning Simulation ===\n");
    
    int shmid = shmget(IPC_PRIVATE, sizeof(SharedData), IPC_CREAT | 0666);
    SharedData *shared = (SharedData *)shmat(shmid, NULL, 0);
    memset(shared, 0, sizeof(SharedData));
    
    srand(time(NULL));
    shared->idea_counter = rand() % MAX_IDEAS;
    
    int semid = semget(IPC_PRIVATE, 5, IPC_CREAT | 0666);
    union semun su;
    
    su.val = MAX_PLANNERS;
    semctl(semid, 0, SETVAL, su);
    
    su.val = MAX_RESOURCES;
    semctl(semid, 1, SETVAL, su);
    semctl(semid, 2, SETVAL, su);
    semctl(semid, 3, SETVAL, su);
    
    su.val = 1;
    semctl(semid, 4, SETVAL, su);
    
    pid_t planners[MAX_PLANNERS];
    for (int i = 0; i < MAX_PLANNERS; i++) {
        planners[i] = fork();
        if (planners[i] == 0) {
            planner_process(shared, semid, i + 1);
            exit(0);
        }
        usleep(10000); 
    }
    
    for (int i = 1; i <= MAX_EVENTS; i++) {
        char msg[256];
        sprintf(msg, "\n=== New Event Request Received! (Event %d) ===\n", i);
        write_log(shared, msg);
        
        sem_signal(semid, 0);
        sleep(1 + rand() % 2);
    }
    
    sleep(15);
    
    for (int i = 0; i < MAX_PLANNERS; i++) {
        kill(planners[i], SIGTERM);
    }
    
    FILE *logFile = fopen("event_log.txt", "w");
    fprintf(logFile, "%s", shared->log_buffer);
    fclose(logFile);
    
    shmdt(shared);
    shmctl(shmid, IPC_RMID, NULL);
    semctl(semid, 0, IPC_RMID);
    
    printf("\nSimulation complete. \n");
    return 0;
}
