# osi6-Kang-BPE218
## 6-ая домашняя работа Кан Олеся
### Код программы-клиента на языке си
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
	
int main() {
    int a[23];
    for (int i = 0; i < 23; i++) {
        a[i] = rand() % 50;						                            	    // Генерирую числа от 0 до 50.
    }
    
    char path[] = "server.c";						                            	// Путь до сервера.
    key_t k;
    if ((k = ftok(path,0)) < 0) {					                            	// Генерирую ключ.
	printf("No server file to create a key!\n");
	exit(-1);
    }
	
    int shm;
    if ((shm = shmget(k, 2000*sizeof(char), 0666|IPC_CREAT)) < 0) {	                // Создаю разделяемую память.
        printf("Can't create shared memory!\n");
        exit(-1);
    }
	
    char *b;
    if ((b = (char *)shmat(shm, NULL, 0)) == (char *)(-1)) {		                // Создаю буфер.
        printf("Can't use shared memory!\n");
        exit(-1);
    }
	
    int cur = 0;
    while (cur < 23) {
	b[cur] = a[cur];						                                        // Записываем элементы массива в буфер.
	++cur;
    }
	
    if (shmdt(b) < 0) {
	printf("Can't disconnect memory!\n");				                            // Отключаем клиент.
	exit(-1);
    }
    return 0;
}
```
### Код программы-сервера на языке си
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
	
int main() {
    char path[] = "server.c";							                        	        // Путь к серверу.
    key_t k;
    if ((k = ftok(path, 0)) < 0) {						                    	            // Генерирую ключ.
	printf("No server file to create a key!\n");
	exit(-1);
    }
	
    int shm;
    if ((shm = shmget(k, 2000*sizeof(char), 0666|IPC_CREAT)) < 0) {		                    // Подключаюсь к разделяемой памяти.
	printf("Can't get an access to shared memory!\n");
	exit(-1);
    }
	
    char *b;
    if ((b = (char *)shmat(shm, NULL, 0)) == (char *)(-1)) {			        	        // Записываю в буфер элементы из разделяемой памяти.
	printf("Can't use shared memory!\n");
    }
	
    int cur = 0;
    while(b[cur] != '\0') {
	printf("%d ", b[cur]);						                                            // Печатаю элементы массива.
	++cur;
    }
	
    if (shmdt(b) < 0) {
	printf("Can't disconnect shared memory!\n");					                    	// Удаляю сервер.
	exit(-1);
    }
	
    if (shmctl(shm, 0, NULL) < 0) {
	printf("Can't free shared memory!\n");						                        	// Освобождаю память.
	exit(-1);
    }
	
    return 0;
}
```
## В этой домашней работе требовалось считать данные из одного файла и с помощью трубы записать полученные данные в другой файл
