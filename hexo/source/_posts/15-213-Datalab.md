---
title: 15-213 Datalab
categories: [csapp]
tags: [csapp]
---

# Datalab 笔记

## 旧版本

### bitAnd(x,y) 

```c
/* 
 * bitAnd - x&y using only ~ and | 
 *   Example: bitAnd(6, 5) = 4
 *   Legal ops: ~ |
 *   Max ops: 8
 *   Rating: 1
 */
int bitAnd(int x, int y) {
  return ~((~x)|(~y));
}
```

可以参考[德摩根定律](https://zh.wikipedia.org/wiki/%E5%BE%B7%E6%91%A9%E6%A0%B9%E5%AE%9A%E5%BE%8B)

### getByte

```C
/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
int getByte(int x, int n) {
  int mask=0xff;
  return (x>>(n<<3))&mask;
}
```

第二题是从 x 中提取出第 i 个字节（i=0,1，2,3），方法就是将那个字节移位至最低位，然后用屏蔽码 `0xff` 提取就可以了

3.logicalShift

```C
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3 
 */
int logicalShift(int x, int n) {
  int mask=((0x1<<(32+~n))+~0)|(0x1<<(32+~n));
  return (x>>n)&mask;
/*  int c=((0x1<<31>>31)^0x1)<<31;
  return ((x>>n)^(c>>n)); it's wrong.
*/
/*return ~((~x)>>n); it's wrong.
*/ 
}
```

第三题要求实现逻辑右移，对于有符号的 `int` ，C 语言默认的移位方式是算术右移，就是右移时在高位扩展符号位，这里我们需要扩展的符号位都设置为 0 ,可以构造一个屏蔽码屏蔽 `x>>n` 中的非扩展的位，用 & 实现目的。   
但这里要注意 C 语言对移位位数超出自身长度的行为是未定义的，因此在这里构造屏蔽码时不能使得移位位数超过了32或是小于0，我这段代码为了避免这种情况的发生，将屏蔽码分了最高位和其他位两部分构造，直接使用 `((0x1<<(33+~n))+~0)` 构造的屏蔽码在 n=0 将会无法确定。    
这里 `32+~n` 表示了 31-n ，可以由补码的运算性质 `-x=~x+1` 得到，同时这里我在注释里写了两个我最初写的 bug 。      

4.bitCount

```C
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) {
  int _mask1=(0x55)|((0x55)<<8);
  int _mask2=(0x33)|((0x33)<<8);
  int _mask3=(0x0f)|((0x0f)<<8);
  int mask1=(_mask1)|(_mask1<<16);
  int mask2=(_mask2)|(_mask2<<16);
  int mask3=(_mask3)|(_mask3<<16);
  int mask4=(0xff)|(0xff<<16);
  int mask5=(0xff)|(0xff<<8);
  int ans=(x&mask1)+((x>>1)&mask1);
  ans=(ans&mask2)+((ans>>2)&mask2);
  ans=(ans&mask3)+((ans>>4)&mask3);
  ans=(ans&mask4)+((ans>>8)&mask4);
  ans=(ans&mask5)+((ans>>16)&mask5);
  return ans;
}
```

这题好难T_T，应该是这个 lab 里最难的了吧；     
题目意思就是要统计一个32位的 `int` 里的 `1` 的个数，但是只能使用40个操作符，直接扫一遍字的话操作符就大大超过规定数了；    
这里构造了五个常数，分别是 `0x55555555，0x33333333，0x0f0f0f0f，0x00ff00ff，0x0000ffff`，就是分别间隔了1个0,2个0,4个0,8个0和16个0,利用这五个常数就能依次计算出五个值，第一个值每两位的值为 x 的对应的两个位的和（即这两位中 `1` 的数目），第二个值每四位是第一个值对应的四位中两个两位的和（即原 x 中 `1`的数目），依次类推最后一个值就是结果了；    
怎么理解呢，可以看到这里构造的五个常数的间隔可以刚好使得只提取 n 位，移位之后再提取相邻 n 位（n=1,2,4,8,16），并且（考虑最大值可知）这两个 n 位加和后不会超出 n 位，使得 x 中的 `1` 一步步加和成最终的结果，可以举一个例子，若要求 1001 中 `1` 的数目，用`(1001&0101)+((1001>>1)&0101)`,就能将每相邻一位加和成一个两位，成 0101，再用`(0101&0011)+((0101>>2)&0011)`，就将每两位加和了，得到 0010 ,就是最终的结果。      

5.bang

```C
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int bang(int x) {
  x=(x>>16)|x;
  x=(x>>8)|x;
  x=(x>>4)|x;
  x=(x>>2)|x;
  x=(x>>1)|x;
  return ~x&0x1;
}
```

这题要求仅用规定的操作符来实现！运算，对 0 运算就得到 1,对非 0 就得到 0；也就是如果 x 的位中含有 1 就返回 0 ,这里运用移位后取或将 x 中的位一步步「折叠」 到了第一位上，然后判断第一位就可以了，这种「折叠」的方法很有趣，值得一看：）  

## 2.补码

6.tmin

```C
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 0x1<<31;/*tmin==~tmax*/
}
```

这题返回补码最小值，注意到 `tmin==~tmax`，补码负数表示部分和正数是不对称的，最小值的绝对值是最大值的绝对值加1。    

7.fitsBits

```C
/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int fitsBits(int x, int n) {
  int c=33+~n;
  int t=(x<<c)>>c;
  return !(x^t);
}
```

这题题目意思是判断 x 能否用一个 n 位的补码表示，能的话就返回 1,开始我没看懂题目…举个例子，101 是不能用 3 位补码表示的，因为 3 位最高位是符号位，最大只能表示 011,注意到这里 x 是32位的，不能直接右移的；   
要用 n 位的补码表示，x 只能是两种情况： `00…0|0|(n-1)位` 或是 `11…1|1|(n-1)位` ，这样 32 位的补码才会与 n 位的补码值相同，这里的方法就是将 x 左移（32-n）再右移回去，这样就能得到那两种情况的值，再判断这样操作之后是否与原来的 x 相等，就解决问题了；    
这里由补码性质，`33+~n` 等于 `32-n` 。      

8.divpwr2

```C
/* 
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int divpwr2(int x, int n) {
  int bias=(x>>31)&((0x1<<n)+~0);
  return (x+bias)>>n;
}
```

这题计算 x/(2^n) ，注意不能直接右移，直接右移是向下舍入的，题目要求是向零舍入，也就是正数向下舍入，负数向上舍入，这里参照 CS:APP 书上的做法，给负数加上一个偏正的因子 `(0x1<<n)+~0)` ，判断负数直接看符号位。    

9.negate

```C
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x+1;
}
```

这题求 -x ，直接利用补码的性质 `-x=~x+1` 就可以了。   

10.isPositive

```C
/* 
 * isPositive - return 1 if x > 0, return 0 otherwise 
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 3
 */
int isPositive(int x) {
  return !(!(x))&!((x>>31)&(0x1));
}
```

这里判断是否是正数，直接判断符号位，但是注意要排除 0 的情况！   

11.isLessOrEqual

```C
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int val=!!((x+~y)>>31);
  x=x>>31;
  y=y>>31;
  return (!!x|!y)&((!!x&!y)|(val));
}
```

这题比较两个数的大小，要求判断第一个数是否小于等于第二个数，这里考虑做减法然后判断符号，注意要考虑溢出的情况，这里 `((x+~y))` 表示了 `x-y-1` ，若其结果为负，则 x <= y ;    
这里先判断 x 与 y 的符号，如果 x 为负，y 为正直接返回 1 ,如果 x 为正，y 为正，直接返回 0；然后就是全正数和全负数的减法，这样不会溢出。   

12.ilog2

```C
/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 */
int ilog2(int x) {
  int ans=0;
  ans=(!!(x>>(16)))<<4;
  ans=ans+((!!(x>>(8+ans)))<<3);
  ans=ans+((!!(x>>(4+ans)))<<2);
  ans=ans+((!!(x>>(2+ans)))<<1);
  ans=ans+((!!(x>>(1+ans)))<<0);
  return ans;
}
```

这题求 x 以 2 为底的对数，解法有点难想到，注意到 32 位数的对数最大也不会超过 32,可以写成是 `16*a+8*b+4*c+2*d+e` 这里 a，b，c，d，e 都是 0 或 1，然后通过向右移 16 位就可以判断符号就可以得到 a ，右移 16*a+8 位可得到 b，以此类推得到其他位。   

## 3.浮点数

以下三题是关于浮点数的，可以使用任何操作符和分支语句。   

13.float_neg

```C
/* 
 * float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */
unsigned float_neg(unsigned uf) {
  int c=0x00ffffff;
  if(((uf<<1)^(0xffffffff))<c){
    return uf;
  }else{
    return uf^(0x80000000);
  }
}
```

这题计算 -f ，f 是浮点数，这里直接改浮点数的符号位，但是注意要单独考虑 NaN 的结果。      

14.float_i2f

```C
/* 
 * float_i2f - Return bit-level equivalent of expression (float) x
 *   Result is returned as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point values.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_i2f(int x) {
  int n=0xffffffff;
  int e=0; /* exp */
  int tmp=0;
  int tmp2=0;
  int cp=0;
  int cp2=0;
  int sign=x&0x80000000; /* 0x80000000 or 0x0 */

  if(x==0x80000000){
      return 0xcf000000;
    }
  if(x==0){
    return 0;
  }
  if(sign){
      x=-x;
  }

  x=x&0x7fffffff; /* remove sign */
  tmp=x;
  while(tmp){
    tmp=tmp>>1;
    n++;
  }

  x=x-(0x1<<n); /* remove highest bit */
  if(n<24){
    x=x<<(23-n);
  }else{
    tmp2=x>>(n-23);
    cp2=0x1<<(n-24);
    cp=x&((cp2<<1)-1);
    if(cp<cp2){
      x=tmp2;
    }else{
      if(tmp2==0x7fffff){
        x=0;
        n++;
      }else{
        if(cp==cp2){
          x=((tmp2)&0x1)+tmp2;
        }else{
          x=tmp2+1;
         }
       }
     }
   }
  e=(127+n)<<23;
  return sign|e|x;
}
```

这题是将整型转化为浮点数的格式，坑点很多，耗时长。。   
整体思路就是依次计算符号位，阶码值和小数字段，符号位可以直接移位提取，阶码值就是除了符号位外最高位的位数减 1 再加上偏差 127，小数字段可以移位（负数可以化为正数操作）获得，但这问题没这么简单，有很多坑点：   
1.特殊值 0 化为浮点数后是非规格化的，单独考虑；   
2.特殊值 0x80000000 是 2 的整数倍，小数部分用移位的话因为舍入问题会溢出，单独考虑；   
3.要仔细考虑移位过程，左移还是右移能得到 23 位的小数部分；   
4.注意舍入问题，这里需要仔仔细细地考虑清楚，默认使用向偶数舍入，就是舍去部分小于中间值舍弃，大于中间值进位，为中间值如 100 就向偶数舍入：就是看前一位，进位或舍弃总使得前一位为 0；   
5.最后就是操作数目限制在 30 位，我最开始写完的代码有 42 个操作符，应该是算法太麻烦了。。写完最后要一步步简化操作符数目，控制中 30 以内，这里我为了减少操作符数目，写了些可读性很不高的表达式，还用了不少变量如 cp，cp2，简化这些耗了我很多时间。

15.float_twice

```C
/* 
 * float_twice - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_twice(unsigned uf) {
  int tmp=uf;
  int sign=((uf>>31)<<31); /* 0x80000000 or 0x0 */
  int exp=uf&0x7f800000;
  int f=uf&0x7fffff;
  tmp=tmp&0x7fffffff; /* remove sign */
  if((tmp>>23)==0x0){
    tmp=tmp<<1|sign;
    return tmp;
  } else if((tmp>>23)==0xff){
    return uf;
  }  else{
    if((exp>>23)+1==0xff){
      return sign|0x7f800000;
    }else{
      return sign|(((exp>>23)+1)<<23)|f;
    }
  }
  return tmp;
}
```

这题计算浮点数的两倍，无穷大和 NaN 时直接返回，然后分规格化和非规格化两种讨论：    
规格化的情况，阶码值直接加 1 ，但是有个特殊情况就是加一后阶码为 255 时，应返回无穷大；   
非规格化的情况，排除符号位左移一位就可以了，因为这时阶码值为 0 ,两倍就相当于小数字段左移一位，不用担心溢出的情况，溢出时阶码值加 1,小数字段左移一位，相当于整体左移了。

## 新版本

### bitXor(x,y) 只使用 ~ 和 & 实现 ^

```c
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
    // 直接推公式，^可以使用~、&和|表示，而|又可以用~和&表示
    return ~(~x&~y)&~(x&y);
}
```

### tmin() 返回最小补码

![2-14](/Users/mac/Desktop/2-14.png)

有符号数是用补码来表示的，Tmin表示最小补码数，对于1个字节大小的补码，最小补码数形式为1000 0000（最小的有符号数，符号位为1，其余都是0），C语言中int类型占4字节，即32位，所以对1左移31位来构造最小补码。

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 1<<31;
}
```

### isTmax(x) 判断是否是补码最大值

函数功能是判断x是否是有符号数的最大值，也就是补码最大值，还是拿1个字节来看，最大补码数的形式为0111 1111，代码中的neg1是为了将-1单独判断出来，因为如果只使用return后面那句(!(~(x+1)^x))的话，会导致当x=-1的时候也会返回1，判断出现错误，而改变后的返回结果可以排除-1的干扰。

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
  int neg1;
  neg1 = !(~x); // 如果x为-1, 则neg1为1，否则neg1为0，这里是为了排除-1的干扰
  return !((~(x+1)^x)|neg1); // 给x加1，再翻转，最后和自身取异或，如果x为Tmax，则返回1，否则返回0
}
```

### allOddBits(x) 判断补码所有奇数位是否都是1

构造掩码操作即可，将掩码和x进行与操作，可以让x的奇数位置不变，而偶数位置变为0。

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2  
 */
int allOddBits(int x) {
  int mask = (0xAA << 8) + 0xAA;
  mask = (mask << 16) + mask; // 构造掩码
  return !((x & mask) ^ mask); // &操作将x的奇数位取出，偶数位置0，之后再与掩码异或判断是否满足条件
}
```

### negate(x) 不使用负号 - 实现 -x

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x + 1; // 补码取相反数操作：按位取反再加一
}
```



### isAsciiDigit(x) 判断 x 是否是 ASCII 码

通过上下界来判断输入的x是否在0x30~0x39的范围中，使用x分别加上界和下界，当x不在这个范围中时，通过判断符号位的变化来得出判断。

```c
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
  int negative = 1 << 31;
  int lessthan = ~(negative | 0x39); // 构造上界，如果超过，则符号位变为1
  int greatthan = (~(0x30) + 1); // 构造下界，如果不足，则符号位变为1

  lessthan = negative & (lessthan + x) >> 31;
  greatthan = negative & (greatthan + x) >> 31;

  return !(lessthan | greatthan); // 判断符号位是否为1
  return 2;
}
```

### conditional(x, y, z) 类似于 C 语言中的 x?y:z

重点在于return语句，这个操作可以根据x的不同来返回不同的值

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  x = !!x; // 判断x是否为0，若x=0，则x赋值为0；若x不为0，则x赋值为1
  x = ~x + 1; // 得到x的补码，0的补码还是0，1的补码为-1(二进制序列全1)

  return (x&y)|(~x&z); // 若x为0，则返回z；若x为1，则返回y
}
```



### isLessOrEqual(x,y) x<=y

判断方法：如果x和y同符号，当x<=y则返回1；或者如果x和y不同符号，那么当x<0则返回1；其余情况返回0。
这里根据y-x的结果的符号来判断x和y的大小。

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int minusx = ~x + 1; // 得到-x
  int result = y + minusx; // 得到y - x
  int sign = (result >> 31) & 1; // 判断result的符号，如果y>=x，则sign等于0，否则等于1
  int xsign = (x >> 31) & 1; // 取出x的符号
  int ysign = (y >> 31) & 1; // 取出y的符号
  int bitXor = xsign ^ ysign; // 判断x和y符号是否一致
  return ((!bitXor)&(!sign)) | ((bitXor&xsign)); // 要么x和y符号相同并且x<=y，要么x和y符号不同并且x<0
}
```



### logicalNeg(x) 计算 !x (不用 ! 运算符)

```
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
  return ((x | (~x + 1)) >> 31) + 1;
}
```



### howManyBits(x) 计算表达 x 所需的最少位数

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
  // 原理：对于正数，从高位到低位，找第一个位是1的（比如是n），再加上符号位，则最少需要n+1个位；
  // 对于负数，从高位到低位，找第一个位是0的（比如是n），则最少需要n位
  int b16, b8, b4, b2, b1, b0; // 表示0~15、16~23、24~27、28~29、30、31的位置处是否含有1，如有，则对其赋值需要的位数
  int sign = x >> 31; // 取符号位
  x = (sign&~x)|(~sign&x); // 如果x为正则不变，x为负则取反，这里是为了统一正负数，我们之后只用找到含有1的位置即可

  b16 = !!(x >> 16) << 4;// 先看高16位是否含有1，若有则表示至少需要16位，所以给b16赋值为16(1 << 4 = 16)
  x =  x >> b16; // 若有1，则原数右移16位，因为上面已经确定是否至少需要16位(针对0~15)；若没有1，则b16为0，x不用移位，继续往下面判断

  b8 = !!(x >> 8) << 3; // 看剩余位的高8位是否含有1，若有则表示至少还需要8位，给b8赋值为8
  x = x >> b8; // 同理...

  b4 = !!(x >> 4) << 2;
  x = x >> b4;

  b2 = !!(x >> 2) << 1;
  x = x >> b2;

  b1 = !!(x >> 1);
  x = x >> b1;
  b0 = x;
  return b16+b8+b4+b2+b1+b0+1; // 最后加上符号位
}
```



### floatScale2(uf) 计算 2.0*uf

需要了解计算机内浮点数的表示方法，了解浮点数中的规格数、非规格数、无穷大和未定义的区别和表示。

我们先看如何表示浮点数：

![float](float.png)

这里的uf类型为unsigned int，并不是浮点数，但是我们将uf看作为单精度类型，它有32位，最高位是符号位，之后8位保存指数信息，最后23位保存小数信息，所以在代码中我们可以看到，我们通过和0x7F800000取与操作来获得指数信息，再右移23位取出这一部分。

![2-32](2-32.png)

浮点数有几种特殊情况：

1.若exp部分全为0(exp = 0)，则是非规格化数，它是一种非常接近0的数；

2.若exp部分全为1(exp = 255)，当小数部分全为0时，表示无穷大；当小数部分不为全0时，表示未初始化数据NaN；

3.以上两种情况以外，就是规格化数。

![2-33](2-33.png)

所以我们需要判断uf是哪一种浮点数，并根据它的类型来进行相应的操作。

```c
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
  int exp = (uf&0x7F800000) >> 23; // 取出阶码
  int sign = uf&(1 << 31); // 取符号位
  if (exp == 0) return uf<<1|sign; // 若为非规格数，直接给uf乘以2后加上符号位即可
  if (exp == 255) return uf; // 若为无穷大或者NaN，直接返回自身
  exp = exp + 1; // 若uf乘以2（也就是阶码加1）后变成255，则返回无穷大
  if (exp == 255) return (0x7F800000|sign);
  return (exp << 23)|(uf&0x807FFFFF); // 返回阶码加1后的原符号数
}
```



### floatFloat2Int(uf) 计算 (int) f

需要了解整数和浮点数之间的转化方法，我们要做的就是将浮点数中的指数部分和小数部分取出来，然后通过这两部分来转化为整数，具体操作可以看代码，在这个过程中还要判断是否会产生溢出，以及浮点数是否为规格数等情况，如果产生溢出，我们需要返回一个特定的溢出值。

这里有一个将整数转化为浮点数的例子：

![prac](prac.png)

### floatPower2(x) 计算 2.0的x次方

```c
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x) {
  int INF = 0xFF << 23; // 设定一个最大值，也就是阶码位置都为1
  int exp = x + 127; // 计算阶码
  if (exp <= 0) return 0; // 阶码小于等于0，则返回0
  if (exp >= 255) return INF; // 阶码大于等于255，则返回INF
  return exp << 23;
}

```

