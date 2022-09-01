# Fast Fourier Transform学习笔记

[toc]

## 一、引入

倘若我们要计算两个n次多项式相乘

​		朴素算法需要**$O(n^2)$**的时间

我们能不能有更快的方法呢？

## 二、性质一

> 若$f(x)$是n次函数，
>
> 即$f(x)=a_1x^n + a_2x^{n-1}+ \dots + a_n$,
>
> 则其必然能被$n + 1$个点表示。

换个说法

> $n+1$个点能够确定唯一一条n次函数的图形

所以我们计算

$h(x)=[f(x)=a_1x^n + a_2x^{n-1}+ \dots + a_n]*[g(x)=b_1x^n + b_2x^{n-1}+ \dots + b_n]$的时候，

能知道$h(x)$必然是一个$2n$次的函数，所以我们能够提出$2n+1$个点，并将其带入到$f(x)$,$g(x)$中，

得到

$\{(x_1,f(x_1)),(x_2,f(x_2)),\dots,(x_{2n+1},f(x_{2n+1}))\}$  和   $\{(x_1,g(x_1)),(x_2,g(x_2)),\dots,(x_{2n+1},g(x_{2n+1}))\}$。等$2n+1$个点。

则$h(x)$可以用$\{(x_1,f(x_1)*g(x_1)),(x_2,f(x_2)*g(x_2)),\dots,(x_{2n+1},f(x_{2n+1})*g(x_{2n+1}))\}$  来表示。这样子计算两个用 **值表示** 的函数相乘，能够以$O(n)$的时间来计算了。

但是仍然存在着问题，对于值表示的每一个点，我们都需要花费$O(n)$的时间来计算，那么$2n+1$个点，不就意味着我们则需要用$O(n^2)$的时间来将整个函数从  **系数表示**  转化为值表示？！！所以我们引入重要的第二个性质。

## 三、性质二

我们先来观察最简单的两个函数

![image-20220721201129728](C:\Users\BY\AppData\Roaming\Typora\typora-user-images\image-20220721201129728.png)

很显然，这是一个一次函数，我们很容易的可以看出它的对称性，即$f(-x) = -f(x)$

![image-20220721201142190](C:\Users\BY\AppData\Roaming\Typora\typora-user-images\image-20220721201142190.png)

同样也很容易看出，这是一个二次函数，这个函数的对称性我们也很容易看出来，即$f(-x)=f(x)$

这两个经典的奇函数和偶函数的对称性我们一眼就能够直观的看出， 

那么这样子有什么用呢？

倘若我们知道了$f(x_1)$的值，是不是相应的就可以知道$f(-x_1)$的值了？所以如果我们选择的$2n+1$个点都是正负对，不就能够减少一半的计算时间了吗？	

那么你可能还会提问，并非是所有函数都是奇函数或者偶函数阿，那我们能不能将其拆分成一个类似奇函数或者偶函数的形式呢？

对于   $P(x)=3x^5+2x^4+x^3+7x^2+5x+1$  

这样一个较为复杂的多项式，我们根据上面的思路，尝试着将其写为这样子的形式

$P(x)=(2x^4+7x^2+1)+(3x^5+x^3+5x)$将其偶数项与奇数项分开

再进一步，我们将奇数项再提出一个$x$，得到

$P(x)=(2x^4+7x^2+1)+x(3x^4+x^2+5)$

是不是这样子两边都成为了偶数项，给这两个多项式再起一个名字，

$P_e(x^2) = 2x^4+7x^2+1$代表偶数项

$P_o (x^2)= 3x^4+x^2+5$代表提取出x后的奇数项

那么$P(x) = P_e(x^2)+xP_o(x^2)$

所以对于每个$x_i$则都有

$ \left. \begin{matrix} P(x_i) = P_e(x_i^2)+xP_o(x_i^2) \\ P(-x_i) = P_e(x_i^2)-xP_o(x_i^2)  \end{matrix} \right\} Lot~of~overlap $    这样子就能够省掉一半的计算

所以我们将上述结论推导到一般形式

对于  $P(x)=p_0+p_1x^n+p_2x^{n-1}+\dots+p_{n-1}x^2+p_nx$ 

$$ \left. \begin{matrix} P(x_i) = P_e(x_i^2)+xP_o(x_i^2) \\ P(-x_i) = P_e(x_i^2)-xP_o(x_i^2)  \end{matrix}\right. $$ 我们都能够将其写成这样子的形式

所以说，如果最初需要求$n$个点的值，那么现在只需要求$n/2$个点的值，这看起来很像是一个递归分治算法的开始，但是，我们还是卡住了，对于每个$x_i^2$我们都不能够再将其再次正负拆分了，这不能使我们再继续正负点拆分，递归到第一层就结束了。

## 四、性质三

倘若实数区域的$x_i^2$不能够满足正负拆分，那我们能不能大胆一点，把数系拓展到虚数呢？这样一来，一切都合理了。

所以该如何选取这些$x_i$呢？如何选定$x_i$才能够使之恰好都具有正负对呢？

我们举个例子,假设一个三次方程
$$
P(x)=x^3+x^2-x-1
$$
它的点集表达需要至少4个点，我们将其写为$\{x_1,-x_1,x_2,-x_2\}$

按照上面步骤，我们得到第二层的点，我们同样将其表示为$\{x_1^2,x_2^2\}$

要让递归继续进行下去的条件是$x_1^2$与$x_2^2$仍然是正负对，所以$x_2^2=-x_1^2$，从而得到第三层的数应该是$x_1^4$，接下来我们尝试着将一些点代入$x_1$中，为了方便我们代入1进入。

可以得到$x_2^2 = -1$所以能够得到，$x_2=i或-i$

所以我们选定的四个点是$\{1,-1,i,-i\}$

上述过程不就是一个求解$x^4=1$的复数域上的所有的解吗？

## 五、性质四

我们最终得到了$h(x)=[f(x)=a_1x^n + a_2x^{n-1}+ \dots + a_n]*[g(x)=b_1x^n + b_2x^{n-1}+ \dots + b_n]$的 **值表示** 形态，但是我们最终仍然是需要得到它的一般形式——系数表示，这个就涉及到傅里叶逆变换。

## 六、代码实现

```C++
#include <iostream>
#include <cstdio>
#include <cmath>
#include <cstring>
#include <complex>
#include <algorithm>
using namespace std;
inline int Read(){
	int dataIn = 0,f = 1;
	char ch = getchar();
	while(!isdigit(ch))	f = ch == '-' ? -1 : 1,ch = getchar();
	while(isdigit(ch))	dataIn = dataIn * 10 + ch -'0',ch = getchar();
	return f * dataIn;
}
const int maxsize = 6000010;
const double Pi = acos(-1);
int n,m;
struct Complex{
	double x;
	double y;
	Complex(double xx = 0,double yy = 0){x = xx,y = yy;}
	inline Complex operator + (const Complex a) const{
		return Complex(x + a.x,y + a.y);
	}
	inline Complex operator - (const Complex a) const{
		return Complex(x - a.x,y - a.y);
	}
	inline Complex operator * (const Complex a) const{
		Complex res;
		res.x = x * a.x - y * a.y;
		res.y = x * a.y + y * a.x;
		return res;
	}
};
Complex f_x[maxsize],g_x[maxsize];
int _limit = 1,cnt;
int r[maxsize];
void FastFourierTransform(Complex *A,int type){
	for(int i = 0;i < _limit;i++)
		if(i < r[i])
			swap(A[i],A[r[i]]);
	for(int mid = 1;mid < _limit;mid<<=1){
		Complex  Wn(cos(Pi / mid),type * sin(Pi / mid));
		for(int R = mid << 1,j = 0;j < _limit;j += R){
			Complex w(1,0);
			for(int k = 0;k < mid;k++,w = w * Wn){
				Complex x = A[j + k],y = w * A[j + mid + k];
				A[j + k] = x + y;
				A[j + mid + k] = x - y;
			}
		}
	}
}
int main(){
	n = Read();m = Read();
	for(int i = 0;i <= n;i++)	f_x[i].x = Read();
	for(int i = 0;i <= m;i++)	g_x[i].x = Read();
	while(_limit <= n + m) _limit <<= 1,cnt++;
	for(int i = 0;i < _limit;i++)
		r[i] = (r[i >> 1] >> 1) | ((i & 1) << (cnt - 1));
	FastFourierTransform(f_x,1);
	FastFourierTransform(g_x,1);
	for(int i = 0;i <= _limit;i++)
		f_x[i] = f_x[i] * g_x[i];
	FastFourierTransform(f_x,-1);
	for(int i = 0;i <= n + m;i++)
		printf("%d ",(int)(f_x[i].x / _limit + 0.5));
	return 0;
}
```

