---
title: "[C] Built-In Functions 정리"
excerpt: "파일 입출력과 프로세스를 제어하는 시스템 내장 함수들을 정리한다."
categories:
  - C
tags:
  - C
toc: true
toc_sticky: true
last_modified_at: 2020-09-10T20:13:00-05:00
---

## 1. open

```c
#include <fcntl.h>
int open(const char *name, int flags);
int open(const char *name, int flags, mode_t mode);
```

``open()`` 시스템 콜은 경로가 파일을 열거나 생성할 때 사용한다. name에 해당하는 파일을 파일 디스크립에 맵핑하고 성공하면 이를 반환한다.

flag는 파일을 어떤 모드로 열 것인지 결정하는데 사용된다. 또한 ``O_WRONLY | OCREAT``와 같이 flag는 여러 인자를 받을 수 있다. ``O_CREAT``로 파일을 생성하게 된다면, mode에 유닉스 접근 권한으로 0644를 추가 명시해주어야 에러가 나지 않는다.

<br>

## 2. read

```c
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t len);
```

호출 시, fd가 참조하고 있는 파일의 현재 오프셋에서 len 크기 만큼을 buf로 읽어들인다. 성공하는 경우 buf에 쓴 바이트의 숫자를, EOF에 도달하는 경우 0을 리턴한다. 실패하는 경우 -1을 리턴한 뒤, errno를 설정한다.

<br>

## 3. close

```c
int close(int fd);
```

데이터 입출력용 버퍼를 해제하며, 사용자 파일 디스크립터 테이블 내에서 할당된 구조체를 해제한다.

<br>

## 4. fork

```c
#include <unistd.h>
pid_t fork (void);
```

``fork()``는 현재 실행되는 프로세스에 대해 복사본 프로세스를 생성한다. 두 프로세스는 별다른 조작을 하지 않는다면, 계속 실행되는 상태가 된다.

성공시 자식 프로세스의 ID가 부모에게 리턴되며, 자식 프로세스에는 0이 리턴된다. 실패시 프로세스는 생성되지 않고, 부모에게는 -1이 리턴된다.

<br>

## 5. wait

```c
#include <sys/wait.h>
pid_t wait(int *statloc);
```

``wait()``를 사용하면 부모 프로세스는 자식 프로세스가 종료할 때 까지 ``sleep()`` 모드로 기다리게 된다.

부모 프로세스가 자식 프로세스보다 먼저 종료되어서, 자식 프로세스가 고아 프로세스가 되는 것을 방지하기 위함이다. 자식 프로세스가 종료되면 함수는 즉시 리턴되며, 자식이 사용한 모든 시스템 자원을 해제한다.

인자에 status를 넣어 자식 프로세스의 상태를 받아올 수 있다. 종료된 자식의 프로세스 ID는 에러일 경우 -1, 그렇지 않을 경우 0을 반환한다.

> 예제

```c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <string.h>

int main()
{
    int pid;
    int status;

    pid = fork();
    if (pid < 0)
    {
        perror("FORK ERROR :");
        exit(0);
    }    
    if (pid == 0) //자식 프로세스
    {
        int i;
        for (i = 0; i < 5; i++)
        {
            printf("Child : %d\n", i);
            sleep(2);
        }
        exit(3);
    }
    else
    {
        // 부모프로세스는 자식프로세스가
        // 종료할때까지 기다린다.
        printf("I wait Child(%d)\n", pid);
        wait(&status);
        printf("Child is exit (%d)\n", status);
    }
}
```

해당 예제를 컴파일하면 다음과 같은 결과가 나온다.

```bash
I wait Child(12128)
Child : 0
Child : 1
Child : 2
Child : 3
Child : 4
Child is exit (768)
```

### 5.1. waitpid

```c
#include <sys/wait.h>
pid_t waitpid(pid_t pid, int *statloc , int options);
```

``waitpid()``는 인수로 주어진 pid 자식프로세스가 종료되거나, 시그널 함수를 호출하는 신호가 전달될때까지 ``waitpid()``를 호출한 영역은 일시 중지 된다.

그러나 자식 프로세스가 종료될 때 까지 차단되는 것을 원하지 않을 경우, 옵션을 사용하여 차단을 방지할 수 있다.

### 5.2. wait3 및 wait4

```c
pid_t wait3(int *staloc, int options, struct rusage *rus);
pid_t wait4(pid_t pid, int *staloc, int options, struct rusage *rus);
```

자원 사용량을 확인할 수 있다.

<br>

## 6. signal

```c
#include <signal.h>
void (*signal(int signum, void (*handler)(int)))(int);
```

특정 시그널이 발생할 때 처리할 방법을 지정하는 함수이다.

> 예제

```c
#include <stdio.h>
#include <signal.h>

void (*old_fun)( int);

void sigint_handler( int signo)
{
   printf( "Ctrl-C 키를 눌루셨죠!!\n");
   printf( "또 누르시면 종료됩니다.\n");
   signal( SIGINT, old_fun); // 또는 signal( SIGINT, SIG_DFL);
}

int main( void)
{
   old_fun = signal( SIGINT, sigint_handler);
   while(1)
   {
      printf( "forum.falinux.com\n");
      sleep(1);
   }
}
```

시그널이 발생할 때, 옵션을 통해 다음과 같이 설정할 수 있다.

1. 기존 방법으로 처리하기.
2. 무시하기.
3. 프로그램에서 직접 처리하기.

<br>

## 7. kill

```c
#include <signal.h>
int kill(pid_t pid, int sig);
```

``kill()``은 쉘에서 프로세스를 죽이는 kill 명령과는 달리 프로세스에 시그널을 전송한다. 프로세스에 SIGKILL을 보내면 쉘 명령의 kill 과 같은 역할을 한다.

<br>

## 8. exit

```c
#include <stdio.h>
void exit(int status);
```

프로그램을 종료한다.

<br>

## 9. getcwd

```c
#include <unistd.h>
char *getcwd(char *buf, size_tsize);
```

현재 작업 중인 디렉토리의 전체 이름을 buf에 담는다.

### 9.1. chdir

```c
#include <direct.h>
int chdir(const char *dirname);
```

현재 작업 중인 디렉토리의 위치를 변경한다.

<br>

## 10. stat과 lstat 및 fstat

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int stat(const char *file_name, struct stat *buf);
int lstat(const char *file_name, struct stat *buf);
int fstat(int filedes, struct stat *buf);
```

파일 정보를 읽는 함수이다. 첫 번째 인자로 주어진 파일(경로 혹은 파일 디스크립터)을 읽고 stat 구조체에 정보를 담는다.

<br>

## 11. execve

```c
#include <unistd.h>
int execve(const char *filename, char *const argv[], char *const envp[]);
```

실행 가능한 파일인 filename의 실행코드를 현재 프로세스에 적재하여 기존의 실행코드와 교체하여 새로운 기능으로 실행한다. 즉, 현재 실행되는 프로그램의 기능은 없어지고 filename 프로그램을 메모리에 loading하여 처음부터 실행한다.

<br>

## 12. dup 및 dup2

```c
#include <unistd.h>
int dup(int oldfd);
int dup2(int oldfd, int newfd);
```

열려 있는 파일 디스크립터를 복제하는 함수이다. ``dup2()`` 함수는 사용자가 원하는 번호로 파일 디스크립터를 설정할 수 있다. 만약 newfd가 이미 사용 중인(열려있는) fd라면, 이를 닫고 oldfd를 복사하도록 한다.
.
같은 파일 테이블을 참조하기 때문에 원본 또는 사본에서 ``read()``로 읽어 들이면 같은 오프셋을 참조한다.

<br>

## 13. pipe

```c
#include <unistd.h>
int pipe(int fd[2]);
```

프로세스들은 메모리가 독립적이기 때문에, 프로세스간 데이터를 주고 받는 것이 불가능하다. 이 때, 파이프를 활용하여 전달할 수 있다.

* fd[0] : 파이프의 출구로서, 데이터를 입력받을 수 있는 파일 디스크립터가 담긴다.
* fd[1] : 파이프의 입구로서, 데이터를 출력할 수 있는 파일 디스크립터가 담긴다.

<br>

## 14. opendir

```c
#include <dirent.h>
DIR* opendir(const char *name);
```

지정한 디렉토리를 열고, 해당 디렉토리를 참조하는 포인터를 리턴한다.

### 14.1. readdir

```c
#include <dirent.h>
struct dirent *readdir(DIR *dir);
```

open된 디렉토리에서 각각의 항목, 즉 엔트리를 읽는다. 반복문을 돌려 사용한다.

```c
#include  <stdio.h>
#include  <sys/types.h>
#include  <sys/stat.h>
#include  <unistd.h>
#include  <dirent.h>

int main()
{
    DIR*  dp  = NULL;
    struct  dirent*  entry  =  NULL;
    struct  stat   buf;

    if ((dp=opendir("/home/thelegend/test"))  == NULL)
    {
        printf("/home/thelegend/test 를 열수 없습니다.\n");
        return -1;
    }
    while ((entry = readdir(dp)) != NULL)
    {
        lstat(entry->d_name, &buf);
        if (S_ISDIR(buf.st_mode))
          printf("[디렉토리이름]  %s\n", entry->d_name);
        else if (S_ISREG(buf.st_mode))
          printf("[파일이름] %s \n", entry->d_name);         
    }
    closedir(dp);
}
```

open한 디렉토리의 엔트리를 하나씩 읽고, 해당 엔트리의 이름에 해당하는 파일 정보를 buf에 담는다.

st_mode는 파일 형식정보로, 해당 파일이 디렉토리인지 파일인지 등의 정보가 담겨 있다.

### 14.2. closedir

```c
#include <dirent.h>
int closedir( DIR *dir);
```

파일처럼 사용한 디렉토리를 닫는다.

<br>

## 15. strerror

```c
#include <string.h>

char* strerror(int errnum);
```

errnum을 통해 발생한 오류에 알맞은 메시지를 가르키는 포인터를 리턴한다. 문자열 리터럴을 반환하기 때문에, 값을 변경할 수 없다.

<br>

## 16. errno

```c
#include <errno.h>
extern int errno;
```

시스템 콜을 실패하면, 정적 전역 변수인 errno를 특정 에러를 나타내는 숫자로 변환한다. 다른 함수를 실행하면 값이 변경되기 때문에, errno 변수는 해당 변수를 설정한 함수 호출 직후에만 유효하다.

<br>

---

## References

* [임베디드 리눅스 시스템 포럼](http://forum.falinux.com/zbxe/)
* [Joinc](https://www.joinc.co.kr/w/FrontPage)
