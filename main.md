## 组合数、阶乘、逆元

```C

const int mod=998244353; 
long long ksm(long long a,long long b){
	long long res=1;
	while(b){
		if(b&1){
			res=res*a%mod;
		}
		a=a*a%mod;
		b>>=1;
	}
	return res;
}
long long f[1011111],inv[1011111];
long long dp[1011111];
long long C(int m,int n){
	return (f[n]*inv[m]%mod)*inv[n-m]%mod;
}

f[0]=1;
for(int i=1;i<=1001111;i++){
	f[i]=f[i-1]*i%mod;
}
inv[1001111] = ksm(f[1001111],mod-2);
for(int i=1001110;i>=0;i--){
	inv[i]=inv[i+1]*(i+1)%mod;
}

```


## 基于随机的大质数判断

### Miller_Rabin

利用费马小定理和随机数进行伪验证，时间复杂度为 $O(n^{1/4})$ 。

```C

#define rep(i , j , n) for(int i = j ; i <= n ; i++)
#define red(i , n , j)  for(int i = n ; i >= j ; i--)

int quickmul(int a , int b , int m){
    return  ((a * b - (ll)(long double)(a/m*b) * m)+m)%m ;
}
 
int quickpow(int a , int b , int m){
    int ans = 1 ;
    while(b){
        if(b&1) ans = quickmul(ans , a , m) ;
        b >>= 1 ;
        a = quickmul(a , a , m);
    }
    return ans ;
}

bool Miller_Rabin(int n){
    if(n == 46856248255981ll || n < 2) return false;
    if(n == 2 || n == 3 || n == 7 || n == 61 || n == 24251) return true;
    if(!(n&1) || !(n%3) || !(n%61) || !(n%24251)) return false;
    int m = n - 1 , k = 0 ;
    while(!(m&1)) k++ , m>>=1 ;// 分解2^s * t = n - 1
    rep(i , 1 , 20){
        int a = rand() % (n - 1) + 1 , x =  quickpow(a , m , n) , y;
        rep(j , 1 , k){
            y = quickmul(x , x , n);
            if(y == 1 && x != 1 && x != n - 1) return false;//二次测探
            x = y ;
        }
        if(y != 1) return false;//费马小定理
    }
    return true;
}

```

## 博弈论
### 巴什博弈
**定义**：一堆n个物品，两个人轮流从中取出不多于m个，最后取光者胜，不能继续取的人输；

**结论**：若 n % ( m + 1 ) ! = 0 ，则先手必胜；反之，先手必输。

**证明**：若 n % ( m + 1 ) ! = 0 ，记 n = k * ( m + 1 ) + r，先者取走r个，那么设后者t个，只要先者拿( m + 1 − t )个，即始终和后者拿的数目之和为( m + 1 ) $先手必胜。反之，先手必输。

### 斐波那契博弈
**定义**：有一堆物品，共n个，两人轮流取物，先手可取任意件，但不能不取，也不能把物品取完，之后每次取的物品数不能超过上一次的两倍，且至少为1件，取走最后一件物品的人获胜。

**结论**：当且仅当n不是斐波那契数时，先手胜。

**证明**：此博弈的证明需要各种不等式关系证明，一般记住结论即可

### 威佐夫博弈
**定义**：有两堆物品，数量分别为a个和b个，两人轮流取物，每次可以从一堆中取出任意个，也可以从两堆中取出相同数量的物品，每次至少要取一个，最后取完所有物品的人获胜。

**结论**：若 $abs(a-b)$ * $\dfrac{\sqrt{5}+1}{2}$ == $min(a,b)$ 成立，则后手获胜，否则先手胜。其中 $\dfrac{\sqrt{5}+1}{2}$ 为黄金分割比。

**证明**：略

### 尼姆博弈
**定义**：n堆物品，每堆物品的个数任意，两人轮流取，每次取某堆中不少于1个，最后取完者胜。

**结论**：将每堆物品的数量全部异或起来，若值为0，则先手必败，否则先手必胜。

**证明**：我们将每堆物品数异或起来为0这个状态称为必败态，顾名思义，这个状态下，谁取谁必败。因为当这个状态时，经过两人轮流取物，后者始终可以维持这个必败态，即A取完后，B一定可以取一个数，使得取完后每堆物品数异或起来仍为0。这样一直到最后一轮，B取完一定会使每堆数都为0，此时同样也是必败态（异或起来为0），这时B获胜，A面对所有堆都为0这个状态取，直接失败。

所以当每堆物品数全部异或起来，若值为0，此时已是必败态，先手必败；若值不为0，则先手一定会取一个数使得每堆数异或起来为0，达到必败态，从而后手必败。

**注**： 博弈时，每个人都会走当前最优策略，所以每个人都会尽量给对方创造必败态，给自己创造必胜态。

**代码实现**

```C
int r=0;
for(int i=1;i<=n;++i){
	r=r^a[i];
} 
if(r) return true;
else return false;

```

### SG函数

实际题目中，并不可能给出如此标准的博弈模型。对于更加一般的博弈论问题，我们需要通过SG函数转化为尼姆博弈的方式进行求解。

首先给出一种 ICG（公平组合游戏）博弈游戏模型：给定一个有向无环图和一个起始顶点上的一枚棋子，两名选手交替的将这枚棋子沿有向边进行移动，无法移动者判负。

将ICG问题进行转换：任意一个ICG都可以通过把每个局面看成一个顶点，对每个局面和它的子局面连成一条有向边来抽象成这个有向图游戏。

首先定义一个mex函数，mex函数就是求集合中未出现的最小的非负整数。例如 $mex(0,1,2,4)=3$ ， $mex(2,3,5)=0$ ， $mex()=0$ ；

对于一个给定的有向无环图，定义关于图的每个顶点的SG函数SG如下( $y_i$ 是 $x$ 的后继) 。

$$SG(x)=mex(SG(y_1),SG(y_2),...,SG(y_k))$$

特别地，整个有向图G的SG值被定义为有向图起点s的SG值即 $SG(G)=SG(s)$ 。

根据以下步骤建树：

一、找出必败态（SG值为0）

二、找出当前所有状态的前驱结点

三、根据定义计算结点的SG值

四、重复以上步骤，直到建树完成


## 线性基
这是一种用于处理异或问题的数据结构。线性基能使用异或运算来表示原数集使用异或运算能表示的所有数。运用这个性质，我们可以极大地缩小异或操作所需的查询次数。
**注意，原数组中不能出现0。**

```C
#include<bits/stdc++.h>
#define reg register
using namespace std;
typedef long long ll;
const int MN=60;
ll a[61],tmp[61];
bool flag;
void ins(ll x){
    for(reg int i=MN;~i;i--)
        if(x&(1ll<<i))
            if(!a[i]){a[i]=x;return;}
            else x^=a[i];
    flag=true;
}
bool check(ll x){
    for(reg int i=MN;~i;i--)
        if(x&(1ll<<i))
            if(!a[i])return false;
            else x^=a[i];
    return true;
}
ll qmax(ll res=0){
    for(reg int i=MN;~i;i--)
        res=max(res,res^a[i]);
    return res;
}
ll qmin(){
    if(flag)return 0;
    for(reg int i=0;i<=MN;i++)
        if(a[i])return a[i];
}
ll query(ll k){
    reg ll res=0;reg int cnt=0;
    k-=flag;if(!k)return 0;
    for(reg int i=0;i<=MN;i++){
        for(int j=i-1;~j;j--)
            if(a[i]&(1ll<<j))a[i]^=a[j];
        if(a[i])tmp[cnt++]=a[i];
    }
    if(k>=(1ll<<cnt))return -1;
    for(reg int i=0;i<cnt;i++)
        if(k&(1ll<<i))res^=tmp[i];
    return res;
}
int main(){
    int n;ll x;scanf("%d",&n);
    for(int i=1;i<=n;i++)scanf("%lld",&x),ins(x);
    printf("%lld\n",qmax());
    return 0;
}
```

## FFT
这里是用的是基于分治的写法，常数更优秀。FFT利用单位根的性质进行分治，可以在 O(nlogn) 的时间内完成DFT和IDFT。

```C
#include<bits/stdc++.h>
#define N 2621450
#define pi acos(-1)
using namespace std;
typedef complex<double> E;
int n,m,l,r[N];
E a[N],b[N];
void FFT(E *a,int f){
    for(int i=0;i<n;i++)if(i<r[i])swap(a[i],a[r[i]]);
    for(int i=1;i<n;i<<=1){
        E wn(cos(pi/i),f*sin(pi/i));
        for(int p=i<<1,j=0;j<n;j+=p){
            E w(1,0);
            for(int k=0;k<i;k++,w*=wn){
                E x=a[j+k],y=w*a[j+k+i];
                a[j+k]=x+y;a[j+k+i]=x-y;
            }
        }
    }
}
inline int read(){
    int f=1,x=0;char ch;
    do{ch=getchar();if(ch=='-')f=-1;}while(ch<'0'||ch>'9');
    do{x=x*10+ch-'0';ch=getchar();}while(ch>='0'&&ch<='9');
    return f*x;
}
int main(){
    n=read();m=read();
    for(int i=0;i<=n;i++)a[i]=read();
    for(int i=0;i<=m;i++)b[i]=read();
    m+=n;for(n=1;n<=m;n<<=1)l++;
    for(int i=0;i<n;i++)r[i]=(r[i>>1]>>1)|((i&1)<<(l-1));
    FFT(a,1);FFT(b,1);
    for(int i=0;i<=n;i++)a[i]=a[i]*b[i];
    FFT(a,-1);
    for(int i=0;i<=m;i++)printf("%d ",(int)(a[i].real()/n+0.5));
}
```
