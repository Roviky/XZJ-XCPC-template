## 博弈论
### 巴什博弈
定义：一堆n个物品，两个人轮流从中取出不多于m个，最后取光者胜，不能继续取的人输；

结论：若n % ( m + 1 ) ! = 0 ，则先手必胜；反之，先手必输。

证明：若n % ( m + 1 ) ! = 0 ，记 n = k * ( m + 1 ) + r，先者取走r个，那么设后者t个，只要先者拿( m + 1 − t )个，即始终和后者拿的数目之和为( m + 1 ) ,先手必胜。反之，先手必输。


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
