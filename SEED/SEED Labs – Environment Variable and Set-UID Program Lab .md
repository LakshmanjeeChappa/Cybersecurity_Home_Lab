Task 1: Manipulating Environment Variables

Commands Used:

printenv
printenv PWD
env | grep PWD
export MYVAR=hello
printenv MYVAR
unset MYVAR
printenv MYVAR



What i did :

First, I used the printenv and env commands to display all environment variables in the system. Then, I specifically checked the PWD variable to see my current working directory.

After that, I created a new environment variable using the export command and verified it. Finally, I removed the variable using the unset command.

What I Observed:

printenv and env both showed environment variables
PWD displayed my current directory
After using export MYVAR=hello, the variable appeared
After using unset MYVAR, the variable disappeared

What I Observed:

printenv and env both showed environment variables
PWD displayed my current directory
After using export MYVAR=hello, the variable appeared
After using unset MYVAR, the variable disappeared


Task 2: Environment Variables and fork()

 Commands Used :

gcc myprintenv.c -o myprintenv
./myprintenv > child_output.txt
./myprintenv > parent_output.txt
diff child_output.txt parent_output.txt

 What I Did :

I created a program that uses fork() to create a child process.
First, I printed environment variables from the child process and saved the output.

Then I modified the program to print environment variables from the parent process instead and saved that output as well.

Finally, I used the diff command to compare both outputs.

 What I Observed:

Both child and parent outputs were almost identical
The diff command showed little to no difference

 Explanation: 

When a process calls fork(), the child process is created as a copy of the parent process. This includes a copy of all environment variables.

Since no changes were made after the fork, both processes had the same environment. That is why the outputs from the child and parent processes were nearly identical.



Task 3: Environment Variables and execve()

 Commands Used:

gcc myenv.c -o myenv
./myenv

(After modifying code)

gcc myenv.c -o myenv
./myenv

 What I Did:


I compiled and ran the program myenv.c, which uses the execve() function to execute /usr/bin/env.

First, the program was written with execve() passing NULL as the environment parameter. I ran the program and observed the output.

Then, I modified the code to pass environ instead of NULL, recompiled the program, and ran it again.

What I Observed:

When NULL was used → no environment variables were printed
When environ was used → all environment variables were displayed

Explanation:


The execve() function does not automatically pass environment variables to the new program. If NULL is provided, the new program runs without any environment variables.

When environ is passed, it explicitly provides the current process’s environment to the new program. As a result, all environment variables are available and displayed.

This shows that environment variables must be manually passed when using execve().


Task 4: Environment Variables and system()
 Commands Used:

gcc sysenv.c -o sysenv
./sysenv

 What I Did:

I created a program that uses the system() function to execute the command /usr/bin/env. Then I compiled and ran the program to observe how environment variables behave when using system().

 What I Observed:

The program printed all environment variables
Variables like PATH, HOME, and USER were visible

 Explanation:

The system() function executes commands through the shell (/bin/sh). The shell automatically inherits environment variables from the parent process.

Because of this, when /usr/bin/env is executed using system(), it has access to all environment variables without needing to pass them explicitly.

This behavior is different from execve(), where environment variables must be manually passed.


Task 5: Environment Variables and Set-UID Programs

 Commands Used:

gcc foo.c -o foo
sudo chown root foo
sudo chmod 4755 foo

export PATH=abc
export LD_LIBRARY_PATH=test
export MYVAR=hello123

./foo

 What I Did:

I created a program that prints all environment variables. Then I compiled it and changed its ownership to root and enabled SetUID permissions.

After that, I set three environment variables in my shell:

PATH
LD_LIBRARY_PATH
A custom variable MYVAR

Finally, I executed the SetUID program to check which variables were passed to it.

 What I Observed:

PATH was visible in the output
MYVAR was not visible
LD_LIBRARY_PATH was not visible

 Explanation:

SetUID programs do not trust all environment variables from the user because they run with elevated privileges. Allowing all variables could lead to security vulnerabilities.

Sensitive variables like LD_LIBRARY_PATH are removed because they can be used to load malicious libraries and manipulate program behavior.

Custom variables like MYVAR may also be filtered.

However, variables like PATH are still preserved, which can be risky because attackers can manipulate it to execute malicious commands.



Task 6: The PATH Environment Variable and Set-UID Programs

 Commands Used:

gcc path_attack.c -o path_attack
sudo chown root path_attack
sudo chmod 4755 path_attack

nano ls
chmod +x ls

export PATH=.:$PATH

./path_attack

(Optional step from lab)

sudo ln -sf /bin/zsh /bin/sh
./path_attack

What I Did:

I created a program that uses system("ls") and made it a SetUID root program.

Then, I created a fake ls script that prints a custom message and shows the current user using whoami. I made this script executable and placed it in the current directory.

After that, I modified the PATH variable to include the current directory at the beginning so that my fake ls would be executed instead of the real /bin/ls.

Finally, I ran the SetUID program to observe the behavior.

 What I Observed:

My fake ls script was executed instead of the real ls

Output showed:

HACKED
lakshman
Even after linking /bin/sh to /bin/zsh, the program still ran with my user privileges instead of root

 Explanation:

The system() function executes commands using the shell, and the shell uses the PATH variable to locate executables. By modifying PATH, I was able to trick the program into running my malicious script instead of the intended command.

However, modern Linux systems implement security protections that prevent privilege escalation in SetUID programs. Even though the program is owned by root, the system drops elevated privileges when executing commands through the shell.

As a result, the malicious code was executed, but it ran with normal user privileges instead of root.



Task 7: The LD_PRELOAD Environment Variable and Set-UID Programs

Commands Used:

gcc -fPIC -g -c mylib.c
gcc -shared -o libmylib.so.1.0.1 mylib.o -lc

export LD_PRELOAD=./libmylib.so.1.0.1

gcc myprog.c -o myprog
./myprog

sudo chown root myprog
sudo chmod 4755 myprog

./myprog

 What I Did:

I created a shared library that overrides the sleep() function. Then I compiled it as a dynamic library.

After that, I set the LD_PRELOAD environment variable to load my custom library before system libraries.

Then I created and ran a program that calls sleep() to observe how the behavior changes.

Finally, I converted the program into a SetUID root program and ran it again.

 What I Observed:

When running as a normal program, the output was:

I am not sleeping!
After making the program SetUID root, the output disappeared and the message was no longer shown

 Explanation:

The LD_PRELOAD variable allows users to load custom shared libraries before the system libraries. This makes it possible to override existing functions, such as sleep(), and change program behavior.

However, in SetUID programs, Linux disables LD_PRELOAD for security reasons. Allowing it would let attackers inject malicious code and execute it with root privileges.

Because of this protection, the custom library was ignored when the program was executed as a SetUID program, and the attack no longer worked.



Task 8: Invoking External Programs Using system() vs execve()

Commands Used:

gcc catall.c -o catall
sudo chown root catall
sudo chmod 4755 catall

./catall /etc/hostname
./catall "/etc/hostname; whoami"

(After modifying code)

gcc catall.c -o catall
sudo chown root catall
sudo chmod 4755 catall

./catall "/etc/hostname; whoami"

 What I Did:

I compiled the given catall.c program and made it a SetUID root program.

First, I ran the program normally to display the contents of a file.

Then, I tried a malicious input by adding ; whoami to inject an extra command.

After that, I modified the program by replacing system() with execve(), recompiled it, and tested the same attack again.

 What I Observed:

With system():
Normal input worked correctly

Malicious input:

/etc/hostname; whoami

produced output:

root
With execve():
The same attack failed

Output showed:

No such file or directory

Explanation:

The system() function executes commands through a shell. The shell interprets special characters like ; as command separators. This allowed me to inject an additional command (whoami), which was executed with root privileges because the program is SetUID.

This is a classic command injection vulnerability.

On the other hand, execve() does not use a shell. It directly executes the specified program with given arguments. As a result, special characters are not interpreted, and the entire input is treated as a single argument.

Because of this, the attack failed when using execve(), making it a safer option.



Task 9: Capability Leaking

 Commands Used:

gcc cap_leak.c -o cap_leak
sudo chown root cap_leak
sudo chmod 4755 cap_leak

sudo touch /etc/zzz
sudo chmod 644 /etc/zzz

./cap_leak

What I Did:


I compiled the cap_leak.c program and made it a SetUID root program.

Then, I created a file /etc/zzz with root ownership and appropriate permissions.

After that, I executed the program as a normal user to observe how it behaves when accessing a protected file.

What I Observed:

The output was:

fd is 3

This means the file was successfully opened and assigned a file descriptor.

 Explanation

The program is a SetUID root program, so it runs with root privileges even when executed by a normal user. Because of this, it was able to open the protected file /etc/zzz, which is normally not accessible to regular users.

The file descriptor (fd = 3) represents an open handle to the file. This is a security issue because the program does not close the file or drop privileges after opening it.

This results in capability leaking, where a non-privileged user gains access to a privileged resource through the program.

Even if the program later drops its privileges, the open file descriptor can still be used to access or modify the file.
