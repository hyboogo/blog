# Overthewire Vortex Level 0 Writeup

```python
#!/usr/bin/env python

from socket import *
from struct import *

conn = socket(AF_INET, SOCK_STREAM)
conn.connect(("vortex.labs.overthewire.org" , 5842))

sum = 0;

for i in range(4):
    data = conn.recv(4)
    sum += unpack("<I", data)[0]

conn.send(pack("<I", sum))
# print conn.recv(1024)
conn.close()
```
