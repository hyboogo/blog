# ASIS Quals CTF 2015: Broken Heart Writeup

First download the file and check the file type:

```bash
$ wget http://tasks.asis-ctf.ir/myheart_7cb6daec0c45b566b9584f98642a7123
--2015-05-16 09:44:53--  http://tasks.asis-ctf.ir/myheart_7cb6daec0c45b566b9584f98642a7123
Resolving tasks.asis-ctf.ir (tasks.asis-ctf.ir)... 79.127.125.110
Connecting to tasks.asis-ctf.ir (tasks.asis-ctf.ir)|79.127.125.110|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2418508 (2.3M) [application/octet-stream]
Saving to: ‘myheart_7cb6daec0c45b566b9584f98642a7123’
$ mv myheart_7cb6daec0c45b566b9584f98642a7123 myheart
$ file myheart
myheart: pcap-ng capture file - version 1.0
```

Hmm, it is a PCAP file. They provide a live capture file. Let's examine the file to see what it's contained.
I recommend you guys to use [Dshell](https://github.com/USArmyResearchLab/Dshell), since wireshark can not reassamble HTTP fragmented packets to generate the RAW data, you have to write a small script to do that for you. Or you can use tshark, but I could not find the right command to do that, if someone can tell me, I would be appreciated.

After installation I fire Dshell:

```bash
$ ./dshell
Dshell> decode -d web ./myheart
web 2015-04-12 06:15:23  192.168.221.128:54391 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:15:34  192.168.221.128:54392 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:15:42  192.168.221.128:54393 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:15:52  192.168.221.128:54394 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:15:59  192.168.221.128:54397 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:06  192.168.221.128:54398 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:12  192.168.221.128:54399 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:18  192.168.221.128:54400 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:25  192.168.221.128:54401 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:32  192.168.221.128:54402 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:38  192.168.221.128:54403 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:44  192.168.221.128:54404 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:50  192.168.221.128:54405 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:16:57  192.168.221.128:54406 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:04  192.168.221.128:54407 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:10  192.168.221.128:54408 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:17  192.168.221.128:54409 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:26  192.168.221.128:54410 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:31  192.168.221.128:54411 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:38  192.168.221.128:54412 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:44  192.168.221.128:54413 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:51  192.168.221.128:54414 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **
web 2015-04-12 06:17:56  192.168.221.128:54415 <-    87.107.124.13:80    ** GET a.asis.io/LoiRLUoq HTTP/1.1                                                  // 206 Partial Content  2015-04-11 10:34:58 **>>>>>>>>>>>>>>>>>>>>>>>
```

A bunch of HTTP partical contents. Let's reassamble the RAW data.

```bash
Dshell> decode -d rip-http --bpf "tcp and port 80" ./myheart
rip-http 2015-04-12 06:15:23  192.168.221.128:54391 <-    87.107.124.13:80    ** New file: LoiRLUoq (a.asis.io/LoiRLUoq) **
 --> Range: 1080486 - 1345387
 --> Range: 986065 - 1150874
 --> Range: 1397670 - 1593207
 --> Range: 337541 - 500782
 --> Range: 2001846 - 2202904
 --> Range: 467298 - 648929
 --> Range: 1507903 - 1694032
 --> Range: 552789 - 781321
 --> Range: 1276598 - 1432659
 --> Range: 1888311 - 1938509
 --> Range: 13 - 179538
 --> Range: 2106781 - 2347915
 --> Range: 1540792 - 1639406
 --> Range: 145550 - 198027
 --> Range: 905781 - 1032111
 --> Range: 1987909 - 2044321
 --> Range: 694834 - 905770
 --> Range: 27943 - 132132
 --> Range: 1774960 - 1959007
 --> Range: 892465 - 1067354
 --> Range: 1904693 - 2059434
 --> Range: 188923 - 359924
 --> Range: 1672374 - 1872648
CTRL_D
$ file LoiRLUoq
LoiRLUoq: data
$ xxd LoiRLUoq | head -n5
00000000: 0000 0000 0000 0000 0000 0000 0048 4452  .............HDR
00000010: 0000 0780 0000 04b0 0806 0000 001a 3057  ..............0W
00000020: f600 0000 0970 4859 7300 000b 1300 000b  .....pHYs.......
00000030: 1301 009a 9c18 0000 0a4f 6943 4350 5068  .........OiCCPPh
00000040: 6f74 6f73 686f 7020 4943 4320 7072 6f66  otoshop ICC prof
```

It looks like a PNG file, since that `pHYs` is a chunk signature of PNG. I think there may exist some tools to examine the chunk signature of files. They should not only regonize the file header but also the chunks. That can save some time when you encounter some rare file types. Let me know if you find one.
It is wired that [wikipedia](http://en.wikipedia.org/wiki/List_of_file_signatures) suggests the header of a PNG file should contain only '89 50 4E 47 0D 0A 1A 0A'. These are only 8 bytes. But according to the captured file we need 13 bytes. Download a random PNG image file from Internet and then check the header:

```bash
$ file cat.png ; xxd cat.png | head -n5
cat.png: PNG image data, 276 x 453, 8-bit/color RGBA, non-interlaced
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
00000010: 0000 0114 0000 01c5 0806 0000 0033 1206  .............3..
00000020: f300 0000 1974 4558 7453 6f66 7477 6172  .....tEXtSoftwar
00000030: 6500 4164 6f62 6520 496d 6167 6552 6561  e.Adobe ImageRea
00000040: 6479 71c9 653c 0000 0368 6954 5874 584d  dyq.e<...hiTXtXM>
```

Now we have a valid header which is '8950 4e47 0d0a 1a0a 0000 000d 49'. Use a hex editor to change the first 13 bytes and open the resulted image. Cheers you have the flag.

```bash
$ tesseract /vagrant/LoiRLUoq1.tiff ./outtext
Tesseract Open Source OCR Engine v3.03 with Leptonica
$ cat outtext.txt
ASIS{8bff6216084db147b3200850bc65eb16}
```
