---
slug: algorithm
title:  Algorithm 
authors: [bkbk]
tags: [Algorithm]
---
 

:::danger 目录
**quicksort**  
**quicksort**  
**质数筛**  
**快速幂**  
**KMP**  
**topsort**  
**spfa**  
**prim**  
**gcd**  
**floyd**  
**dijkstra**  
**组合数**  
**中国剩余定理**  
**前缀和**  
**二分图最大匹配**  
**二分**  
**单调队列**  
**差分**  
**集成springboot**  
**序列化**   
**SpringCache**    
::: -->
<!--truncate-->
 


##  quicksort
<!-- [mybatis文档](https://mybatis.net.cn/index.html)  
[Mybatis-plus](https://www.baomidou.com/)  -->
<!-- #### 1.环境搭建     -->

 
``` cpp
#include <iostream>

using namespace std;

const int N = 1000010;

int q[N];

void quick_sort(int q[], int l, int r)
{
    if (l >= r) return;

    int i = l - 1, j = r + 1, x = q[l + r >> 1];
    while (i < j)
    {
        do i ++ ; while (q[i] < x);
        do j -- ; while (q[j] > x);
        if (i < j) swap(q[i], q[j]);
    }

    quick_sort(q, l, j);
    quick_sort(q, j + 1, r);
}

int main()
{
    int n;
    scanf("%d", &n);

    for (int i = 0; i < n; i ++ ) scanf("%d", &q[i]);

    quick_sort(q, 0, n - 1);

    for (int i = 0; i < n; i ++ ) printf("%d ", q[i]);

    return 0;
} 

#include <bits/stdc++.h>
using namespace std;
const int N=1000000+100;
int a[N],n,m,i,j;
int main()//C++ Stl使人快乐
{
    ios::sync_with_stdio(false);
    cin>>n;
    for(int i=1;i<=n;i++)
        cin>>a[i];
    sort(a+1,a+1+n);
    for(int i=1;i<=n;i++)
        cout<<a[i]<<" ";
    cout<<endl;
}
```
 

 



##  质数筛
``` cpp
1.最普通的筛法 O(nlogn) 
void get_primes2(){
    for(int i=2;i<=n;i++){

        if(!st[i]) primes[cnt++]=i;//把素数存起来
        for(int j=i;j<=n;j+=i){//不管是合数还是质数，都用来筛掉后面它的倍数
            st[j]=true;
        }
    }
}

2.诶氏筛法 O(nloglogn) 
void get_primes1(){
    for(int i=2;i<=n;i++){
        if(!st[i]){
            primes[cnt++]=i;
            for(int j=i;j<=n;j+=i) st[j]=true;//可以用质数就把所有的合数都筛掉；
        }
    }
}

3.线性筛法 O(n) 
void get_primes(){
    //外层从2~n迭代，因为这毕竟算的是1~n中质数的个数，而不是某个数是不是质数的判定
    for(int i=2;i<=n;i++){
        if(!st[i]) primes[cnt++]=i;
        for(int j=0;primes[j]<=n/i;j++){//primes[j]<=n/i:变形一下得到——primes[j]*i<=n,把大于n的合数都筛了就
        //没啥意义了
            st[primes[j]*i]=true;//用最小质因子去筛合数

            //1)当i%primes[j]!=0时,说明此时遍历到的primes[j]不是i的质因子，那么只可能是此时的primes[j]<i的
            //最小质因子,所以primes[j]*i的最小质因子就是primes[j];
            //2)当有i%primes[j]==0时,说明i的最小质因子是primes[j],因此primes[j]*i的最小质因子也就应该是
            //prime[j]，之后接着用st[primes[j+1]*i]=true去筛合数时，就不是用最小质因子去更新了,因为i有最小
            //质因子primes[j]<primes[j+1],此时的primes[j+1]不是primes[j+1]*i的最小质因子，此时就应该
            //退出循环，避免之后重复进行筛选。
            if(i%primes[j]==0) break;
        }
    }

} 
```
 
## 快速幂
``` cpp
快速幂之迭代版 O(n∗logb) 
#include<iostream>
using namespace std;
long long qmi(long long a,int b,int p)
{
    long long res=1;
    while(b)//对b进行二进制化,从低位到高位
    {
        //如果b的二进制表示的第0位为1,则乘上当前的a
        if(b&1) res = res *a %p;
        //b右移一位
        b>>=1;
        //更新a,a依次为a^{2^0},a^{2^1},a^{2^2},....,a^{2^logb}
        a=a*a%p;
    }
    return res;
}
int main()
{
    int n;
    cin>>n;
    while(n--)
    {
        cin.tie(0);
        ios::sync_with_stdio(false);
        int a,b,p;
        long long res=1;
        cin>>a>>b>>p;
        res = qmi(a,b,p);
        cout<<res<<endl;
    }
    return 0;
}
 
```

## KMP
``` cpp
#include <iostream>

using namespace std;

const int N = 100010, M = 10010; //N为模式串长度，M匹配串长度

int n, m;
int ne[M]; //next[]数组，避免和头文件next冲突
char s[N], p[M];  //s为模式串， p为匹配串

int main()
{
    cin >> n >> s+1 >> m >> p+1;  //下标从1开始

    //求next[]数组
    for(int i = 2, j = 0; i <= m; i++)
    {
        while(j && p[i] != p[j+1]) j = ne[j];
        if(p[i] == p[j+1]) j++;
        ne[i] = j;
    }
    //匹配操作
    for(int i = 1, j = 0; i <= n; i++)
    {
        while(j && s[i] != p[j+1]) j = ne[j];
        if(s[i] == p[j+1]) j++;
        if(j == m)  //满足匹配条件，打印开头下标, 从0开始
        {
            //匹配完成后的具体操作
            //如：输出以0开始的匹配子串的首字母下标
            //printf("%d ", i - m); (若从1开始，加1)
            j = ne[j];            //再次继续匹配
        }
    }

    return 0;
}

```

## topsort
```cpp
#include<iostream>
#include<queue>
#include<cstring>
using namespace std;
const int N=100010;
int h[N],e[N],ne[N],idx;
int n,m;
int out[N];
void Add(int a,int b)
{
    e[idx]=b;ne[idx]=h[a];h[a]=idx++;
}
int Q[N];
bool topsort()
{
    int hh=0,tt=-1;
    for(int i=1;i<=n;i++)
        if(out[i]==0)
            Q[++tt]=i;
    while(hh<=tt)
    {
        int t=Q[hh++];
        for(int i=h[t];i!=-1;i=ne[i])
        {
            int j=e[i];
            out[j]--;
            if(out[j]==0)
                Q[++tt]=j;
        }   
    }       
    return tt==n-1; 
}
int main()
{
    memset(h,-1,sizeof h);
    cin>>n>>m;
    for(int i=0;i<m;i++)
    {
        int a,b;cin>>a>>b;
        Add(a,b);
        out[b]++;
    }
    if(topsort()==false) cout<<-1;
    else
    {
        for(int i=0;i<n;i++) cout<<Q[i]<<" ";
    }
    return 0;
}
 
```

## spfa  
``` cpp
#include<iostream>
#include<queue>
#include<cstring>
using namespace std;

const int N=1e5+10;

#define fi first
#define se second

typedef pair<int,int> PII;//到源点的距离，下标号

int h[N],e[N],w[N],ne[N],idx=0;
int dist[N];//各点到源点的距离
bool st[N];
int n,m;
void add(int a,int b,int c){
    e[idx]=b;w[idx]=c;ne[idx]=h[a];h[a]=idx++;
}

int spfa(){
    queue<PII> q;
    memset(dist,0x3f,sizeof dist);
    dist[1]=0;
    q.push({0,1});
    st[1]=true;
    while(q.size()){
        PII p=q.front();
        q.pop();
        int t=p.se;
        st[t]=false;//从队列中取出来之后该节点st被标记为false,代表之后该节点如果发生更新可再次入队
        for(int i=h[t];i!=-1;i=ne[i]){
            int j=e[i];
            if(dist[j]>dist[t]+w[i]){
                dist[j]=dist[t]+w[i];
                if(!st[j]){//当前已经加入队列的结点，无需再次加入队列，即便发生了更新也只用更新数值即可，重复添加降低效率
                    st[j]=true;
                    q.push({dist[j],j});
                }
            }
        }
    }
    if(dist[n]==0x3f3f3f3f) return -1;
    else return dist[n];
}

int main(){
    scanf("%d%d",&n,&m);
    memset(h,-1,sizeof h);
    while(m--){
        int a,b,c;
        scanf("%d%d%d",&a,&b,&c);
        add(a,b,c);
    }
    int res=spfa();
    if(res==-1) puts("impossible");
    else printf("%d",res);

    return 0;
} 
```

##  prim
``` cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 510;
int g[N][N];//存储图
int dt[N];//存储各个节点到生成树的距离
int st[N];//节点是否被加入到生成树中
int pre[N];//节点的前去节点
int n, m;//n 个节点，m 条边

void prim()
{
    memset(dt,0x3f, sizeof(dt));//初始化距离数组为一个很大的数（10亿左右）
    int res= 0;
    dt[1] = 0;//从 1 号节点开始生成 
    for(int i = 0; i < n; i++)//每次循环选出一个点加入到生成树
    {
        int t = -1;
        for(int j = 1; j <= n; j++)//每个节点一次判断
        {
            if(!st[j] && (t == -1 || dt[j] < dt[t]))//如果没有在树中，且到树的距离最短，则选择该点
                t = j;
        }

        //2022.6.1 发现测试用例加强后，需要判断孤立点了
        //如果孤立点，直返输出不能，然后退出
        if(dt[t] == 0x3f3f3f3f) {
            cout << "impossible";
            return;
        }


        st[t] = 1;// 选择该点
        res += dt[t];
        for(int i = 1; i <= n; i++)//更新生成树外的点到生成树的距离
        {
            if(dt[i] > g[t][i] && !st[i])//从 t 到节点 i 的距离小于原来距离，则更新。
            {
                dt[i] = g[t][i];//更新距离
                pre[i] = t;//从 t 到 i 的距离更短，i 的前驱变为 t.
            }
        }
    }

    cout << res;

}

void getPath()//输出各个边
{
    for(int i = n; i > 1; i--)//n 个节点，所以有 n-1 条边。

    {
        cout << i <<" " << pre[i] << " "<< endl;// i 是节点编号，pre[i] 是 i 节点的前驱节点。他们构成一条边。
    }
}

int main()
{
    memset(g, 0x3f, sizeof(g));//各个点之间的距离初始化成很大的数
    cin >> n >> m;//输入节点数和边数
    while(m --)
    {
        int a, b, w;
        cin >> a >> b >> w;//输出边的两个顶点和权重
        g[a][b] = g[b][a] = min(g[a][b],w);//存储权重
    }

    prim();//求最下生成树
    //getPath();//输出路径
    return 0;
}
 
```

## gcd
```  cpp
#include<bits/stdc++.h>
using namespace std;
/*
求两个正整数 a 和 b 的 最大公约数 d
则有 gcd(a,b) = gcd(b,a%b)
证明：
    设a%b = a - k*b 其中k = a/b(向下取整)
    若d是(a,b)的公约数 则知 d|a 且 d|b 则易知 d|a-k*b 故d也是(b,a%b) 的公约数
    若d是(b,a%b)的公约数 则知 d|b 且 d|a-k*b 则 d|a-k*b+k*b = d|a 故而d同时整除a和b 所以d也是(a,b)的公约数
    因此(a,b)的公约数集合和(b,a%b)的公约数集合相同 所以他们的最大公约数也相同 证毕#
*/
int gcd(int a, int b){
    return b ? gcd(b,a%b):a; 
}

int main(){
    int n,a,b;
    cin>>n;
    while(n--) cin>>a>>b,cout<<gcd(a,b)<<endl;
    return 0;
}
 
```

## floyd
```cpp
#include <iostream>
using namespace std;

const int N = 210, M = 2e+10, INF = 1e9;

int n, m, k, x, y, z;
int d[N][N];

void floyd() {
    for(int k = 1; k <= n; k++)
        for(int i = 1; i <= n; i++)
            for(int j = 1; j <= n; j++)
                d[i][j] = min(d[i][j], d[i][k] + d[k][j]);
}

int main() {
    cin >> n >> m >> k;
    for(int i = 1; i <= n; i++)
        for(int j = 1; j <= n; j++)
            if(i == j) d[i][j] = 0;
            else d[i][j] = INF;
    while(m--) {
        cin >> x >> y >> z;
        d[x][y] = min(d[x][y], z);
        //注意保存最小的边
    }
    floyd();
    while(k--) {
        cin >> x >> y;
        if(d[x][y] > INF/2) puts("impossible");
        //由于有负权边存在所以约大过INF/2也很合理
        else cout << d[x][y] << endl;
    }
    return 0;
}

 
```

## dijkstra
```cpp
#include <cstring>
#include <iostream>
#include <algorithm>
#include <queue>//堆的头文件

using namespace std;

typedef pair<int, int> PII;//堆里存储距离和节点编号

const int N = 1e6 + 10;

int n, m;//节点数量和边数
int h[N], w[N], e[N], ne[N], idx;//邻接矩阵存储图
int dist[N];//存储距离
bool st[N];//存储状态

void add(int a, int b, int c)
{
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
}

int dijkstra()
{
    memset(dist, 0x3f, sizeof dist);//距离初始化为无穷大
    dist[1] = 0;
    priority_queue<PII, vector<PII>, greater<PII>> heap;//小根堆
    heap.push({0, 1});//插入距离和节点编号

    while (heap.size())
    {
        auto t = heap.top();//取距离源点最近的点
        heap.pop();

        int ver = t.second, distance = t.first;//ver:节点编号，distance:源点距离ver 的距离

        if (st[ver]) continue;//如果距离已经确定，则跳过该点
        st[ver] = true;

        for (int i = h[ver]; i != -1; i = ne[i])//更新ver所指向的节点距离
        {
            int j = e[i];
            if (dist[j] > dist[ver] + w[i])
            {
                dist[j] = dist[ver] + w[i];
                heap.push({dist[j], j});//距离变小，则入堆
            }
        }
    }

    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}

int main()
{
    scanf("%d%d", &n, &m);

    memset(h, -1, sizeof h);
    while (m -- )
    {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c);
    }

    cout << dijkstra() << endl;

    return 0;
} 
```

## 组合数
```cpp
#include<iostream>
using namespace std;
const int mod = 1e9+7;
long long f[2010][2010];
int main()
{
    //预处理
    for(int i=0;i<=2000;i++)
    {
        for(int j=0;j<=i;j++)
        {
            if(!j) f[i][j]=1;
            else f[i][j]=(f[i-1][j-1]+f[i-1][j])%mod;
        }
    }
    int n;
    cin>>n;
    while(n--)
    {
        int a,b;
        cin>>a>>b;
        printf("%ld\n",f[a][b]);
    }
}

 
```

## 中国剩余定理
中国剩余定理
设 x∈Zx∈Z，有

⎧⎩⎨⎪⎪⎪⎪⎪⎪x≡a1(modm1)x≡a2(modm2)⋮x≡an(modmn)
{x≡a1(modm1)x≡a2(modm2)⋮x≡an(modmn)
令 ⎧⎩⎨⎪⎪⎪⎪M=m1m2m3…mnMi=Mmiti是Mi关于mi的逆元，即Miti≡1(modmi){M=m1m2m3…mnMi=Mmiti是Mi关于mi的逆元，即Miti≡1(modmi)
则
x=∑i=1naiMiti
x=∑i=1naiMiti 
Code

```cpp

#include <iostream>

using namespace std;

typedef long long LL;

const int N = 15;

int n;
int a[N], m[N];

void exgcd(LL a, LL b, LL &x, LL &y) {
    if (!b) x = 1, y = 0;
    else {
        exgcd(b, a % b, y, x);
        y -= a / b * x;
    }
}
int main() {
    cin >> n;
    LL M = 1;
    for (int i = 0; i < n; ++ i) {
        cin >> m[i] >> a[i];
        M *= m[i];  //读入mi的同时计算M
    }
    LL res = 0;
    for (int i = 0; i < n; ++ i) {
        LL Mi = M / m[i];   //计算Mi = M/mi
        LL ti, y;
        //这一步是求逆元，根据逆元公式的衍生公式可以得到 ti * Mi + y * mi = 1
        exgcd(Mi, m[i], ti, y);
        res += a[i] * Mi * ti;  //计算的同时累加到res中（上述公式里有个sum需要累加）
    }
    cout << (res % M + M) % M << endl;  //对于任意x+kM都是满足要求的解，但目标是输出最小的正整数x，因此取模即可
    return 0;
}
 
```

## 前缀和
``` cpp
#include<iostream>
#include<cstdio>
using namespace std;
const int N=1e5+10;
int a[N],sum[N];
int main()
{
    int n,m,x;
    cin>>n>>m;
    for(int i=1;i<=n;i++)
    {
        cin>>x;
        sum[i]=x+sum[i-1];
    }
    while(m--)
    {
        int l,r;
        cin>>l>>r;
        cout<<sum[r]-sum[l-1]<<endl;
    }
    return 0;
}

 
```


```cpp
//前缀和
#include <iostream>
#include <cstdio>

using namespace std;

typedef long long LL;
const int N = 1e5+10;
int A[N], B[N], C[N];
int cnta[N], cntc[N], sa[N], sc[N];

int main() {
    int n;
    scanf("%d", &n);
    //获取数i在A中有cntc[i]个，并对cnt求前缀和sa
    for(int i = 1; i <= n; ++i) {
        scanf("%d", &A[i]);
        cnta[++A[i]]++;
    }
    sa[0] = cnta[0];
    for(int i = 1; i < N; ++i) sa[i] = sa[i-1]+cnta[i];
    //B只读取即可
    for(int i = 1; i <= n; ++i) scanf("%d", &B[i]), B[i]++;

    //获取数i在C中有cntc[i]个，并对cnt求前缀和sc
    for(int i = 1; i <= n; ++i) {
        scanf("%d", &C[i]);
        cntc[++C[i]]++;
    }
    sc[0] = cntc[0];
    for(int i = 1; i < N; ++i) sc[i] = sc[i-1]+cntc[i]; 

    //遍历B求解
    LL ans = 0;
    for(int i = 1; i <= n; ++i) {
        int b = B[i];
        ans += (LL)sa[b-1] * (sc[N-1] - sc[b]);
    }
    cout<<ans<<endl;
    return 0;
}
 
```

## 二分图最大匹配
```cpp
#include<iostream>
#include<cstring>
using namespace std;
const int N = 510 , M = 100010;
int n1,n2,m;
int h[N],ne[M],e[M],idx;
bool st[N];
int match[N];

void add(int a , int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void init()
{
    memset(h,-1,sizeof h);
}

int find(int x)
{
    //遍历自己喜欢的女孩
    for(int i = h[x] ; i != -1 ;i = ne[i])
    {
        int j = e[i];
        if(!st[j])//如果在这一轮模拟匹配中,这个女孩尚未被预定
        {
            st[j] = true;//那x就预定这个女孩了
            //如果女孩j没有男朋友，或者她原来的男朋友能够预定其它喜欢的女孩。配对成功
            if(!match[j]||find(match[j]))
            {
                match[j] = x;
                return true;
            }

        }
    }
    //自己中意的全部都被预定了。配对失败。
    return false;
}
int main()
{
    init();
    cin>>n1>>n2>>m;
    while(m--)
    {
        int a,b;
        cin>>a>>b;
        add(a,b);
    }


    int res = 0;
    for(int i = 1; i <= n1 ;i ++)
    {  
         //因为每次模拟匹配的预定情况都是不一样的所以每轮模拟都要初始化
          memset(st,false,sizeof st);
        if(find(i)) 
          res++;
    }  

    cout<<res<<endl;
}
 
```

## 二分
```cpp
#include <iostream>
using namespace std;
const int maxn = 100005;
int n, q, x, a[maxn];
int main() {
    scanf("%d%d", &n, &q);
    for (int i = 0; i < n; i++)    scanf("%d", &a[i]);
    while (q--) {
        scanf("%d", &x);
        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + r >> 1;
            if (a[mid] < x)  l = mid + 1;
            else    r = mid;
        }
        if (a[l] != x) {
            printf("-1 -1\n");
            continue;
        }
        int l1 = l, r1 = n;
        while (l1 + 1 < r1) {
            int mid = l1 + r1 >> 1;
            if (a[mid] <= x)  l1 = mid;
            else    r1 = mid;
        }
        printf("%d %d\n", l, l1);
    }
    return 0;
}

 
```

## 单调队列
```cpp
# include <iostream>
using namespace std;
const int N = 1000010;
int a[N], q[N], hh, tt = -1;

int main()
{
    int n, k;
    cin >> n >> k;
    for (int i = 0; i < n; ++ i)
    {
        scanf("%d", &a[i]);
        if (i - k + 1 > q[hh]) ++ hh;                  // 若队首出窗口，hh加1
        while (hh <= tt && a[i] <= a[q[tt]]) -- tt;    // 若队尾不单调，tt减1
        q[++ tt] = i;                                  // 下标加到队尾
        if (i + 1 >= k) printf("%d ", a[q[hh]]);       // 输出结果
    }
    cout << endl;
    hh = 0; tt = -1;                                   // 重置！
    for (int i = 0; i < n; ++ i)
    {
        if (i - k + 1 > q[hh]) ++ hh;
        while (hh <= tt && a[i] >= a[q[tt]]) -- tt;
        q[++ tt] = i;
        if (i + 1 >= k) printf("%d ", a[q[hh]]);
    }
    return 0;
}
 
```
## 差分
``` cpp
#include<iostream>
using namespace std;

const int N = 1e5 + 10;

int n, m;
int b[N];

void insert(int l, int r, int c)
{
    b[l] += c;
    b[r+1] -= c;
}

int main()
{
    cin >> n >> m;
    for(int i=1; i<=n; i++)
    {
        int x;
        cin >> x;
        insert(i, i, x);
    }

    while(m--)
    {
        int l, r, c;
        cin >> l >> r >> c;
        insert(l, r, c);
    }

    for(int i=1; i<=n; i++)
    {
        b[i] = b[i-1] + b[i];
        cout << b[i] << " ";
    }

    return 0;
}
 

```