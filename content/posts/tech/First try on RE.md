---
title: "First try on RE"
draft: false
author: ["nanimonai"]
date: 2024-09-10
hidemeta: true
description: "没有悲悯他的神明，命运倒向金钱的天平。"
---
# 第N次RE碰壁

​	近来参与尝试 BaseCTF 赛事中的“NEURO爱数学”题，结合官方WP进行总结。

​	题目共给出三项提示：

 	1. flag为BaseCTF开头。z3可能存在多解，请尝试从数学角度解决。
 	2. 你知道的，11AdD8_result_21。
 	3. 多项式展开

​	下载下来是一个exe，选择拖入die。没什么问题：

![](https://img.nanimonai.org/DIE.png) 

​	再拖入ida端详。找到main后进入Pseduocode：

```
int __fastcall main(int argc, const char **argv, const char **envp)
{
  FILE *v3; // rax
  FILE *v4; // rax
  FILE *v5; // rax
  __int64 v6; // rdx
  char v9[26]; // [rsp+50h] [rbp-80h] BYREF
  _BYTE v10[26]; // [rsp+6Ah] [rbp-66h] BYREF
  int Src[7]; // [rsp+90h] [rbp-40h] BYREF
  unsigned __int8 v12; // [rsp+AFh] [rbp-21h]
  __int64 v13; // [rsp+B0h] [rbp-20h]
  _QWORD *v14; // [rsp+B8h] [rbp-18h]
  int m; // [rsp+C0h] [rbp-10h]
  int k; // [rsp+C4h] [rbp-Ch]
  int j; // [rsp+C8h] [rbp-8h]
  int i; // [rsp+CCh] [rbp-4h]

  _main();
  feclearexcept(63);
  if ( fetestexcept(3) )
  {
    puts("Floating point exceptions are set, possibly being debugged.");
    exit(1);
  }
  if ( IsDebuggerPresent() )
  {
    puts("This program is being debugged. Exiting!");
    exit(1);
  }
  printf("I'm practicing my neuro math skills. Give me nine integers: ");
  scanf(
    "%d %d %d %d %d %d %d %d %d",
    coeffs,
    &dword_408044,
    &dword_408048,
    &dword_40804C,
    &dword_408050,
    &dword_408054,
    &dword_408058,
    &dword_40805C,
    &dword_408060);
  printf("Hmm, let me think");
  v3 = __iob_func();
  fflush(v3 + 1);
  if ( IsDebuggerPresent() )
  {
    puts("This program is being debugged. Exiting!");
    exit(1);
  }
  putchar(46);
  v4 = __iob_func();
  fflush(v4 + 1);
  putchar(46);
  v5 = __iob_func();
  fflush(v5 + 1);
  puts(".");
  for ( i = -60; i <= 59; ++i )
  {
    total = 0;
    power = 1;
    for ( j = 0; j <= 8; ++j )
    {
      total += power * coeffs[j];
      power *= i;
      result = total;
    }
    if ( i == 44
      || i == 58
      || (::v12 = i + 37, (unsigned int)(i + 37) <= 0x36)
      && (v14 = &::v5, v13 = (unsigned int)::v12, v6 = ::v5, (v12 = _bittest64(&v6, (unsigned int)::v12)) != 0) )
    {
      if ( IsDebuggerPresent() )
      {
        puts("This program is being debugged. Exiting!");
        exit(1);
      }
      if ( result )
        goto LABEL_21;
    }
    else if ( !result )
    {
      if ( IsDebuggerPresent() )
      {
        puts("This program is being debugged. Exiting!");
        exit(1);
      }
LABEL_21:
      puts("Those aren't the right numbers. Try again!");
      return 1;
    }
  }
  if ( dword_408060 != 1 )
  {
    for ( k = 0; k <= 8; ++k )
      coeffs[k] /= dword_408060;
  }
  if ( IsDebuggerPresent() )
  {
    puts("This program is being debugged. Exiting!");
    exit(1);
  }
  if ( dword_40805C == -80 && dword_408058 == -358 )
  {
    printf("Correct! Here's the flag: ");
    Src[0] = coeffs[0];
    Src[1] = dword_408044;
    Src[2] = dword_408048;
    Src[3] = dword_40804C;
    LOWORD(Src[4]) = dword_408050;
    HIWORD(Src[4]) = dword_408054;
    LOWORD(Src[5]) = dword_408058;
    HIWORD(Src[5]) = dword_40805C;
    LOWORD(Src[6]) = dword_408060;
    memcpy(v9, Src, sizeof(v9));
    memcpy(v10, Src, sizeof(v10));
    for ( m = 0; m <= 51; ++m )
    {
      v9[m] ^= xorcode[m];
      putchar((unsigned __int8)v9[m]);
    }
    putchar(10);
    return 0;
  }
  else
  {
    printf("WRONG");
    return 1;
  }
}
```

​	本质上是一个代码审计加逻辑复写，需要输入九个数。程序前半段大概是这样的公式：
$$
x_1*i^8+x_2*i^7+x_3*i^6+x_4*i^5+x_5*i^4+x_6*i^3+x_7*i^2+x_8*i^1+x_9*i^0
$$
​	而后是去计算(i-44)(i-58)(i-17)(i-6)(i-5)(i+4)(i+9)(i+37)————这个巨他妈难看，官方给出的答案是考虑本地模拟 or 动调。

​	程序要求用户输入9个整数，存储在 `coeffs` 数组中。使用这9个整数作为系数，构造一个8次多项式。然后在整数范围 [-60, 59] 内对这个多项式进行求值。对于每个 i 从 -60 到 59：初始化 total = 0 和 power = 1，对每个系数 j 从 0 到 8，然后：

- total += power * coeffs[j]
- power *= i

​	最终的 total 就是多项式在 i 点的值。期间验证程序检查多项式在特定点的值是否为0。这些特定点上述以及概括而出。如果上述条件满足，程序会将所有系数除以最后一个系数（如果最后一个系数不等于1）。最终检查： 程序检查调整后的第8个系数是否等于-80，第7个系数是否等于-358。所有条件都满足，程序使用调整后的系数生成一个标志（flag）。生成过程涉及将系数转换为字节，然后与预定义的 xor code 进行异或解密。

​	以下是复现：

```
#include <stdio.h>
#include <stdint.h>

int main(){
    uint64_t v5 = 0x400C0210000001LL;  // 模拟位数组

    for(int x = -60; x <= 60; x++){
        if(x == 44 || x == 58){
            printf("%d\n", x);
            continue;
        }

        unsigned int v12 = (unsigned int)(x + 37);

        if(v12 <= 54 && (v5 & (1ULL << v12))){
            printf("%d\n", x);
        }
    }

    return 0;
}

```

​	将其展开与上面对比就可以知道x1-x9的数值：

```
from sympy import symbols, expand
x = symbols('x')
polynomial = (x - 44) * (x - 58) * (x - 5) * (x + 37) * (x - 17) * (x + 9) * (x - 6) * (x + 4)
expanded_polynomial = expand(polynomial)
standard_form = expanded_polynomial.as_poly()
coefficients = standard_form.all_coeffs()
print(coefficients[::-1])
```

​	输出如下：

![](https://img.nanimonai.org/yunxing.png) 

​	堂堂完结，代入即可得到flag。

​	（笔者注：使用cmd打开做，不然那个flag一闪就过去了。）

​	（事后看了师傅的直播，发现在coeffs这个地方ida里面，它九个整数的地址是连续的，所以可以直接右键改一下，看成一个数组就行）
