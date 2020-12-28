# Modify environment variables of a running process

Use GDB to attach to the process

```bash
# gdb -p <Process-PID>
```

Set environment variable
```
(gdb) p setenv("Key", "Value")
```

Exit
```
(gdb) q
```

### Sample C Program
```c
/*
 * setlive_env.c
 */
#include <stdlib.h>
#include <stdio.h>

int
main(int argc, char *argv[])
{
        int     i = 0;
        char    *env_val;

        for (i = 0; i < 10; i++) {
                env_val = getenv("Env_Key");
                printf("Env Val for TZ: %s\n", env_val);
                sleep(5);
        }

        return (0);
}
```

Build the code:
```bash
# gcc setenv_live.c -o setenv_live.o
```

Run the code:
```bash
# ./setenv_live.o
```

Initial Output:
```bash
# ./setenv_live.o
Env Val for TZ: (null)
Env Val for TZ: (null)
```

Run gdb to attach to the process
```bash
# gdb -p <PID-OF-ABOVE-PROCESS>
```

Set env value and exit
```bash
(gdb) p setenv("Env_Key", "New_Value")
$1 = 0
(gdb) q
A debugging session is active.

        Inferior 1 [process 11414] will be detached.

Quit anyway? (y or n) y
Detaching from program: /root/setenv_live.o, process 11414
```

Output after setting env variable:
```bash
# ./setenv_live.o
Env Val for TZ: (null)
Env Val for TZ: (null)
Env Val for TZ: New_Value
Env Val for TZ: New_Value
Env Val for TZ: New_Value
Env Val for TZ: New_Value
Env Val for TZ: New_Value
Env Val for TZ: New_Value
Env Val for TZ: New_Value
Env Val for TZ: New_Value
```
