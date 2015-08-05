# Overthewire Vortex Level 7 Writeup

The main emphasis of this level is reversing CRC32. I found a better [article](http://www.danielvik.com/2010/10/calculating-reverse-crc.html) other than the one given. The reversing algo is as follow:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define POLY 0x04C11DB7

unsigned int crctable[256];

unsigned int reverse_bits(unsigned int num)
{
  unsigned int NO_OF_BITS = sizeof(num) * 8;
  unsigned int reverse_num = 0, i, temp;

  for (i = 0; i < NO_OF_BITS; i++)
  {
    temp = (num & (1 << i));
    if (temp)
      reverse_num |= (1 << ((NO_OF_BITS - 1) - i));
  }

  return reverse_num;
}

void print_bits(unsigned int x)
{
  int i;
  for (i = 8 * sizeof(x) - 1; i >= 0; i--) {
    (x & (1 << i)) ? putchar('1') : putchar('0');
  }
  printf("\n");
}

uint32_t crc32(char * buf, int len)
{
  char * str;
  uint32_t a1 = 0;
  for (str = buf; len > 0; str++, --len) {
    uint8_t entry = a1 ^ *str;
    a1 = (a1 >> 8) ^ crctable[entry];
  }
  return a1;
}

uint8_t abc(uint32_t s, int i)
{
  return s >> (i * 8) & 0xff;
}

uint32_t get_table_entry(uint8_t b)
{
  int i;
  for (i = 0; i < 256; i++)
  {
    if (abc(crctable[i], 3) == b)
      break;
  }
  // printf("found entry %d\n", i);
  return i;
}


int main()
{
  printf("Calculating the table...\n");

  // print_bits(POLY);

  unsigned int rev_poly = reverse_bits(POLY);
  // print_bits(rev_poly);


  unsigned int j = 0;
  for (unsigned int i = 0; i < 256; i++)
  {
    j = i;
    for (unsigned int k = 0; k < 8; k++)
    {
      if ((j & 0x01) == 1)
        j = rev_poly ^ (j >> 1);
      else
        j >>= 1;
    }
    crctable[i] = j;
    // printf("%d: %x\n", i, j);
  }

  char * msg = "abcd";
  uint32_t before = crc32(msg, strlen(msg)), after = 0xee95cae1;
  printf("CRC: %x\n", before);

  uint8_t top_bytes[4];
  top_bytes[0] = abc(after, 3);
  top_bytes[1] = abc(after, 2);
  top_bytes[2] = abc(after, 1);
  top_bytes[3] = abc(after, 0);

  uint32_t index[4];
  index[0] = get_table_entry(top_bytes[3]);
  index[1] = get_table_entry(top_bytes[2] ^
                             abc(crctable[index[0]], 2));
  index[2] = get_table_entry(top_bytes[1] ^
                             abc(crctable[index[0]], 1) ^
                             abc(crctable[index[1]], 2));
  index[3] = get_table_entry(top_bytes[0] ^
                             abc(crctable[index[0]], 0) ^
                             abc(crctable[index[1]], 1) ^
                             abc(crctable[index[2]], 2));

  uint8_t pads[4];
  pads[0] = index[3] ^ abc(before, 0);
  pads[1] = index[2] ^ abc(before, 1) ^
            abc(crctable[index[3]], 0);
  pads[2] = index[1] ^ abc(before, 2) ^
            abc(crctable[index[3]], 1) ^
            abc(crctable[index[2]], 0);
  pads[3] = index[0] ^ abc(before, 3) ^
            abc(crctable[index[3]], 2) ^
            abc(crctable[index[2]], 1) ^
            abc(crctable[index[1]], 0);

  printf("Padding...\n");
  for (int i = 0; i < 4; i++)
  {
    printf("%x\n", pads[i]);
  }
  return 0;
}
```

Padding the byte one by one to the end of the string you want reveals the string which produces the CRC checksum you need.
The next part is a standard buffer overflow. Just overwrite the return address to an env to gain access to the next level.