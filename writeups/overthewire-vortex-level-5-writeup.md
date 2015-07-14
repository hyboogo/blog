# Overthewire Vortex Level 5 Writeup

Simple one. The hash is in the source file:

```c
if(memcmp(buf, "\x15\x5f\xb9\x5d\x04\x28\x7b\x75\x7c\x99\x6d\x77\xb5\xea\x51\xf7", 16) == 0){
                printf("You got the right password, congrats!\n");
                setresgid(getegid(), getegid(), getegid());
                setresuid(geteuid(), geteuid(), geteuid());
                system("/bin/sh");
        } else {
                usleep(500);
                printf("Incorrect password\n");
        }
```

Using Google you can get the right password.