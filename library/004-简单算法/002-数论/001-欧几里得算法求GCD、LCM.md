# 001-欧几里得算法（辗转相除法）求GCD、LCM

## 代码实现
```c++
#include <iostream>
#include <string>
#include <cstring>
#include <algorithm>
#include <math.h>
#include <cstdio>

using namespace std;

long long a,b;

//a,b gcd转化为求 b,a%b 的 gcd问题，直至b为0  
long long gcd(long long x,long long y){
	if(y == 0) return x;
	return gcd(y, x%y);
}

int main(){
	cin>>a>>b;
	//GCD
	cout<<gcd(a,b)<<endl;
	//LCM
	cout<<a*b/gcd(a,b)<<endl;
	return 0;
}
```
## 思路
这其实是一个定理，两个整数最大公约数等于其中较小数和两数的差的最大公约数，也就是GCD(a, b) = GCD(b, a%b)，定理的正确性证明可以参考下面维基百科。

## 维基百科
[欧几里得算法正确性证明](https://zh.wikipedia.org/wiki/%E8%BC%BE%E8%BD%89%E7%9B%B8%E9%99%A4%E6%B3%95#%E6%AD%A3%E7%A1%AE%E6%80%A7%E7%9A%84%E8%AF%81%E6%98%8E)
