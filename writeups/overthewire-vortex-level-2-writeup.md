# Overthewire Vortex Level 2 Writeup

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>


int main(int argc, char **argv)
{
        char *args[] = { "/bin/tar", "cf", "/tmp/ownership.$$.tar", argv[1], argv[2], argv[3] };
        execv(args[0], args);
}
```

It is noteworthy that bash doesn't interpret `$$` here to PID, since it execv '/bin/tar' here, not `/bin/sh`. So just use quotes to prevent `$$` from being parsed:

```
$ tar xf '/tmp/ownership.$$.tar' -O
```