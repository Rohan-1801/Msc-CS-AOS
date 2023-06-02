AOS Program
Slip1
A) Write a C program to find whether a given file is present in current directory or not [10]
#include <stdio.h>
#include <dirent.h>
#include <string.h>
int main()
{
DIR *dir;
struct dirent *entry;
char filename[100];
int found = 0;
printf("Enter file name to search: ");
scanf("%s", filename);
dir = opendir(".");
if (dir == NULL)
{
printf("Error opening directory.\n");
return 1;
}
while ((entry = readdir(dir)) != NULL)
{
if (strcmp(entry->d_name, filename) == 0)
{
found = 1;
break;
}
}if (found)
{
printf("File found.\n");
}
else
{
printf("File not found.\n");
}
closedir(dir);
return 0;
}
Output:
Enter file name to search:

==================================================================
B) Write a C program which blocks SIGOUIT signal for 5 seconds. After 5 second process checks any
occurrence of quit signal during this period, if so, it unblock the signal. Now another occurrence of
quit signal terminates the program. (Use sigprocmask() and sigpending() ) [20]
Ans:
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
#include <stdlib.h>
int main()
{
sigset_t mask, oldmask, pending;
struct timespec timeout;
sigemptyset(&mask);sigaddset(&mask, SIGQUIT);
sigprocmask(SIG_BLOCK, &mask, &oldmask);
printf("Blocked SIGQUIT signal for 5 seconds.\n");
timeout.tv_sec = 5;
timeout.tv_nsec = 0;
nanosleep(&timeout, NULL);
sigpending(&pending);
if (sigismember(&pending, SIGQUIT))
{
printf("SIGQUIT signal occurred during blocking period. Unblocking signal.\n");
sigprocmask(SIG_UNBLOCK, &mask, NULL);
}
else
{
printf("No SIGQUIT signal occurred during blocking period.\n");
}
sigpending(&pending);
if (sigismember(&pending, SIGQUIT))
{
printf("SIGQUIT signal occurred after unblocking. Terminating program.\n");
exit(0);
}
else
{
printf("No SIGQUIT signal occurred after unblocking. Continuing program.\n");
}return 0;
}
===================================================================================

Slip2 
A) Write a C program that a string as an argument and return all the files that begins with that name
in the current directory. For example > ./a.out foo will return all file names that begins with foo.[10]

#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <string.h>
int main(int argc, char *argv[])
{
DIR *dir;
struct dirent *entry;
char *prefix;
int len;
if (argc != 2)
{
fprintf(stderr, "Usage: %s prefix\n", argv[0]);
exit(EXIT_FAILURE);
}
prefix = argv[1];
len = strlen(prefix);
dir = opendir(".");
if (dir == NULL)
{
perror("opendir");
exit(EXIT_FAILURE);
}while ((entry = readdir(dir)) != NULL)
{
if (strncmp(entry->d_name, prefix, len) == 0)
{
printf("%s\n", entry->d_name);
}
}
closedir(dir);
exit(EXIT_SUCCESS);
}

===============================================================
B) Write a C program to demonstrates the different behavior that can be seen with automatic,
global, register, static and volatile variables (Use setjmp() and longjmp() system call).

#include <stdio.h>
#include <setjmp.h>
jmp_buf env;
int global_var = 1;
void func1()
{
int auto_var = 2;

register int reg_var = 3;
// Static variable
static int static_var = 4;
// Volatile variable
volatile int vol_var = 5;

printf("func1: auto_var = %d, reg_var = %d, static_var = %d, vol_var = %d\n", auto_var, reg_var,
static_var, vol_var);
// Set jump point
if (setjmp(env) != 0)
{
printf("func1: longjmp called.\n");
return;
}
// Modify variables
auto_var++;
reg_var++;
static_var++;
vol_var++;
global_var++;
printf("func1: auto_var = %d, reg_var = %d, static_var = %d, vol_var = %d, global_var = %d\n",
auto_var, reg_var, static_var, vol_var, global_var);
// Call func2

}
void func2()
{
printf("func2: global_var = %d\n", global_var);
// Long jump to func1
longjmp(env, 1);
}
int main()
{
func1();
return 0;

}
Ans:
func1: auto_var = 2, reg_var = 3, static_var = 4, vol_var = 5
func1: auto_var = 3, reg_var = 4, static_var = 5, vol_var = 6, global_var = 2

==================================================================
Slip3
A) Write a C program to find file properties such as inode number, number of hard link, File
permissions, File size, File access and modification time and so on of a given file using stat() system
call. [10]

#include <stdio.h>
#include <sys/stat.h>
#include <time.h>
int main()
{
struct stat file_stat;
char file_name[100];
printf("Enter file name: ");
scanf("%s", file_name);
// Call stat() to get file properties
if (stat(file_name, &file_stat) == -1)
{
perror("Error in stat");
return 1;
}
printf("File name: %s\n", file_name);
printf("Inode number: %ld\n", file_stat.st_ino);
printf("Number of hard links: %ld\n", file_stat.st_nlink);
printf("File permissions: %o\n", file_stat.st_mode & 0777);

printf("File size: %ld bytes\n", file_stat.st_size);
printf("Last access time: %s", ctime(&file_stat.st_atime));
printf("Last modification time: %s", ctime(&file_stat.st_mtime));

return 0;
}
Output:
Enter file name:

=================================================================
B) Write a C program to create ‘n’ child processes. When all ‘n’ child processes terminates, Display
total cumulative time children spent in user and kernel mode. [20]

#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/times.h>
#include <unistd.h>
int main(int argc, char *argv[])
{
if (argc != 2)
{
printf("Usage: %s <n>\n", argv[0]);
return 1;
}
int n = atoi(argv[1]);
pid_t pid;
int i;
struct tms t_start, t_end;
clock_t c_start, c_end;

long utime = 0, stime = 0;
c_start = times(&t_start);
for (i = 0; i < n; i++)
{
pid = fork();
if (pid == -1)
{
perror("fork error");
return 1;
}
else if (pid == 0)
{
// Child process
exit(0);
}
}
// Wait for all child processes to terminate
for (i = 0; i < n; i++)
{
wait(NULL);
}
c_end = times(&t_end);
// Calculate user and system time for all child processes
utime = t_end.tms_cutime - t_start.tms_cutime;
stime = t_end.tms_cstime - t_start.tms_cstime;
printf("Total user time: %ld ticks\n", utime);
printf("Total system time: %ld ticks\n", stime);
printf("Total CPU time: %ld ticks\n", utime + stime);
return 0;

}
================================================================================

Slip4
A) Write a C program to find file properties such as inode number, number of hard link, File
permissions, File size, File access and modification time and so on of a given file using fstat() system
call. [10]

#include <stdio.h>
#include <sys/stat.h>
int main(int argc, char *argv[])
{
struct stat file_info;
char *filename;
if (argc < 2)
{
printf("Usage: %s <filename>\n", argv[0]);
return 1;
}
filename = argv[1];
if (stat(filename, &file_info) != 0)
{
printf("Error: Could not get file information.\n");
return 1;
}
printf("File name: %s\n", filename);
printf("Inode number: %ld\n", file_info.st_ino);
printf("Number of hard links: %ld\n", file_info.st_nlink);
printf("File permissions: %o\n", file_info.st_mode & 0777);
printf("File size: %ld bytes\n", file_info.st_size);
printf("File access time: %ld seconds since Epoch\n", file_info.st_atime);

printf("File modification time: %ld seconds since Epoch\n", file_info.st_mtime);

return 0;
}
===================================================================================
B) Write a C program to implement the following unix/linux command (use fork, pipe and exec

system call). Your program should block the signal Ctrl-C and Ctrl-\ signal during the execution. ls –l |
wc–l [20]
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <signal.h>
void handle_signal(int signum)
{
// Do nothing
}
int main()
{
// Block SIGINT and SIGQUIT signals
struct sigaction sa;
sa.sa_handler = handle_signal;
sigemptyset(&sa.sa_mask);
sa.sa_flags = 0;
sigaction(SIGINT, &sa, NULL);
sigaction(SIGQUIT, &sa, NULL);
// Create pipe
int pipefd[2];
if (pipe(pipefd) == -1)
{

perror("pipe");
exit(EXIT_FAILURE);
}
// Fork
pid_t pid = fork();
if (pid == -1)
{
perror("fork");
exit(EXIT_FAILURE);
}
if (pid == 0) { // Child process (ls -l)
// Redirect stdout to write end of pipe
dup2(pipefd[1], STDOUT_FILENO);
// Close unused read end of pipe
close(pipefd[0]);
// Execute ls -l
execlp("ls", "ls", "-l", NULL);
// This code should not be reached unless execlp fails
perror("execlp");
exit(EXIT_FAILURE);
}
else
{
// Parent process
// Fork again
pid_t pid2 = fork();
if (pid2 == -1)
{
perror("fork");

exit(EXIT_FAILURE);
}

if (pid2 == 0)
{
// Child process (wc -l)
// Redirect stdin to read end of pipe
dup2(pipefd[0], STDIN_FILENO);
// Close unused write end of pipe
close(pipefd[1]);
// Execute wc -l
execlp("wc", "wc", "-l", NULL);
// This code should not be reached unless execlp fails
perror("execlp");
exit(EXIT_FAILURE);
}
else
{
// Parent process
// Close both ends of pipe
close(pipefd[0]);
close(pipefd[1]);
// Wait for both child processes to complete
int status;
waitpid(pid, &status, 0);
waitpid(pid2, &status, 0);
}
}
return 0;

}


===================================================================================

Slip 6
A) Write a C program to map a given file in memory and display the contain of mapped file in
reverse. [10]
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
int main(int argc, char *argv[])
{
int fd;
struct stat sb;
char *addr, *start, *end;
if (argc != 2)
{
fprintf(stderr, "Usage: %s <file>\n", argv[0]);

exit(EXIT_FAILURE);
}

fd = open(argv[1], O_RDONLY);
if (fd == -1)
{
perror("open");
exit(EXIT_FAILURE);
}
if (fstat(fd, &sb) == -1)
{
perror("fstat");
exit(EXIT_FAILURE);
}
addr = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
if (addr == MAP_FAILED)
{
perror("mmap");
exit(EXIT_FAILURE);
}
// Traverse the mapped file in reverse and print its contents
start = addr;
end = addr + sb.st_size - 1;
while (end >= start)
{
putchar(*end--);
}
if (munmap(addr, sb.st_size) == -1)
{

perror("munmap");
exit(EXIT_FAILURE);
}

close(fd);
exit(EXIT_SUCCESS);
}
===============================================================================
B) Write a C program which receives file names as command line arguments and display those
filenames in ascending order according to their sizes. (e.g $ a.out a.txt b.txt c.txt, …) [20]
#include<stdio.h>
#include<dirent.h>
#include<string.h>
#include<sys/stat.h>
#include<time.h>
#include<stdlib.h>
struct filelist
{
char fname[100];
int fsize;
}
int main(int argc, char *argv[])
{
DIR *dp;
int i,j,k;
struct dirent *ep;
struct stat sb;
char mon[100];
struct filelist f1[100],temp;
j=0;

for(i=1;i<argc;i++)
{
dp=opendir("./");
if(dp=NULL)
{
while(ep=readdir(dp))
{
if((strcmp(ep->d_name,argv[i]))==0)
{
stat(ep->d_name,&sb);
strcpy(f1[j].fname,ep->d_name);
f1[j].fsize=sb.st_size;
j++;
break;
}
}
}
(void)closedir(dp);
}
for(i=0;i<j;i++)
{
for(k=0;k<j;k++)
{
if(f1[i].fsize<f1[k].fsize)
{
temp=f1[k];
f1[k]=f1[i];
f1[i]=temp;
}

}
}
for(i=0;i<j;i++)
{
printf("%s\t%d\n",f1[i].fname,f1[i].fsize);
}
return 0;
}
======================================================================================

Slip7
A) Write a C program to create a file with hole in it. [10]
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
int main()
{
int n =create(“file txt”;”w”)
char ch[16]=”hello world how ”;
char str[20]=”od-c file text ”;
system(“chmod 777 file.txt”);
write(n,ch,16);
lseek(n,48,SEEK_SET);
write(n,ch,16);
system(str);
return(0);
}
===================================================================================
B) Write a C program which display the information of a given file similar to given by the unix /linux
command on current directory (l.e File Access permission, file name, file type, User id, group id, file size,
file access and modified time and so on) ls –l DO NOT simply exec ls -l or system command from the
program. [20]

#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <pwd.h>
#include <grp.h>
#include <time.h>
int main(int argc, char* argv[])
{
struct stat st;
struct passwd* pwd;
struct group* grp;
char* filename;
char permissions[11];
int i;

// Check if a file name is provided as command line argument
if (argc != 2)
{
printf("Usage: %s <filename>\n", argv[0]);
return 1;
}

// Get the file name from command line argument
filename = argv[1];

// Get the file information using stat function
if (stat(filename, &st) != 0) {
printf("Error: Unable to get file information.\n");
return 1;

}

// Get file permissions
permissions[0] = (S_ISDIR(st.st_mode)) ? 'd' : '-';
permissions[1] = (st.st_mode & S_IRUSR) ? 'r' : '-';
permissions[2] = (st.st_mode & S_IWUSR) ? 'w' : '-';
permissions[3] = (st.st_mode & S_IXUSR) ? 'x' : '-';
permissions[4] = (st.st_mode & S_IRGRP) ? 'r' : '-';
permissions[5] = (st.st_mode & S_IWGRP) ? 'w' : '-';
permissions[6] = (st.st_mode & S_IXGRP) ? 'x' : '-';
permissions[7] = (st.st_mode & S_IROTH) ? 'r' : '-';
permissions[8] = (st.st_mode & S_IWOTH) ? 'w' : '-';
permissions[9] = (st.st_mode & S_IXOTH) ? 'x' : '-';
permissions[10] = '\0';

// Get owner user name and group name
pwd = getpwuid(st.st_uid);
grp = getgrgid(st.st_gid);

// Print file information
printf("%s %2ld %s %s %5ld %s %s\n", permissions, st.st_nlink, pwd->pw_name, grp->gr_name,
st.st_size, ctime(&st.st_mtime), filename);

return 0;
}


=======================================================================================

Slip 9
A) Write a C program to display as well as resets the environment variable such as path, home, root etc.
[10]
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int main()
{
// Display the current environment variable values
printf("Current value of PATH: %s\n", getenv("PATH"));
printf("Current value of HOME: %s\n", getenv("HOME"));
printf("Current value of ROOT: %s\n", getenv("ROOT"));

// Set new values for environment variables
setenv("PATH", "/usr/local/bin:/usr/bin:/bin", 1);
setenv("HOME", "/home/user", 1);
setenv("ROOT", "/", 1);

// Display the new environment variable values
printf("New value of PATH: %s\n", getenv("PATH"));
printf("New value of HOME: %s\n", getenv("HOME"));
printf("New value of ROOT: %s\n", getenv("ROOT"));

// Reset the environment variables to their original values
setenv("PATH", "", 1);
setenv("HOME", "", 1);
setenv("ROOT", "", 1);

// Display the reset environment variable values
printf("Reset value of PATH: %s\n", getenv("PATH"));
printf("Reset value of HOME: %s\n", getenv("HOME"));
printf("Reset value of ROOT: %s\n", getenv("ROOT"));

return 0;
}
Output:
Current value of PATH: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/run/swift-5.7.2RELEASE-ubuntu22.04/usr/bin
Current value of HOME: /home/compiler
Current value of ROOT: (null)
New value of PATH: /usr/local/bin:/usr/bin:/bin
New value of HOME: /home/user
New value of ROOT: /
Reset value of PATH:
Reset value of HOME:
Reset value of ROOT:

==================================================================
B) Write a C program that will only list all subdirectories in alphabetical order from current directory.
[20]
#include <stdio.h>
#include <dirent.h>
#include <string.h>

#include <sys/stat.h>
int main()
{
DIR *dir;
struct dirent *ent;
struct stat st;
// Open the current directory
dir = opendir(".");
if (dir == NULL)
{
perror("opendir failed");
return 1;
}
// Loop through all entries in the directory
while ((ent = readdir(dir)) != NULL)
{
// Check if the entry is a directory
if (ent->d_type == DT_DIR && strcmp(ent->d_name, ".") != 0 && strcmp(ent->d_name, "..") != 0)
{
// Print the name of the directory
printf("%s\n", ent->d_name);
}
}
// Close the directory
closedir(dir);
return 0;
}

Output:
lock
systemd
mount
secrets
node_modules
swift-5.7.2-RELEASE-ubuntu22.04
apache2
sendsigs.omit.d
log
user
=======================================================================================

Slip 10
A) Write a C program to display statistics related to memory allocation system. (Use mallinfo() system
call). [10]
solution:

#include <stdio.h>
#include <malloc.h>
int main()
{
struct mallinfo mi;
// Get memory allocation statistics using mallinfo
mi = mallinfo();
// Print the statistics
printf("Total non-mapped bytes (arena):
printf("# of free chunks (ordblks):

%d\n", mi.arena);

%d\n", mi.ordblks);

printf("# of free fastbin blocks (smblks): %d\n", mi.smblks);
printf("# of mapped regions (hblks):

%d\n", mi.hblks);

printf("Bytes in mapped regions (hblkhd): %d\n", mi.hblkhd);
printf("Max. total allocated space (usmblks):%d\n", mi.usmblks);
printf("Free bytes held in fastbins (fsmblks):%d\n", mi.fsmblks);
printf("Total allocated space (uordblks): %d\n", mi.uordblks);
printf("Total free space (fordblks):

%d\n", mi.fordblks);

printf("Topmost releasable block (keepcost): %d\n", mi.keepcost);
return 0;
}

Output:
Total non-mapped bytes (arena):
# of free chunks (ordblks):

0

1

# of free fastbin blocks (smblks): 0
# of mapped regions (hblks):

0

Bytes in mapped regions (hblkhd): 0
Max. total allocated space (usmblks):0
Free bytes held in fastbins (fsmblks):0
Total allocated space (uordblks): 0
Total free space (fordblks):

0

Topmost releasable block (keepcost): 0
====================================================================================================
B) Write a C program which creates two files. The first file should have read and write permission to

owner, group of owner and other users whereas second file has read and write permission to owner(use
umask() function). Now turn on group-id and turn off group execute permission of first file. Set the read
permission to all user for second file (use chmod() function). [20]
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int main()
{

int fd1, fd2;
// Create the first file with read and write permission for all users
fd1 = open("file1.txt", O_WRONLY | O_CREAT, 0666);
if (fd1 < 0)
{
perror("open failed for file1.txt");
return 1;
}
// Set the umask to remove group and other write permissions for the second file
umask(S_IWGRP | S_IWOTH);

// Create the second file with read and write permission for the owner only
fd2 = open("file2.txt", O_WRONLY | O_CREAT, 0600);
if (fd2 < 0)
{
perror("open failed for file2.txt");
return 1;
}
// Set the read permission for all users on the second file
if (chmod("file2.txt", S_IRUSR | S_IRGRP | S_IROTH) < 0)
{
perror("chmod failed for file2.txt");
return 1;
}
// Turn on group ID and turn off group execute permission for the first file
if (chmod("file1.txt", S_ISGID | S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH) < 0)
{
perror("chmod failed for file1.txt");
return 1;

}
return 0;
}
=================================================================================================================================
Slip11Q2: 
A) Write a C program to create variable length arrays using alloca() system
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <alloca.h>
int main()
{int n;
printf("Enter the size of the array:
");scanf("%d", &n);

// Allocate memory for the array using
alloca()int *arr = (int*) alloca(n * sizeof(int));

// Initialize the array with random
valuesfor (int i = 0; i < n; i++) {
arr[i] = rand() % 100;
}

// Print the array
printf("The array is: ");
for (int i = 0; i < n; i++)
{printf("%d ", arr[i]);
}

return 0;
}
==================================================================================================

B): Write a C program that behaves like a shell (command interpreter). It has its own prompt say
“NewShell$”. Any normal shell command is executed from your shell by starting a child process to
execute the system program corresponding to the command. It should additionally interpret the
following command. i) count c - print number of characters in file ii) count w - print number of words
in file iii) count l - print number of lines in fil

Solution:
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_COMMAND_LENGTH 100
#define MAX_FILENAME_LENGTH 100

int count_characters(char *filename)
{FILE *fp = fopen(filename, "r");
if (fp == NULL) {
printf("Error: Failed to open file %s\n", filename);
return -1;
}

int count =
0;int ch;
while ((ch = fgetc(fp)) != EOF) {
count++;
}

fclose(fp);
return
count;
}

int count_words(char *filename)
{FILE *fp = fopen(filename,
"r"); if (fp == NULL) {
printf("Error: Failed to open file %s\n", filename);

return -1;
}

int count =
0;int ch;
while ((ch = fgetc(fp)) != EOF) {
if (ch == ' ' || ch == '\t' || ch == '\n') {
count++;
}
}

fclose(fp);
return
count;
}

int count_lines(char *filename)
{ FILE *fp = fopen(filename,
"r");if (fp == NULL) {
printf("Error: Failed to open file %s\n", filename);
return -1;
}

int count =
0;int ch;
while ((ch = fgetc(fp)) != EOF) {
if (ch == '\n') {
count++;
}
}

fclose(fp);

return count;
}

int main() {
char command[MAX_COMMAND_LENGTH];
char filename[MAX_FILENAME_LENGTH];
while (1) {
printf("NewShell$ ");
fgets(command, MAX_COMMAND_LENGTH, stdin);
command[strcspn(command, "\n")] = 0; // Remove newline character from input

if (strcmp(command, "exit") == 0) {
break;
}

char *token = strtok(command, " ");
if (strcmp(token, "count") == 0) {
token = strtok(NULL, " ");
if (token == NULL) {
printf("Error: Invalid command format\n");
continue;
}

char *filename = strtok(NULL, " ");
if (filename == NULL) {
printf("Error: Invalid command format\n");
continue;
}

if (strcmp(token, "c") == 0) {
int count = count_characters(filename);

if (count != -1) {
printf("Number of characters in file %s: %d\n", filename, count);
}
} else if (strcmp(token, "w") == 0) {
int count = count_words(filename);
if (count != -1) {
printf("Number of words in file %s: %d\n", filename, count);
}
} else if (strcmp(token, "l") == 0) {
int count = count_lines(filename);
if (count != -1) {
printf("Number of lines in file %s: %d\n", filename, count);
}
} else {
printf("Error: Invalid count command format\n");
}
} else {
system(command);
}
}

return 0;
}
=======================================================================================================================================

Slip12
A)Write a C program to send SIGALRM signal by child process to parent process and
parent processmake a provision to catch the signal and display alarm is fired. (Use Kill, fork, signal
and sleep systemcall)
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
void signal_handler(int sig)
{if (sig == SIGALRM) {
printf("Alarm is fired!\n");
}
}

int main() {
pid_t pid = fork();

if (pid == -1) {
printf("Error: Failed to create child process\n");
exit(EXIT_FAILURE);
}

if (pid == 0) {
// Child process
sleep(5); // Wait for 5 seconds
kill(getppid(), SIGALRM); // Send SIGALRM signal to parent process
exit(EXIT_SUCCESS);
} else {
// Parent process
signal(SIGALRM, signal_handler); // Register signal handler for SIGALRM
while(1) {
// Do nothing, wait for signal to be caught
}
}

return 0;
}

==========================================================================================================================================

Slip12
Q2B): Write a C program to display all the files from current directory and its subdirectory
whose size is greater than ’n’ Bytes Where n is accepted from user through command.
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dirent.h>
#include <sys/stat.h>

void display_files(char* dir_path, int min_size) {
DIR* dir = opendir(dir_path);
if (dir == NULL) {
printf("Error: Failed to open directory '%s'\n", dir_path);
exit(EXIT_FAILURE);
}

struct dirent* dp;
while ((dp = readdir(dir)) != NULL) {
if (strcmp(dp->d_name, ".") == 0 || strcmp(dp->d_name, "..") == 0) {
// Ignore current and parent directories
continue;
}

char path[1024];
snprintf(path, sizeof(path), "%s/%s", dir_path, dp->d_name);

struct stat st;
if (stat(path, &st) == -1) {
printf("Error: Failed to get file information for '%s'\n", path);

continue;
}

if (S_ISDIR(st.st_mode)) {
// Recurse into subdirectory
display_files(path, min_size);
} else if (S_ISREG(st.st_mode)) {
// Regular file
if (st.st_size > min_size) {
printf("%s (Size: %ld bytes)\n", path, st.st_size);
}
}
}

closedir(dir);
}

int main(int argc, char** argv)
{if (argc != 3) {
printf("Usage: %s <directory> <minimum size in bytes>\n", argv[0]);
exit(EXIT_FAILURE);
}

char* dir_path = argv[1];
int min_size = atoi(argv[2]);

display_files(dir_path, min_size);

return 0;
}

====================================================================================================================================
Slip13
Q2a: Write a C program that redirects standard output to a file output.txt. (use of dup andopen
system call)
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
int fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, S_IRUSR |
S_IWUSR);if (fd == -1) {
perror("Failed to open output file");
exit(EXIT_FAILURE);
}

int stdout_copy =
dup(STDOUT_FILENO);if (stdout_copy
== -1) {
perror("Failed to duplicate stdout file descriptor");
exit(EXIT_FAILURE);
}

int result = dup2(fd,
STDOUT_FILENO);if (result == -1) {
perror("Failed to redirect stdout to file");
exit(EXIT_FAILURE);
}

printf("This output is redirected to a file.\n");

fflush(stdout);
result = dup2(stdout_copy, STDOUT_FILENO);

if (result == -1) {
perror("Failed to restore stdout");
exit(EXIT_FAILURE);
}

printf("This output goes to the original stdout.\n");

close(fd);

return 0;
}

=========================================================================================
B): Write a C program which creates a child process and child process catches a signal
SIGHUP, SIGINT and SIGQUIT. The Parent process send a SIGHUP or SIGINT signal after every 3
seconds, at the end of 15 second parent send SIGQUIT signal to child and child terminates by
displaying message "My Papa has Killed me!!!”

Solution:
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

void sighandler(int signum) {
switch (signum) {
case SIGHUP:
printf("Child caught SIGHUP signal\n");
break;
case SIGINT:
printf("Child caught SIGINT signal\n");
break;
case SIGQUIT:

printf("My Papa has Killed me!!!\n");
exit(EXIT_SUCCESS);
default:
printf("Child caught unknown signal\n");
break;
}
}

int main() {
pid_t pid = fork();

if (pid < 0) {
perror("Failed to fork child process");
exit(EXIT_FAILURE);
} else if (pid == 0) {
signal(SIGHUP, sighandler);
signal(SIGINT, sighandler);
signal(SIGQUIT, sighandler);
while (1) {
sleep(1);
}
} else {
int i;
for (i = 0; i < 5; i++) {
sleep(3);
if (i % 2 == 0) {
kill(pid, SIGHUP);
} else {
kill(pid, SIGINT);
}
}

kill(pid, SIGQUIT);
wait(NULL);
}

return 0;
}
======================================================================================================================================

Slip14
A): Write a C program to create an unnamed pipe. Write following three messages to pipe
and display it. Message1 = “Hello World” Message2 = “Hello SPPU” Message3 = “Linux is Funny”

Solution:
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define MSGSIZE 80

int main() {
int fd[2], n;
char buffer[MSGSIZE];

if (pipe(fd) < 0) {
perror("Failed to create pipe");
exit(EXIT_FAILURE);
}

write(fd[1], "Hello World", MSGSIZE);
write(fd[1], "Hello SPPU", MSGSIZE);
write(fd[1], "Linux is Funny", MSGSIZE);
close(fd[1]);

while ((n = read(fd[0], buffer, MSGSIZE)) > 0) {
printf("%s\n", buffer);
}
return 0;
}
========================================================================================

B): Write a C program to display all the files from current directory which are created in a
particular month.
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>
#include <time.h>
int main(int argc, char *argv[]) {
DIR *dir;
struct dirent
*entry;struct stat
file_stat; char
*month_str;
int month,
year;if (argc !=
3) {
printf("Usage: %s <month> <year>\n", argv[0]);
exit(EXIT_FAILURE);
}
month = atoi(argv[1]);
year = atoi(argv[2]);
switch (month) {
case 1:
month_str = "Jan";

break;
case 2:
month_str = "Feb";
break;
case 3:
month_str = "Mar";
break;
case 4:
month_str = "Apr";
break;
case 5:
month_str = "May";
break;
case 6:
month_str = "Jun";
break;
case 7:
month_str = "Jul";
break;
case 8:
month_str = "Aug";
break;
case 9:
month_str = "Sep";
break;
case 10:
month_str = "Oct";
break;
case 11:
month_str = "Nov";
break;

case 12:
month_str = "Dec";
break;
default:
printf("Invalid month number\n");
exit(EXIT_FAILURE);
}
dir =
opendir(".");if
(dir == NULL) {
perror("opendir");
exit(EXIT_FAILURE);
}
printf("Files created in %s %d:\n", month_str, year);
while ((entry = readdir(dir)) != NULL) {
if (stat(entry->d_name, &file_stat) == -1) {
perror("stat");
continue;
}
if ((file_stat.st_mode & S_IFMT) == S_IFREG &&
localtime(&file_stat.st_ctime)->tm_mon == month - 1 &&
localtime(&file_stat.st_ctime)->tm_year == year - 1900) {
printf("%s\n", entry->d_name);
}
}
closedir(dir
);return 0;
}

================================================================
Slip15
A): Write a C program to Identify the type (Directory, character device, Block device, Regular
file, FIFO or pipe, symbolic link or socket) of given file using stat() system call.
Solution:

#include <stdio.h>
#include <sys/stat.h>
#include <stdlib.h>
int main(int argc, char *argv[]) {
struct stat fileStat;
if (argc != 2) {
fprintf(stderr, "Usage: %s <filename>\n", argv[0]);
exit(EXIT_FAILURE);
}
if (stat(argv[1], &fileStat) < 0) {
fprintf(stderr, "Error: Unable to stat file %s\n", argv[1]);
exit(EXIT_FAILURE);
}
if (S_ISDIR(fileStat.st_mode)) {
printf("%s is a directory\n", argv[1]);
} else if (S_ISCHR(fileStat.st_mode)) {
printf("%s is a character device\n", argv[1]);
} else if (S_ISBLK(fileStat.st_mode)) {
printf("%s is a block device\n", argv[1]);
} else if (S_ISREG(fileStat.st_mode)) {
printf("%s is a regular file\n", argv[1]);
} else if (S_ISFIFO(fileStat.st_mode)) {
printf("%s is a FIFO/pipe\n", argv[1]);
} else if (S_ISLNK(fileStat.st_mode)) {
printf("%s is a symbolic link\n", argv[1]);
} else if (S_ISSOCK(fileStat.st_mode)) {
printf("%s is a socket\n", argv[1]);
} else {
printf("%s is of unknown type\n", argv[1]);
}

return 0;
}

=================================================================================
B): Write a C program which creates a child process and child process catches a signal
SIGHUP, SIGINT and SIGQUIT. The Parent process send a SIGHUP or SIGINT signal after every 3
seconds, at the end of 15 second parent send SIGQUIT signal to child and child terminates by
displaying message "My Papa has Killed me!!
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
void sighandler(int signum) {
switch (signum) {
case SIGHUP:
printf("Child caught SIGHUP signal\n");
break;
case SIGINT:
printf("Child caught SIGINT signal\n");
break;
case SIGQUIT:
printf("My Papa has Killed me!!!\n");
exit(EXIT_SUCCESS);
default:
printf("Child caught unknown signal\n");
break;
}
}
int main() {
pid_t pid =
fork();if (pid <
0) {
perror("Failed to fork child process");

exit(EXIT_FAILURE);
} else if (pid == 0) {
signal(SIGHUP, sighandler);
signal(SIGINT, sighandler);
signal(SIGQUIT, sighandler);
while (1) {
sleep(1);
}
} else {
int i;
for (i = 0; i < 5; i++) {
sleep(3);
if (i % 2 == 0) {
kill(pid, SIGHUP);
} else {
kill(pid, SIGINT);
}
}
kill(pid, SIGQUIT);
wait(NULL);
}

return 0;
}

================================================================
Slip16
A): Write a C program that catches the ctrl-c (SIGINT) signal for the first time and display the
appropriate message and exits on pressing ctrl-c again.
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

volatile sig_atomic_t signal_count = 0;
void handle_signal(int sig) {
if (sig == SIGINT) {
if (signal_count == 0) {
printf("\nCaught SIGINT signal. Press Ctrl-C again to exit.\n");
signal_count++;
} else {
printf("\nExiting...\n");
exit(0);
}
}
}
int main() {
signal(SIGINT, handle_signal);
while (1) {
// Do nothing, wait for signal
}
return 0;
}

====================================================================================
B): Write a C program to implement the following unix/linux command on current directory
ls –l > output.txt DO NOT simply exec ls -l > output.txt or system command from the program.

Solution:
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/stat.h>
int main() {

int fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR | S_IRGRP |
S_IROTH);
if (fd < 0) {
perror("open");
return 1;
}
int stdout_copy =
dup(STDOUT_FILENO);if (stdout_copy <
0) {
perror("dup");
close(fd);
return 1;
}
if (dup2(fd, STDOUT_FILENO) < 0) {
perror("dup2");
close(fd);
return 1;
}
execlp("ls", "ls", "-l", (char *)NULL);
if (dup2(stdout_copy, STDOUT_FILENO) < 0) {
perror("dup2");
close(fd);
return 1;
}
close(stdout_copy
);close(fd);
return 0;
}
======================================================================================================
Slip17
A: Write a C program to display the given message ‘n’ times. (make a use of setjmp and
longjmp system call)
Solution:
#include <stdio.h>

#include <setjmp.h>
jmp_buf env;
void display_message(char *msg, int n)
{int i;
for (i = 0; i < n; i++) {
printf("%s\n", msg);
if (i == n-1)
longjmp(env, 1);
}
}
int main()
{int n;
printf("Enter the number of times to display the message: ");
scanf("%d", &n);
if (setjmp(env) == 0) {
display_message("Hello, world!", n);
}
else {
printf("Displaying the message %d times completed.\n", n);
}
return 0;
}

================================================================
B): Write a C program to display all the files from current directory which are created in a
particular month.
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>
#include <time.h>

int main(int argc, char *argv[]) {
DIR *dir;
struct dirent
*entry;struct stat
file_stat; char
*month_str;
int month,
year;if (argc !=
3) {
printf("Usage: %s <month> <year>\n", argv[0]);
exit(EXIT_FAILURE);
}
month = atoi(argv[1]);
year = atoi(argv[2]);
switch (month) {
case 1:
month_str = "Jan";
break;
case 2:
month_str = "Feb";
break;
case 3:
month_str = "Mar";
break;
case 4:
month_str = "Apr";
break;
case 5:
month_str = "May";
break;
case 6:
month_str = "Jun";
break;

case 7:
month_str = "Jul";
break;
case 8:
month_str = "Aug";
break;
case 9:
month_str = "Sep";
break;
case 10:
month_str = "Oct";
break;
case 11:
month_str = "Nov";
break;
case 12:
month_str = "Dec";
break;
default:
printf("Invalid month number\n");
exit(EXIT_FAILURE);
}
dir =
opendir(".");if
(dir == NULL) {
perror("opendir");
exit(EXIT_FAILURE);
}
printf("Files created in %s %d:\n", month_str, year);
while ((entry = readdir(dir)) != NULL) {
if (stat(entry->d_name, &file_stat) == -1) {
perror("stat");

continue;
}
if ((file_stat.st_mode & S_IFMT) == S_IFREG &&
localtime(&file_stat.st_ctime)->tm_mon == month - 1 &&
localtime(&file_stat.st_ctime)->tm_year == year - 1900) {
printf("%s\n", entry->d_name);
}
}
closedir(dir
);return 0;
}

OR//
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>
#include <time.h>

int main()
{
char dir_name[256];
struct dirent *entry;
struct stat file_stat;
DIR *dir;
int month;
printf("Enter the month number (1-12): ");
scanf("%d", &month);
// Get the current directory
if (getcwd(dir_name, sizeof(dir_name)) == NULL)
{

perror("getcwd() error");
exit(EXIT_FAILURE);
}
// Open the directory
dir =
opendir(dir_name);if
(dir == NULL)
{
perror("opendir() error");
exit(EXIT_FAILURE);
}
// Loop through the directory entries
while ((entry = readdir(dir)) != NULL)
{
char file_name[256];
sprintf(file_name, "%s/%s", dir_name, entry->d_name);
// Get the file status
if (stat(file_name, &file_stat) < 0)
{
perror("stat() error");
continue;
}
// Check if the file was created in the specified month
struct tm *tm_info = localtime(&file_stat.st_ctime);
if (tm_info->tm_mon + 1 == month)
{
printf("%s\n", entry->d_name);
}
}
closedir(dir
);return 0;
}

=================================================================================================
Slip18
A): Write a C program to display the last access and modified time of a given file.
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <time.h>
int main()
{
char
file_name[256];
struct stat file_stat;
printf("Enter the file name:
");scanf("%s", file_name);
// Get the file status
if (stat(file_name, &file_stat) < 0)
{
perror("stat() error");
exit(EXIT_FAILURE);
}
// Print the last access and modified time
printf("Last access time: %s", ctime(&file_stat.st_atime));
printf("Last modified time: %s", ctime(&file_stat.st_mtime));
return 0;
}

==============================================================
B): Write a C program to implement the following unix/linux command (use fork, pipe and
exec system call). Your program should block the signal Ctrl-C and Ctrl-\ signal during the execution.
ls –l | wc –l
Solution:
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

#include <sys/wait.h>
#include <signal.h>
int main()
{
int fd[2]; // pipe file descriptors
pid_t pid1, pid2; // child process IDs
// Block the SIGINT and SIGQUIT
signalssigset_t block_mask;
sigemptyset(&block_mask);
sigaddset(&block_mask, SIGINT);
sigaddset(&block_mask, SIGQUIT);
sigprocmask(SIG_BLOCK, &block_mask, NULL);
// Create the
pipeif (pipe(fd)
< 0)
{
perror("pipe() error");
exit(EXIT_FAILURE);
}
// Fork the first child
processif ((pid1 = fork()) <
0)
{
perror("fork() error");
exit(EXIT_FAILURE);
}
else if (pid1 == 0)
{
// Child process 1: execute "ls -l" and write the output to the pipe
// Close the read end of the pipe
close(fd[0]);
// Redirect the standard output to the write end of the pipe
dup2(fd[1], STDOUT_FILENO);

// Execute "ls -l"
execlp("ls", "ls", "-l", NULL);
perror("execlp() error");
exit(EXIT_FAILURE);
}
// Fork the second child
processif ((pid2 = fork()) < 0)
{
perror("fork() error");
exit(EXIT_FAILURE);
}
else if (pid2 == 0)
{
// Child process 2: read from the pipe and execute "wc -l"
// Close the write end of the pipe
close(fd[1]);
// Redirect the standard input to the read end of the pipe
dup2(fd[0], STDIN_FILENO);
// Execute "wc -l"
execlp("wc", "wc", "-l", NULL);

perror("execlp() error");
exit(EXIT_FAILURE);
}

// Parent process: wait for the child processes to
completeclose(fd[0]);
close(fd[1]);
waitpid(pid1, NULL, 0);
waitpid(pid2, NULL, 0);

// Unblock the SIGINT and SIGQUIT signals
sigprocmask(SIG_UNBLOCK, &block_mask,
NULL);

return 0;
}
