#Problem Statement

>Bernie Sanders 2018

>[consul](https://s3.amazonaws.com/hackthevote/consul.dcdcdac48cdb5ca5bc1ec29ddc53fb554d814d12094ba0e82f84e0abef065711)

#Solution

Because this is a reversing problem without a netcat server, we can be pretty sure we are looking for `flag{something}`
The hex for flag is `66 6c 61 67` so can we keep an eye out for these values.

We first download the binary they provided and run it. We can see that all it prints out is `Poor Bernie.`

Then, we can run nm on the file to look at the symbols and data of the file.

>nm consul.dcdcdac48cdb5ca5bc1ec29ddc53fb554d814d12094ba0e82f84e0abef065711

```
0000000000601280 d b
00000000006012a0 d b0
00000000006012e0 d b1
00000000006012fc d b2
0000000000601320 d b3
0000000000601358 A __bss_start
0000000000400874 T c1
0000000000400892 T c1_
00000000004008ef T c2
0000000000400925 T c3
0000000000400828 T c4
00000000004009b3 T c5
0000000000400a3a T c55
0000000000400a09 T c8
000000000040056c t call_gmon_start
0000000000601358 b completed.6092
0000000000601260 D __data_start
0000000000601260 W data_start
0000000000400590 t deregister_tm_clones
0000000000400600 t __do_global_dtors_aux
0000000000601008 t __do_global_dtors_aux_fini_array_entry
0000000000400a77 T dont_call_me
0000000000601268 D __dso_handle
0000000000601018 d _DYNAMIC
0000000000601358 A _edata
0000000000601370 A _end
0000000000400ab2 T fake_help
0000000000400bfc T _fini
0000000000400620 t frame_dummy
0000000000601000 t __frame_dummy_init_array_entry
0000000000400f90 r __FRAME_END__
0000000000601200 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000400b16 T help
00000000004004b8 T _init
0000000000601008 t __init_array_end
0000000000601000 t __init_array_start
0000000000400c08 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000601010 d __JCR_END__
0000000000601010 d __JCR_LIST__
                 w _Jv_RegisterClasses
0000000000400b60 T __libc_csu_fini
0000000000400b70 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
0000000000601368 b m0
0000000000400b3d T main
                 U malloc@@GLIBC_2.2.5
0000000000601360 b mem
                 U printf@@GLIBC_2.2.5
                 U puts@@GLIBC_2.2.5
0000000000400ad9 T real_help
00000000004005c0 t register_tm_clones
0000000000400540 T _start
                 U strlen@@GLIBC_2.2.5
00000000004007b0 T sub_198A
00000000004006c0 T sub_41F2
000000000040064c T sub_43E8
0000000000400738 T sub_9F36
0000000000601358 D __TMC_END__
                 U usleep@@GLIBC_2.2.5
```	 

We can see that there is local data stored at `b` `b0` `b1` `b2` `b3` denoted by `d`

We can also see a multitude of interesting functions denoted by `T`
`help` `real_help` `fake_help``dont_call_me` `c55` `c8` `c5` `c3` `c2` `c1_` `c1`

Hmmm interesting... 1, 1, 2, 3, 5, 8... It seems like the fibonacci sequence to me.

We can then use gdb to call these functions and see what they do.

>gdb consul.dcdcdac48cdb5ca5bc1ec29ddc53fb554d814d12094ba0e82f84e0abef065711
>break main
>run
>call real_help()

We can confirm that the fibonacci sequence has something to do with this problem by looking at the output of `real_help`
```
Leonardo De Pisa? Who's that–The next president?
```
If you google this, fibonacci is the first result.

Then, we proceeded to calling c1, c1_, c2, c3, c5, c8 in order and find that c8 gives a segmentation fault. So we use objdump to decompile the program look at the code for the function c8

>objdump -d consul.dcdcdac48cdb5ca5bc1ec29ddc53fb554d814d12094ba0e82f84e0abef065711

```
0000000000400a09 <c8>:
  400a09:       55                      push   %rbp
  400a0a:       48 89 e5                mov    %rsp,%rbp
  400a0d:       8b 05 4d 09 20 00       mov    0x20094d(%rip),%eax        # 601360 <mem>
  400a13:       83 c0 09                add    $0x9,%eax
  400a16:       89 05 44 09 20 00       mov    %eax,0x200944(%rip)        # 601360 <mem>
  400a1c:       bf 80 12 60 00          mov    $0x601280,%edi
  400a21:       e8 12 fd ff ff          callq  400738 <sub_9F36>
  400a26:       48 89 c6                mov    %rax,%rsi
  400a29:       bf 0c 0c 40 00          mov    $0x400c0c,%edi
  400a2e:       b8 00 00 00 00          mov    $0x0,%eax
  400a33:       e8 c8 fa ff ff          callq  400500 <printf@plt>
  400a38:       5d                      pop    %rbp
  400a39:       c3                      retq
```

Hmm it's messing with the data at 0x601360 as denoted by the # comments. Let's take a look what is around that address

>x/80x 0x601320
Note: we use 0x601320 because we want to see the data before and after that address.

We can see that there are interesting values before 0x601360

````
0x601320 <b3>:  0x0e535642      0x0e525c53      0x540e6157      0x6453605d
0x601330 <b3+16>:       0x0e1c6053      0x0e626330      0x5362544f      0x56620e60
0x601340 <b3+32>:       0x0e1a624f      0x15635d67      0x550e5360      0x0e525d5d
0x601350 <b3+48>:       0x550e5d62      0x00001c5d      0x00000000      0x00000000
```

b3? That sounds familiar. It's the local data as shown from nm. And since we see printf later on in the function c8, we can deduce that something around here may be the flag! So we look at more data before it and look for the hex values for flag: `66 6c 61 67`

>x/80x 0x601260

```
0x601260:       0x00000000      0x00000000      0x00000000      0x00000000
0x601270:       0x00000000      0x00000000      0x00000000      0x00000000
0x601280 \<b>:   0x27212c26      0x2932373b      0x291f2534      0x25221f2e
0x601290 <b+16>:        0x25292e32      0x00003de1      0x00000000      0x00000000
0x6012a0 <b0>:  0x6162583f      0x62576554      0x13583713      0x54665c43
0x6012b0 <b0+16>:       0x5b4a1332      0x13661a62      0x67545b67      0x478673d5
0x6012c0 <b0+32>:       0x6113585b      0x13676b58      0x66586563      0x6158575c
0x6012d0 <b0+48>:       0x00003267      0x00000000      0x00000000      0x00000000
0x6012e0 <b1>:  0x5814594b      0x1b62585d      0x59581468      0x6a665967
0x6012f0 <b1+16>:       0x59361459      0x595d6266      0xa2b0cb22      0x002d6867
0x601300 <b2+4>:        0x59141300      0x01072303      0x00005913      0x00000000
0x601310:       0x00000000      0x00000000      0x00000000      0x00000000
0x601320 <b3>:  0x0e535642      0x0e525c53      0x540e6157      0x6453605d
0x601330 <b3+16>:       0x0e1c6053      0x0e626330      0x5362544f      0x56620e60
0x601340 <b3+32>:       0x0e1a624f      0x15635d67      0x550e5360      0x0e525d5d
0x601350 <b3+48>:       0x550e5d62      0x00001c5d      0x00000000      0x00000000
0x601360 <mem>: 0x00000000      0x00000000      0x00000000      0x00000000
0x601370:       0x00322e37      0x3a434347      0x65442820      0x6e616962
0x601380:       0x342e3420      0x322d372e      0x2e342029      0x00372e34
0x601390:       0x79732e00      0x6261746d      0x74732e00      0x62617472
```

If we look at the first hex value that's not 0s, it is `0x27212c26`
Hmmmmm... `0x27212c26` vs `66 6c 61 67` Seems pretty similar right?
We convert `0x27212c26` from little endian to big endian by reversing the order in groups of 2 and add 40 and we get the hex for flag!

Now we just have to repeat the same thing for the following values of hex and we get our flag.

#Flag

>flag{write_in_bernie!}

Note: this method only worked because the encryption of the flag was very simple, but a flag is a flag.
