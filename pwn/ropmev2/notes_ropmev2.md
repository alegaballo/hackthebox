
# ropmev2

When executed, the program prints:

```
Please dont hack me
```

then waits for a input. If the input is "DEBUG", the program calls its main again:

```
Please dont hack me
DEBUG
I dont know what this is 0x7fffd6972c00
Please dont hack me
```

## Decompiled source


```c++
int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  char **v3; // rdx
  char *buf[26]; // [rsp+0h] [rbp-D0h]

  sub_401213();
  printf("Please dont hack me\n", buf);
  read(0, buf, 0x1F4uLL);
  if ( !strcmp("DEBUG\n", (const char *)buf) )
  {
    printf("I dont know what this is %p\n", buf);
    main((__int64)"I dont know what this is %p\n", buf, v3);
  }
  sub_401238((const char *)buf);
  return 0LL;
}

int sub_401213()
{
  return setvbuf(stdout, 0LL, 1, 0LL);
}

void __fastcall sub_401238(const char *a1)
{
  int i; // [rsp+1Ch] [rbp-14h]

  if ( a1 )
  {
    for ( i = 0; i < strlen(a1); ++i )
    {
      if ( a1[i] <= '`' || a1[i] > 'm' ) // x < 'a' or  x > 'm'
      {
        if ( a1[i] <= '@' || a1[i] > 'M' ) // x < 'A' or x > 'M'
        {
          if ( a1[i] <= 'm' || a1[i] > 'z' )  // x < 'm' or x > 'z'
          {
            if ( a1[i] > 'M' && a1[i] <= 'Z' ) // x > 'M' or x < 'Z'
              a1[i] -= 13;
          }
          else
          {
            a1[i] -= 13;
          }
        }
        else
        {
          a1[i] += 13;
        }
      }
      else
      {
        a1[i] += 13;
      }
    }
  }
}
```


## Memory Corruption

By inserting the string genered with ``print("DEBUG\n"+'A'*208)``

```
DEBUG
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

as input, we corrupt the program:

```
Please dont hack me
DEBUG
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAI dont know what this is 0x7ffcf69a5d60
Please dont hack me

zsh: segmentation fault (core dumped)  ./ropmev2
```


## source analysis 


```c++
// a1 = argc
// a2 = argv
int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  // pointer - 8 bytes
  char **v3; // rdx
  // buf is an array of pointers to string - total byte length: 26*8 = 208
  char *buf[26]; // [rsp+0h] [rbp-D0h]

  // change stdin settings
  sub_401213();

  // buf is useless here, but probably here the decompiler was confused with the next call
  printf("Please dont hack me\n", buf);

  // read from stdin 0x1F4=500 bytes 
  read(0, buf, 0x1F4uLL);

  if ( !strcmp("DEBUG\n", (const char *)buf) )
  {
    printf("I dont know what this is %p\n", buf);
    // calls main again, passing buf as argc and v3 as argv
    main((__int64)"I dont know what this is %p\n", buf, v3);
  }
  sub_401238((const char *)buf);
  return 0LL;
}

// this procedure probably makes read() to not stop at newlines
int sub_401213()
{
  return setvbuf(stdout, 0LL, 1, 0LL);
}

// sub_401238 implements ROT13, it modifies the string inplace
void __fastcall sub_401238(const char *a1)
{
  int i; // [rsp+1Ch] [rbp-14h]

  if ( a1 )
  {
    for ( i = 0; i < strlen(a1); ++i )
    {
      if ( a1[i] <= '`' || a1[i] > 'm' ) // x < 'a' or  x > 'm'
      {
        if ( a1[i] <= '@' || a1[i] > 'M' ) // x < 'A' or x > 'M'
        {
          if ( a1[i] <= 'm' || a1[i] > 'z' )  // x < 'm' or x > 'z'
          {
            if ( a1[i] > 'M' && a1[i] <= 'Z' ) // x > 'M' or x < 'Z'
              a1[i] -= 13;
          }
          else
          {
            a1[i] -= 13;
          }
        }
        else
        {
          a1[i] += 13;
        }
      }
      else
      {
        a1[i] += 13;
      }
    }
  }
}
```