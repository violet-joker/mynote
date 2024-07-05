

## 一、基础

### 语法基础：

函数， 指针， 引用， 结构体、类

### 算法基础

##### 模拟

题目怎么说，代码就怎么写。

##### 递归、回溯

全排列

```c++
/*
用数组a储存每一次排列的情况，用b来判断某个数是否储存过
递归过程：
依次往a里填数，每填一个数，需要排列的数就会减一，也就是说，n个数排列转换成n-1个数排列
直到n个数都填进数组a，则递归结束，打印数组a，即此次排列情况。而index即已经填进a的元素个数。
每一次递归，前index元素都已经选好数了，后n-index个元素待排序。
*/
int a[Max], n;
bool b[Max];

void arrange(int index){
    if(index == n+1){						//n个数都已填完，递归终止
        for(int i = 1; i <= n; i++)
            printf("%d ",a[i]);
        printf("\n");
        return;
    }
    for(int i = 1; i <= n; i++)				//将不同的数插入到当前a[index]位置
        if(!b[i]){
            a[index] = i;
            b[i] = true;
            arrange(index+1);				//对后n-index个元素全排
            b[i] = false;					//a[index]为i的排列结束，取出a[index]里的值，换下一个'i'
        }
}

int main()
{
    scanf("%d", &n);
    arrange(1);
}
```

[递归实现指数型枚举](http://oj.daimayuan.top/course/13/problem/578)
[递归实现组合型枚举](http://oj.daimayuan.top/course/13/problem/579)
[递归实现排列型枚举](http://oj.daimayuan.top/course/13/problem/580)



##### 递推

##### 贪心

##### 二分

###### 1.二分查找

注意查找范围是（l，r）

```c++
int search(int key){
    int l = -1, r = n+1, m;
    while(l+1 < r){
        m = l+r>>1;
        if(a[m] == key)       return m;
        else if(a[m] < key)   l = m;
        else			     r = m;
    }
}
```

[二分查找 ](http://oj.daimayuan.top/problem/59)， [二分查找2](http://oj.daimayuan.top/problem/60)， [二分查找3 ](http://oj.daimayuan.top/problem/79)， [蜗牛的悄悄话](http://oj.daimayuan.top/problem/33)

```c++
//实数二分
double search(double key){
    double l = 0, r = n+1, m;
    while(r-l < eps){
        m = l+r>>1;
        if(m == key)        return m;
        else if(m < key)    l = m;
        else                r = m;
	}
}

//然而某些情况下仍会发生莫名奇妙的精度问题，所以最好使用下面这种二分
double search(double key){
	double l = 0, r = n+1, m;
    for(int i = 0; i < 100; i++){
        m = l+r>>1;
        if(m == key)     return m;
        else if(m < key) l = m;
        else             r = m;
	}
}
```



###### 2.二分答案

假设答案为ans，ans在区间(l,r)内；且对于大于（小于）ans的数都能满足条件，小于（大于）ans的数都不能满足条件，此时即可对ans二分，找出满足条件的最大（最小）的答案。
一般是求最小的最大值，或者最大的最小值。

```c++
//例：求最大的最小值
int solve(int l, int r){
    int m;
    while(l+1 < r){
        m = l+r>>1;
        if(check(m)) r = m;
        else         l = m;
    }
    return r;
}
```

[二分答案](http://oj.daimayuan.top/problem/61)，  [吃糖果 ](http://oj.daimayuan.top/course/11/problem/863)



##### 高精度

模拟竖式计算，用字符串读取数据，用数组保存结果

###### 1.高精度加法

```c++
void solve(string a, string b, int *ans){
    int n = a.size(), m = b.size(), k = max(n, m);
    for(int i = n-1; i >= 0; i--)
        ans[n-i-1] = a[i] - '0';
   	for(int i = m-1; i >= 0; i--)
        ans[m-i-1] += b[i] - '0';
   	for(int i = 0; i < k; i++)
        ans[i+1] += ans[i]/10, ans[i] %= 10;
   	if(ans[k]) k++;
    while(k--) printf("%d", ans[k]);
}
```

[大数加法 ](http://oj.daimayuan.top/problem/158)

###### 2.高精度乘法

```c++
/*模拟a*b的竖式计算，先不考虑进位，则一定有len-1位
数，把竖式计算的n层加起来后再依次进位。每一层计算都
会进一位，用k来记录进位情况。
举个例子：a = 21, b = 121
 121
x 21
-----
 121          第一层, k = 0
242           第二层, k = 1
-----
2541
*/
void solve(string a, string b, int *ans){
    int n = a.size(), m = b.size(), len = n+m;
    for(int i = n-1, k = 0; i >= 0; i--, k++)
        for(int j = m-1; j >= 0; j--)
            ans[n-i-1+k] += (a[i]-'0') * (b[j]-'0');    //每一位求和
    for(int i = 0; i < len; i++)
        ans[i+1] += ans[i]/10, ans[i] %= 10;            //模拟进位(len位如果存在，可以不用进位)
    while(!ans[len] && len>0) len--;                    //去掉前导零(防止乘0的情况)
    for(int i = len; i >= 0; i--)
        printf("%d", ans[i]);
}
```

[大数乘法2](http://oj.daimayuan.top/problem/160)

###### 3.高精度除法

模拟减法，高位对齐再逐渐相减，时间复杂度O(n²)

```c++
/*
eg:224 / 11
  224
- 11          
------
  114
- 11
-------
    4
 在十位上减了两次，个位上减了0次，所以最终结果为20，余数为4
*/
void div(string a, string b, int *ans){  //默认输入数据为非负数
    int n = a.size(), m = b.size();
    if(b == "0"){      //被除数不能为0
        cout<<"erorr\n";
        return;
    int A[Max] = {}, B[Max] = {}, k = n-m, len = k;        //用k模拟对齐，len记录k初始值
    for(int i = n-1; i >= 0; i--) A[n-i-1] = a[i]-'0';     //逆序存储a，b
    for(int i = m-1; i >= 0; i--) B[m-i-1] = b[i]-'0';
    while(n >= m){
        while(cmp){     //判断是否可以相减
            for(int i = k; i < n; i++){
                A[i] -= B[i-k];
                if(A[i] < 0)
                    A[i] += 10, A[i+1] -= 1;
            }
            ans[K]++;
        }
        n--, k--;
        A[n-1] += A[n]*10;   //把上一最高位剩余的部分加进来，eg:100/3,  1/3不够,下一位变成10/3
    }
    while(len>0 && !ans[len]) len--;  //去掉前导零
    for(int i = len; i >= 0; i--) printf("%d", ans[i]);
}
    
bool cmp(int *A, int *B, int n, int k){   //返回真则表示对齐部分A>=B
    for(int i = n-1; i >= k; i--)
        if(A[i] > B[i-k]) return true;
    	else if(A[i] < B[i-k]) return false;
    return true;
}
```



##### 位运算

基本内容：
两数相与(&)，二进制对应位都为1，则结果为1，否则为0；
两数相或(|)，二进制对应位都为0，则结果位0，否则为1；
两数异或(^)，二进制对应位相同，则结果为0，否则为1。

位运算有时是一种巧妙的思路，一点点总结吧
a & 1可判断a奇偶性；
a ^ a == 0，0 ^ a == a；
a ^ b最高位与a,b较大的数一致，与较小的数相反；
a ^ b = c   --->  a ^ b ^ b = c ^ b  --->  a = c ^ b 

[只出现一次的数字2](http://oj.daimayuan.top/course/22/problem/899)



##### 时空复杂度分析



##### 排序

###### 1.选择排序

依次选出最大/最小值

```c++
//时间复杂度O(n^2)
for(int i = 0; i < n; i++)
	for(int j = i+1; j < n; j++)
		if(a[i] < a[j])
			swap(a[i], a[j]);
```

###### 2.冒泡排序

确保相邻两项有序，n个数，每个数遍历n次即可确保一定有序，原理：每次交换相邻两数，逆序数发生改变（+1 或-1）

```c++
//时间复杂度O(n^2)
for(int i = 0; i < n; i++)
	for(int j = 0; j < n-1; j++)
		if(a[j] < a[j+1])
         	swap(a[j], a[j+1]);
```

###### 3.插入排序

模拟理清扑克牌，将牌依次插入合适的位置

```c++
//时间复杂度O(n^2)
for(int i = 0; i < n; i++){
	int j = i, tmp = a[i];
    while(j > 0 && a[j-1] < tmp){
        swap(a[j-1], a[j]);
        j--;
	}
}
```

###### 4.快速排序

分治，将数组以tmp为界限分成两部分，左边一定大于（小于）tmp， 右边一定小于（大于）tmp，直到只有一个数。

```c++
//平均时间复杂度O(nlog(n)),最糟情况为O(n^2)
int a[Max];
void sort(int begin, int end){
    if( begin >= end) return;
    int i = begin;
    int j = end;
    int tmp = a[i];
    while(i < j){
        while(i < j && a[j] >= tmp)
            j--;
            a[i] = a[j];
        while(i <j && a[i] <= tmp)
            i++;
        	a[j] = a[i];
    }
    a[i] = tmp;
    sort(begin, j-1);
    sort(i+1,end);
}
```

###### 5.归并排序

将两个有序数组合并成一个数组的时间复杂度为O(n)，而归并就是分治下去，使得begin到mid有序，mid+1到end有序，再将两部分合并，使得begin到end有序。

```c++
//时间复杂度O(nlog(n)),需要额外的空间来辅助排序
int a[Max], b[Max];
void MergeSort(int begin, int end){
    if(begin >= end) return;
    int mid = begin+end >> 1;
    MergeSort(begin, mid);
    MergeSort(mid+1, end);
    Merge(begin, mid, end);
}

void Merge(int begin, int mid, int end){
    int p1 = begin, p2 = mid+1, p = begin;
    while(p1 <= mid && p2 <= end){
        if(a[p1] < a[p2])
            b[p++] = a[p1++];
        else
            b[p++] = a[p2++];
    }
    while(p1 <= mid) b[p++] = a[p1++];
    while(p2 <= end) b[p++] = a[p2++];
    for(int i = begin; i <= end; i++)
        a[i] = b[i];
}
```



###### 6.桶排序



###### 7.计数排序

利用数组下标是线性连续的特点进行排序，优点是排序快，缺点是只能处理小数据，且只能给整数排序

```c++
//时间复杂度为O(n+值域), 空间复杂度为T( max(a[i]) )
for(int i = 0; i < n; i++)
    b[a[i]]++;
for(int i = Min; i <= Max; i++)
    while(b[i]--)
        printf("%d ", i);
```



###### 8.堆排序（优先队列）

大根堆/小根堆 -- 完全二叉树
以大根堆为例（父亲大于儿子）
上浮：当父亲小于当前结点，交换数值，循环直到无法上浮。（上浮只需判断一次当前结点和父结点，因为父结点只有一个）
下沉：当儿子结点大于当前节点，交换数值，循环直到无法下沉。（下沉需要先找出最大的儿子，再判断是否下沉，确保换上来的数一定不小于另一个儿子）
自顶向下排序：
对1~n个元素依次上浮，得到大根堆，a[1]即为最大值，每次输出a[1]，交换a[1]与队列中最后一个元素，通过down操作维护大根堆

```c++
/*
插入操作即为上浮操作，单次操作时间复杂度为O(logn)，删除/弹出元素需要下沉操作维护序列，单词操作时间复杂度为O(logn);因此堆排序时间复杂度为O(nlogn)
*/
int n, a[Max] = {1,-2,3,-4,5,-6,7,-8,9,-10};

void up(int index){
    while(index/2 >= 1 && a[index] > a[index/2]){  //儿子大于父亲，交换
        swap(a[index], a[index/2]);
        index >>= 1;
    }
}
void down(int index){
    while(2*index <= n){
        int tmp = 2*index;
        if(tmp+1 <= n && a[tmp] < a[tmp+1]) tmp++;  //判断右儿子是否更合适
        if(a[index] < a[tmp])
            swap(a[index], a[tmp]), index = tmp;
        else
            break;
    }
}

int main()
{
    for(int i = 1; i <= n; i++) up(i);
    while(n){
        printf("%d ", a[1]);
        swap(a[1], a[n--]);
        down(1);
    }
}
```

[合并数列](http://oj.daimayuan.top/course/16/problem/493)



## 二、字符串

### KMP

KMP用于匹配主串与模式串，BF算法是对主串进行回溯，而KMP是对模式串进行回溯。核心其实就是求最大公共前后缀。
（PS：什么是公共前后缀？举个例子abaABCaba,注意加粗部分**a**baABCab**a**，a是该串的公共前后缀；再看**abc**ABC**abc**，最长公共前后缀为abc，长度为3。）
求nxt数组的时候，第一个元素对应的nxt值一定是0，表示没有匹配的字符，因此**自匹配求nxt数组时从下标2开始**

```c++
char s[Max], p[Max];
int nxt[Max], f[Max];

void KMP(void){
    scanf("%s%s", s+1, p+1);   //读入主串和模式串，从1开始方便元素项数与位置索引对应
    int n = strlen(s+1);
    int m = strlen(p+1);
    for(int i = 2, j = 0; i <= m; i++){    //j表示1~j项都匹配成功
    	while(j>0 && p[j+1]!=p[i])       //下一项不匹配，回溯j
            j = nxt[j];        //j回溯到以第j项结尾的最大匹配数，该数同时也是匹配到的项数，然后再循环判断该项的下一位
   		if(p[j+1] == p[i])    //如果匹配成功，j+1
            j++;
    	nxt[i] = j;   //nxt[i]表示以i结尾的最大匹配数，1~j项与i-j+1~i项都匹配
	}
	for(int i = 1, j = 0; i <= n; i++){  //匹配主串与模式串与上面模式串自匹配一样
        while((j>0 && p[j+1]!=s[i]) || (j == m)) j = nxt[j];
       	if(p[j+1] == s[i]) j++;
        f[i] = j;
	}
	for(int i = 1; i <= n; i++)
        if(f[i] == m)  //最大公共前后缀等于模式串的长度，即匹配成功
            printf("%d ", i - m + 1);  //减去相对长度，即匹配的起始位置
}
```

```c++
//由于主串匹配模式串与模式串自匹配一样，所以可以把主串拼接在模式串后面，用特定符号分割，再求最大公共前后缀数即可，
char s[Max], p[Max];
int nxt[Max];

void KMP(void){
    scanf("%s%s", s+1, p+1);
    int n = strlen(s+1);
    int m = strlen(p+1), k = m;
    p[++m] = '#';      //用一定不会出现的字符分割
    for(int i = 1; i <= n; i++)
        p[++m] = s[i];
    for(int i = 2, j = 0; i <= m; i++){    //遍历即求1~i这个串的最大公共前后缀
        while((j>0 && p[j+1]!=p[i]) || (j == k)) j = nxt[j];
        if(p[j+1] == p[i]) j++;
        nxt[i] = j;
        if(j == k) printf("%d ", i - 2*m);  // i - (m+1) - m + 1 = i - 2*m
	}
}
```



### 字符串哈希

将字符串看成一个P进制数，则该串的哈希值为某个P进制数%Q，当P取131或13331，Q取2^64时，哈希冲突最低（前辈总结出的经验数据），假定为没有冲突。可以用unsigned long long来储存哈希值，这样在计算的过程中**自然溢出**部分相当于%Q。
用h[i]表示从1~i的字符串前缀哈希值(左->右，高位->低位)，则h[i] = h[i-1] * P + s[i]; (‘A’可以映射为1，也可以映射为其ASCLL码，但不能映射为0，否则AA与A的哈希值一样)
求l到r的字符串哈希值可以由h[r]减去h[l-1] * p^(r-l+1)得出（区间长度差多少，就乘p的多少次幂）。
（若单哈希被卡，可采用双哈希，对于两种选取的P值，都能得出字符串哈希值相同，则字符串相同的概率几近100%）

```c++
//例：输入长度为n的字符串，m次询问，每次询问区间l~r是否与x~y相同
typedef unsigned long long ULL;
ULL h[N], p[N], P = 131;
int n, m, l, r, x, y;
char s[N];

int main() {
	scanf("%d%d%s", &n, &m, s+1);
    p[0] = 1;
    for (int i = 1; i <= n; i++)
        p[i] = p[i-1] * P, h[i] = h[i-1] * P + s[i];
    while (m--) {
		cin >> l >> r >> x >> y;
        ULL a = h[r] - h[l-1] * p[r-l+1];
        ULL b = h[y] - h[x-1] * p[y-x+1];
        if (a == b)
            cout << "Yes\n";
       	else
            cout << "No\n";
    }
}
```



### 字典树

### AC自动机



## 三、数据结构

### 队列

###### 普通队列

先进先出，只能队尾入，队头出

```c++
#include <queue>
queue<int> q;
q.push(x); //元素插入队尾
q.front(); //返回队头元素
q.pop();   //队头元素出队
```



###### 双端队列

可以从两端入队、出队

```c++
#include <deque>
deque<int> q;
q.push_back(x); //元素插入队尾
q.push_front(x); //元素插入队头
q.front(); //返回队头元素
q.back();  //返回队尾元素
q.pop_front(); //队头元素出队
q.pop_back();  //队尾元素出队
```



###### 单调队列

用双端队列实现 —— “如果一个选手比你小还比你强，你就可以退役了。”

求动态区间m内的最值，时间复杂度**O(n)**。
从头到尾扫描数组，用双端队列储存数组下标，便于判断是否在区间内。
队头储存区间内最值，队尾储存后续区间可能出现的最值。

```c++
/*
求以i结尾，区间不超过m的区间最值，由于求最大值，队列单调递减，队头为最大值
*/
int n, m, a[N], q[N], l, r;

for(int i = 1; i <= n; i++) {
    if (i - q[l] + 1 > m) l++;   //队头元素和a[i]的距离不能超过m
    // a[i]与a[q[l]]之间的距离等于 i - q[l] + 1; 大于 m 时出队
    while (l < r && a[i] > a[q[r-1]]) r--;   //a[i]比队尾更优，队尾出队
    q[r++] = i;              //a[i]入队
    if (i >= m) cout << a[q[l]] << " ";  //队头即区间最值
}
```

[动态区间最大数](http://oj.daimayuan.top/course/7/problem/509)

### 栈

###### 普通栈

先进后出

###### 单调栈

单调递增/递减栈，栈内元素有序，对于数组中的某一元素，查找左边或者右边第一个与其满足大小关系的元素，时间复杂度O(n)

```c++
/*
eg：寻找a[i]右边第一个大于a[i]的数，数组模拟栈，t指向栈顶。
找右边的数，因此从右往左遍历，当前遍历到的元素a[i]可能会满足左边
的元素，而如果栈里的元素如果不满足a[i]，那么一定不满足a[i]左边的
所有元素（至少不是最近），所以将其不满足的依次出栈。栈后进先出的特点
达成优先判断距离进的元素，栈顶是距离最近的。
排除不满足的元素后，若栈不为空，则栈顶元素即最近的满足条件的元素。
使用f数组辅助储存。
*/
int n, a[N], stack[N], t, f[N];

void solve(void) {
    for (int i = n; i; i--) {
        while (t && a[i] >= stack[t]) t--;  //a[i] >= 栈顶，若栈顶>a[j], 则a[i] >= 栈顶 > a[j]，而对于左边元素，a[i]更近；栈顶元素出栈
        if (t) f[i] = stack[t];
        else f[i] = -1;
        stack[++t] = a[i];  //此时栈为空或者a[i]小于栈顶，a[i]入栈 (a[j]与a[i]关系未知)
    }
    for (int i = 1; i <= n; i++)
        cout << f[i] << " ";
}
```

[求下一个更大数 - 题目 - Daimayuan Online Judge](http://oj.daimayuan.top/course/7/problem/506)



### 链表

普通链表，双向链表，循环链表

```c++
//普通链表用vector模拟即可
vector<int> a[N];
a[i].push(x);

// 数组模拟链表，用下标模拟地址，计数器nxt便于返回“地址”
int a[N], nxt;

int get_node() {
    nxt++;
    return nxt;
}
```



### 堆

普通堆，对顶堆
普通堆：大根堆/小根堆 -- 完全二叉树 -- 父结点大于(小于)儿子结点

```c++
//创建一个堆只需要上浮操作即可
void up(int index){
    while(index/2 >= 1 && a[index] < a[index/2])
        swap(a[index], a[index/2]), index >>= 1;
}
```



映射二叉堆：在普通堆的基础上加上 **初始元素位置**与**排序后堆中位置**的  **双向映射**，支持删除、修改第k个元素，

用h[i]表示堆中第i项第几次插入，p[k]表示第k个插入的元素在堆中的位置

```c++
/*
以小根堆为例，上浮和下沉操作仍是以堆中索引来操作
h[i] = k, p[k] = i, 两者互为逆运算
h[p[k]] = k, p[h[i]] = i
*/
int h[N], p[N], a[N], n;

void h_swap(int i, int j) {
    swap(p[h[i]], p[h[j]]);
    swap(h[i], h[j]);
    swap(a[i], a[j]);
}

void up(int k) {
    while (k/2 >= 1 && a[k] < a[k/2]) {
        h_swap(k, k/2);
        k >>= 1;
    }
}

void down(int k) {
    while (k+k <= n) {
        int tmp = k + k;
        if (tmp + 1 <= n && a[tmp+1] < a[tmp]) tmp++;
        if (a[tmp] < a[k])
            h_swap(tmp, k);
        else
            break;
    }
}

void change(int k, int x) {  //将第k个元素的值修改成x
	a[p[k]] = x;   //p[k]找到第k个值在堆中的位置
    up(p[k]);      //下沉、上浮操作使用堆中位置
    down(p[k]);
}

int main() {
	for (int i = 1; i <= n; i++) {
        cin >> a[i];
        p[i] = h[i] = i;  //p[i]表示第i个插入的元素，h[i]表示第i个元素在堆中的位置
        up(i);
    }
}
```



### 树

##### 树的基本概念

##### 树的遍历

##### 树的重心

##### 树的直径

##### LCA

(最近公共祖先)

###### 倍增法（在线求解LCA）



```c++
/* 
d[i]表示节点i所处的深度，f[x][i]表示点x向上跳2^i后的祖先节点编号
dfs预处理d数组和f数组
f[x][i]相当于跳了两次2^(i-1)，由此得出f数组状态转移方程
示例代码为有向树，若为无向树，需vis数组，在dfs时标记是否访问过
*/

// y是x的父节点，初始时设置根节点的父节点为0，作为哨兵
void dfs(int x, int y) {
    d[x] = d[y] + 1;
    f[x][0] = y;    // 2^0 = 1，跳1步，即父节点
    for (int i = 1; 1 << i <= d[x]; i++)
        f[x][i] = f[f[x][i-1]][i-1];  // 状态转移
    for (auto i : e[x]) dfs(i, x);    // dfs
}

int lca(int a, int b) {
    if (d[a] < d[b]) swap(a, b);    // 假设a深度大于b
    for (int i = 15; i >= 0; i--)   // 将a跳到和b相同的深度
        if (d[a] - (1 << i) >= d[b])
            a = f[a][i];
    if (a == b) return a;    // 若相等，则b是最近公共祖先
    for (int i = 15; i >= 0; i--)
        if (f[a][i] != f[b][i])  // 找到最远的同深度、值不相等的祖先，则父节点即为最近公共祖先
            a = f[a][i], b = f[b][i];
    return f[a][0];
}
```



求树上两点间的最短距离：假设求a、b两点的距离，p为最近公共祖先，则树上最短距离为dist[a] + dist[b] - 2 * dist[p];



##### DFN

##### 括号序



##### 平衡二叉树

###### treap

tree + heap，极端情况下二叉搜索树会形成一条链，导致搜索效率降低，通过给每个节点设置权值，再根据权值构建大/小根堆。当权值随机设置时，可以有效使得二叉树趋于平衡，从而维持logn级别的操作效率。理论上存在某种随机情况，使得即使所有权值随机设置仍导致形成一条链，但次概率很低。

treap树既满足二叉搜索树的定义（左子树严格小于根节点，右子树严格大于根节点），又满足堆的定义（以大根堆为例，父节点的权值大于等于儿子节点权值），而堆的结构是固定的，因此当一颗treap树确定时，树的结构是唯一的。

左右旋操作画一下图即可理解，旋转操作并不改变节点大小顺序，即中序遍历顺序不变

这里的前驱定义为中序遍历该节点的前一个节点，也就是小于该节点的最大值；后驱定义为中序遍历后一个节点，也就是大于该节点的最小值。

注意，插入，旋转，删除都涉及到节点的变化，函数引用传值可以在更改根节点的同时更改父节点的儿子节点，其他未涉及到节点变化的节点可以不使用引用，不过使用引用开销更小，固定的格式也更不易出错，因此可以将所有使用节点的地点都设置成引用传值



```c++
#include <iostream>
#include <cstdlib>

using namespace std;
const int N = 1e5 + 10;
int n, op, x, t, idx;
struct Node {  //节点储存信息
    int l, r;  //左右儿子
    int key, val;  //key值和权值
    int cnt, size;  //当前节点key值个数，该棵子树包含的key值的个数
}tr[N];  //这里定义的全局变量tr数组，内部初始化为0，因此后续某些地方利用了这一特性没有细分

void pushup(int& p) {  //更新节点size （左右节点的size加上当前节点的cnt值）
    tr[p].size = tr[tr[p].l].size + tr[tr[p].r].size + tr[p].cnt;
}

int get_node(int key) {  //初始化节点，并返回节点编号
    tr[++idx].key = key;
    tr[idx].val = rand();  //设置随机val用于构建大根堆
    tr[idx].cnt = tr[idx].size = 1;
    return idx;
}


void zig(int& p) {   //右旋
    int q = tr[p].l;
    tr[p].l = tr[q].r, tr[q].r = p, p = q;
    pushup(tr[p].r), pushup(p);  //旋转后先更新子树，再更新节点
}

void zag(int& p) {  //左旋
    int q = tr[p].r;
    tr[p].r = tr[q].l, tr[q].l = p, p = q;
    pushup(tr[p].l), pushup(p);
}

void init(void) {  //初始化添加正负无穷节点，利于处理寻找前驱、后驱无结果的情况
    t = get_node(-1e9);
    tr[t].r = get_node(1e9);
    pushup(t);
    if (tr[t].val < tr[tr[t].r].val) zag(t);
}

void insert(int& p, int key) {   //插入数据
    if (!p) p = get_node(key);   //未找到key值节点，初始化创建一个节点
    else if (tr[p].key == key) tr[p].cnt++;  //找到key值节点
    else if (tr[p].key > key) {  //节点值大了，往左子树找
        insert(tr[p].l, key);
        if (tr[p].val < tr[tr[p].l].val) zig(p);  //插入后如果左儿子权值更大，右旋
    } else {
        insert(tr[p].r, key);
        if (tr[p].val < tr[tr[p].r].val) zag(p);  //右儿子权值更大，左旋
    }
    pushup(p);  //别忘了更新节点！虽然旋转操作里有更新操作，但若没旋转，这里的更新则起到作用。
}

void remove(int& p, int key) {  //删除一个key值
    if (!p) return;  //未搜索到key，直接返回即可
    else if (tr[p].key < key) remove(tr[p].r, key);  //节点小了，往右子树找
    else if (tr[p].key > key) remove(tr[p].l, key);  //节点大了，往左子树找
    else {  //找到key值节点，需要分情况判断
        if (tr[p].cnt > 1) tr[p].cnt--; //cnt > 1，cnt--就行，否则将权值更大的子树旋转上来
        else if (!tr[p].l && !tr[p].r) p = 0;  //左右子树为空，直接删掉当前节点即可（置零）
        else if (tr[tr[p].l].val < tr[tr[p].r].val) zag(p), remove(tr[p].l, key);  //右儿子权值大，左旋
        else zig(p), remove(tr[p].r, key);  //左儿子权值大，右旋；记得旋转后删除原来的p节点
    }
    pushup(p);  //更新节点大小
}

int get_rank_by_key(int& p, int key) {  //求key排第几
    if (!p) return -1;  //未找到key
    else if (tr[p].key == key) return tr[tr[p].l].size + 1;  //找到key，排名为左子树大大小+1
    else if (tr[p].key > key) return get_rank_by_key(tr[p].l, key);  //节点大了，往左儿子找
    else return tr[tr[p].l].size + tr[p].cnt + get_rank_by_key(tr[p].r, key);  //节点小了，往右子树找
}

int get_key_by_rank(int& p, int rank) {  //已知排名求key
    if (tr[tr[p].l].size >= rank) return get_key_by_rank(tr[p].l, rank);  //左子树size大于等于rank，继续往左子树找
    else if (tr[tr[p].l].size + tr[p].cnt >= rank) return tr[p].key;  //rank在区间(左子树size，左子树size + 当前节点cnt]，则key排名在当前节点
    else return get_key_by_rank(tr[p].r, rank - tr[tr[p].l].size - tr[p].cnt); //rank > (左子树size + 当前节点cnt)，在右子树继续找剩余排名
}

int getFront(int& p, int key) {  //找前驱，小于key的最大值
    if (!p) return -1e9;  //搜索到空节点，设置最小，则不会影响最大值
    else if (tr[p].key >= key) return getFront(tr[p].l, key);  //节点大于等于key，则前驱在左子树里
    else return max(tr[p].key, getFront(tr[p].r, key));  //答案为当前节点，或者在右子树
}

/*
模拟查找4和6的前驱
   7
 /
3
 \
  5
*/

int getBack(int& p, int key) {  //找后继，大于key的最小值，同理找前驱
    if (!p) return 1e9;
    else if (tr[p].key <= key) return getBack(tr[p].r, key);
    else return min(tr[p].key, getBack(tr[p].l, key));
}

void dfs(int& p) {   //中序遍历
    if (!p) return;
    dfs(tr[p].l);
    cout << tr[p].key << " ";
    dfs(tr[p].r);
}

int main() {
    scanf("%d", &n);
    init();
    while(n--) {
        scanf("%d%d", &op, &x);
        if (op == 1) insert(t, x);
        else if (op == 2) remove(t, x);
        else if (op == 3) printf("%d\n", get_rank_by_key(t, x) - 1);
        else if (op == 4) printf("%d\n", get_key_by_rank(t, x + 1)); 
        else if (op == 5) printf("%d\n", getFront(t, x));
        else printf("%d\n", getBack(t, x));
    }
}
```







##### 线段树

###### 基础线段树

便于动态修改区间元素以及区间求和，查询区间最值， 时间复杂度**O(nlogn)**

```c++
/*
线段树其实就是棵完全二叉树，第1个节点储存数组a[1]~a[n]的元素和，取m = l+r >> 1;
左儿子储存l~m的元素和，右儿子储存m+1~r的元素和，递归下去完成建树
而修改a数组中的一个元素时，对于包含这个元素的所有区间都需要修改，单次修改时间复杂度为O(logn)

*/
int n, a[Max], f[Max];  //输入n个数据，数组a用于读取数据，数组f组织线段树，数组f的空间至少开4n

void buildTree(int k, int l, int r){  //k表示第几个节点，f[k]表示第k个节点储存的l~r的区间和
   	if(l == r){   //l == r，到最底层了，区间只有一个元素，递归终止
        f[k] = a[l];
        return;
	}
    int m = l+r >> 1;
    buildTree(k+k, l, m);  //求左儿子
    buildTree(k+k+1, m+1, r); //求右儿子
    f[k] = f[k+k] + f[k+k+1]; //该节点对应的区间和等于左右儿子的和
}

void add(int k, int l, int r, int x, int y){  //x表示修改的数的索引，y表示修改大小
    f[k] += y;   //修改该区间
    if(l == r)   //到达最底层，递归结束
        return;
    int m = l+r >> 1;
    if(x <= m)   //x在左区间，修改左儿子
        add(k+k, l, m, x, y);
    else         //x在右区间，修改右儿子
        add(k+k+1， m+1, r, x, y);
}

int calc(int k, int l, int r, int x, int y){  //求区间[x, y]的和
    if(l==x && r==y)  //f[k]正好是[x, y]的区间和，返回f[k],递归结束
        return f[k];
   	int m = l+r >> 1;
    if(y <= m)        //[x, y]在左儿子区间
        return calc(k+k, l, m, x, y);
    else if(x > m)    //[x, y]在右儿子区间
        return calc(k+k+1, m+1, r, x, y);
    else              //[x, y]横跨左右儿子区间
        return calc(k+k, l, m, x, m) + calc(k+k+1, m+1, r, m+1, y);
}
```

[动态区间最大数](http://oj.daimayuan.top/course/7/problem/509)



###### 带标记的线段树



##### 树状数组（Binary Index Tree）

树状数组基础

单点修改，动态维护区间和

![img](https://img-blog.csdnimg.cn/20210703110058974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoZVdheUZvckRyZWFt,size_16,color_FFFFFF,t_70)

f[i]表示从1~i的区间和，单点添加时，a[i]直接加到f[i]

lowbit，x取反+1，即-x， 从x的最末位1开始，其右全部相同，其左全部相反，两数相与即可仅保留x最末位1

树状数组的性质：i的父节点为 i + lowbit(i)，i - lowbit(i) 到达左边兄弟节点(上一层节点)（分析 t[6]到 t[4]）

```c++
/*
举个例子，-1010 的二进制为 0101+1 ---> 0110
	1010
&	0110
---------
	0010
*/
int lowbit(int x){
    return x & (-x);
}

void add(int i, int x){
    while(i <= n){
        f[i] += x;
        i += lowbit(i);
    }
}

int ask(int i){
    int res = 0;
    while(i){
        res += f[i];
        i -= lowbit(i);
	}
    return res;
}
```







树状数组求第k大，

多维树状数组



##### 树链剖分

重链剖分

##### 笛卡尔树

##### 树的启发式合并

##### DSU on tree



##### 并查集

并查集基础：

并查集其实就是用树的结构来表示一个集合；
查寻是否在同一集合，只需查询根节点是否相同；
合并即将根结点合并。

```c++
int f[Max], n;

void initialize(int n){ //初始化,开始每一个元素自身即为根结点(结点数为1的树)
    for(int i = 1; i <= n; i++)
        f[i] = i;
}

int findset(int x){   //查找x所在集合的根结点,当父亲结点等于自身即达到根结点
    return  x == f[x] ? x : f[x] = findset(f[x]);  //路径压缩,使得每一个集合树扁平,使得单次查询时间复杂度为O(1)
}

void Union(int x, int y){ //将x,y所在集合合并
    int fx = findset(x);  //查找x,y的根结点
    int fy = findset(y);
    f[fx] = fy;   //将根结点x并入y
}
```

[连通块计数](http://oj.daimayuan.top/course/14/problem/629)



启发式合并：小集合并入大集合(按大小合并)，可以使得单次查询时间复杂度为O(logn)

(启发式合并不会破坏树的结构，支持撤回操作)

```c++
int n, f[Max], sz[Max];  //sz数组记录每个集合的结点数

void initialize(int n){
    for(int i = 1; i <= n; i++)
        f[i] = i, sz[i] = 1;   //初始结点数都是1
}

void findset(int x){
    return x == f[x] ? x : findset(f[x]);
}

void Union(int x, int y){
    int fx = findset(x);
    
    int fy = findset(y);
    if(sz[fx] > sz[fy])
        swap(fx, fy);    //确保x集合小于y集合
    f[fx] = fy;          //将x并入y
    sz[fy] += sz[fx];    //x集合的结点数加到y集合
}
```

带权的并查集

##### 分块

莫队，链上分块

### STL

pair，vector，priority_queue，set，map，multiset，bitset

```c++
/*
使用范围for循环遍历容器，同时想修改容器内元素时，需要引用，否则修改的只是拷贝的副本
老忘，debug大半天......
*/
for (auto i : tmp)
for (auto &i : tmp)
```



```c++
//优先队列默认大根堆(较大的数在前面)，用以下方式定义小根堆
priority_queue<int, vector<int>, greater<int> > q;
```



```c++
map<int, int> mp;

void test(void) {
    auto it = mp.lower_bound(x);  //找到大于等于x的键值对
   	it = mp.upper_bound(x);  //找到大于x的键值对
    it = prev(it, 1);   //返回it迭代器前1个元素
    it = advance(it, 1);  //返回it迭代器后1个元素
    it->first;   //键
    it->second;  //值
}
```

```c++
bitset<N> a[M], b;   //设置M个长度为N的bitset
b.set(i);
b.count();
```

```c++
set<int> st;
st.insert(key);  //插入数据
st.erase(key);   //删除数据
*st.begin();     //获得第一个元素（set从小到大排序 ）
```



### ST表

通过预处理，解决静态RMQ(区间查询)问题，尤其是可重复贡献问题。例如区间按位与，区间按位和，区间最值，区间gcd

```c++
/*
这里以求区间最大值为例，
利用位运算倍增，区间长度分别为1 2 4 8 、、、
因此深度为logn，预处理时间复杂度为O(nlogn)
f[i][k]表示以i为起点，长度为2^k区间的最大值
在查询时，求以L为起点和以R为终点，长度为2^k区间的最值，
！！！ 对于可重复贡献问题，重合区间不会影响结果 ！！！
因此单次查询时间复杂度为O(1)
*/
int a[N], f[N][M];

void init() {
	for (int i = 1; i <= n; i++)
        f[i][0] = a[i];  //区间长度为1
    for (int k = 1; (1 << k) <= n; k++)  //区间长度不超过n
        for (int i = 1; i + (1 << k) - 1 <= n; i++) 
            // 左半区，右半区取最大值
            f[i][k] = max(f[i][k-1], f[i + (1 << (k-1))][k-1]);
}

/*
1 2 3 9 5 6 7
return max(max(1, 2, 3, 9), max(9, 5, 6, 7));
max属于可重复贡献，判断的两个区间有元素重合，但不影响
*/
int query(int l, int r) {
    // 向下取整
	int k = log2(r - l + 1);
    return max(f[l][k], f[r - (1 << k) + 1][k]);    //[l, t1]， [t2, r], t2 <= t1 <= r
}
```

也可求区间和，类似线段树，对需要求解的区间递归下去，每次找最大的k，直到求完区间，单次查询时间复杂度O(logn)。然而这种问题用树状数组求解更好。





## 四、动态规划

### 记忆化搜索

### 动态规划基础

最优子结构，无后效性，状态转移

##### 最长上升子序列

```c++
/*
dp时间复杂度 O(n^2)
f[i]表示以i结尾的最长上升子序列的最大长度，
f[i]由f[j]转移过来，
转移条件为i > j && a[i] > a[j]
*/
int LIS(void) {
    int res = 0;
    for (int i = 1; i < n; i++) {
        cin >> a[i];
        f[i] = 1;
        for (int j = 1; j < i; j++)
            if (a[i] > a[j])
                f[i] = max(f[i], f[j]+1);
        res = max(res, f[i]);
	}
    return res;
}

```

[最长上升子序列](http://oj.daimayuan.top/course/5/problem/135)

```c++
/*
贪心+二分优化,时间复杂度 O(nlog(n))
用f数组记录最长上升子序列的各个元素，用较小的元素更新数组f
*/
int LIS(void) {
    for (int i = 1; i <= n; i++) {
		cin >> a[i];
        if (a[i] > f[cnt])
            f[++cnt] = a[i];
       	else {
            int l = 0, r = cnt+1, m;
            while (l+1 < r) {
                m = l+r >> 1;
                if (f[m] <= a[i])
                	l = m;
               	else
                   	r = m;
            }
            if (f[l] != a[i])
                f[r] = a[i];
        }
    }
    return cnt;
}
```



##### 最长公共子序列

```c++

/*
最长公共子序列
f[i][j]表示以a[i],b[j]为结尾元素最长公共子序列长度
状态转移方程：
当结尾元素相同
f[i][j] = f[i-1][j-1]+1
当结尾元素不同,则取以a[i]结尾和以b[j]结尾的最大值
f[i][j] = max(f[i][j-1], f[i-1][j])
*/
int n, m, a[1010], b[1010], f[1010][1010];

void solve(){
    cin>>n>>m;
    for(int i = 1; i <= n; i++) cin>>a[i];
    for(int i = 1; i <= m; i++) cin>>b[i];
    for(int i = 1; i <= n; i++)
        for(int j = 1; j <= m; j++)
            if(a[i] == b[j])
                f[i][j] = f[i-1][j-1]+1;
            else
                f[i][j] = max(f[i-1][j], f[i][j-1]);
    cout<<f[n][m];
}
```

[最长公共子序列](http://oj.daimayuan.top/course/5/problem/136)



### 背包问题

##### 1.01背包

n个物品，背包空间为m，每个物品体积为v，收益为w，每个物品只能选一次或不选，求最大收益。

二维数组f用行数来模拟n次选择，用列数来模拟空间的情况。对于第i个物品可以选择取或不取，所以空间为j的状态可以转移为第i-1次取加上i的受益；或者不取，转移为i-1的受益。

```c++
int f[Max][Max], v[Max], w[Max], n, m;
int solve(void){
    for(int i = 1; i <= n; i++)
        for(int j = 1; j <= m; j++)
            if(j >= v[i])
                f[i][j] = max(f[i-1][j], f[i-1][j-v[i]]+w[i]);
            else
                f[i][j] = f[i-1][j];
    return f[n][m];
}
```

[01背包](http://oj.daimayuan.top/course/5/problem/161)



##### 2.完全背包

n种物品，背包空间为m，每种物品体积为v，收益为w，每种物品可以选择多次或不选，求最大收益。

一维数组模拟背包空间，对空间i考虑是否可以取第j个物品。状态转移为空间为i - v的收益 + w

```c++
int f[Max], v[Max], w[Max], n, m;
int solve(void){
	for(int i = 1; i <= m; i++){
        f[i] = f[i-1];
        for(int j = 1; j <= n; j++)
            if(i >= v[j])
                f[i] = max(f[i], f[i-v[j]]+w[j]);
	}
    return f[m];
}
```

[完全背包](http://oj.daimayuan.top/course/5/problem/162)



##### 3.多重背包

O(mnl)、O(nmlogl)、O(nm)

##### 4.树形背包

O(n^3)， O(n^2)

##### 5.分组背包

##### 6.混合背包

##### 7.多维背包

### 区间dp

基础

不断扩展区间长度，大区间由小区间转移过来

```c++
/*
最长回文子串
f[i][j]表示i~j是否为回文串
状态转移：
若a[i]==a[j]且f[i+1][j-1]是回文串,则f[i][j]是回文串
先预处理,使得当a[i-1]==a[i]时,f[i-1][i] = true (偶回文串),以及f[i][i] = true (奇回文串)
然后遍历每一个元素并向两边扩
*/
int n, a[1010], ans;
bool f[1010][1010];

int main()
{
    cin>>n;
    for(int i = 1; i <= n; i++){
        cin>>a[i];
        f[i][i] = true;
    }
    for(int i = 2; i <= n; i++)
        if(a[i-1]==a[i]) f[i-1][i] = true;
    for(int i = 1; i <= n; i++)  //i表示扩的长度
        for(int j = 1; j <= n; j++){  //遍历每一个元素
            int l = j, r = j+i;
            if(f[l+1][r-1] && a[l]==a[r]) f[l][r] = true;
        }
    for(int i = 1; i <= n; i++)
        for(int j = i; j <= n; j++)
            if(f[i][j]) ans = max(ans, j-i+1);  //找出最长回文串长度
    cout<<ans;
}
```

[最长回文子串](http://oj.daimayuan.top/course/5/problem/137)



```c++
/*
f[i][j]表示区间i~j合并的最小消耗
状态转移：f[i][j] = f[i][k] + f[k+1][r] + (a[r] - a[l-1])
找到一个k，使得两部分合并的消耗最小
*/
#include <iostream>

using namespace std;
const int N = 510;
int n, a[N], f[N][N];

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        a[i] += a[i-1];    //前缀和预处理合并成一个区间的消耗
    }
    for (int len = 2; len <= n; len++)   //枚举区间长度
    for (int i = 1; i+len-1 <= n; i++) {
        int l = i, r = i+len-1;  //枚举左右区间
        f[l][r] = 1e9;
        for (int k = l; k < r; k++)  //枚举左右区间内k的取值
            f[l][r] = min(f[l][r], f[l][k] + f[k+1][r] + a[r]-a[l-1]);
    }
    cout << f[1][n];
}
```

[石子合并](http://oj.daimayuan.top/course/5/problem/199)



```c++
//环形石子合并
#include <iostream>

using namespace std;
const int N = 510;
int n, a[N], f[N][N], res = 1e9;

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        a[i+n] = a[i];   //重复拼接一次形成环
    }
    for (int i = 1; i <= n+n; i++) a[i] += a[i-1];
    for (int len = 2; len <= n; len++)
    for (int i = 1; i+len-1 <= n+n; i++) {
        int l = i, r = i+len-1;
        f[l][r] = 1e9;
        for (int k = l; k < r; k++)
            f[l][r] = min(f[l][r], f[l][k] + f[k+1][r] + a[r]-a[l-1]);
    }
    for (int i = 1; i <= n; i++)
        res = min(res, f[i][i+n-1]);
    cout << res;
}
```

[石子合并2](http://oj.daimayuan.top/course/5/problem/201)



进阶

### 数位dp

基础

时间复杂度为O(状态数*转移复杂度)

进阶

### 树形dp（普通型，换根型）

基础，

###### 求最长树的直径

树的直径指由某一节点到其最远的节点的长度 （也可以是边的权值之和，可以为负数）。

假设a为某条直径上的节点，则要么是一条链一直往下走，要么是往下走的两条链，所以记录往下走的两条链即可（最大和次大）；往上走的作为父节点的考虑范围。

暴力求解，对每个节点都dfs直径，然后取最大值，时间复杂度O(n^2)；然而父节点可以由子节点状态转移，求出最大和次大权值链，从而求出以该节点作为局部根节点的直径，dfs一遍即可，时间复杂度O(n)

```c++
int n, a, b, c, res;
vecotr<pair<int, int>> e[N];


// 传入节点，和父节点
int dfs(int k, int fa) {
    // 记录最大和次大。若权值为负，则不取下面的链即可，所以初始化为0
    int d1 = 0, d2 = 0;
    for (auto i : e[k]) {
        if (i.first == fa) continue;  //确保向下搜索
        int d = dfs(i.first, k) + i.second;
        if (d > d1) d2 = d1, d1 = d;
        else if (d > d2) d2 = d;
    }
    res = max(res, d1 + d2);
    // 返回经过节点k的传值最大的一条链，为父节点计算做准备
    return d1;
}

int main() {
	cin >> n;
    // 无向连边建树
    for (int i = 1; i < n; i++) {
        cin >> a >> b >> c;
        e[a].push_back({b, c});
        e[b].push_back({a, c});
    }
    // 任选一个节点作为根节点，传入一个不存在的父节点开始搜索
    dfs(1, -1);
    cout << res << endl;
}
```



进阶，高级



### 状压dp

普通型，轮廓型，高维前缀和

### 概率dp

基础，进阶

### 计数dp

基础，进阶

### 其他

图上dp，基环树上dp

### dp优化

单调栈优化，单调队列优化，四边形不等式优化，斜率优化



## 五、图论

### 图的概念

### 图的遍历

### 拓扑排序

拓扑排序（有向无环图的所有顶点的线性序列）（时间复杂度**O(n+e)**）：每个顶点只出现一次；若存在A到B的路径，则序列中点A出现在点B前面。（拓扑序列不唯一，若按节点从小到大输出，可用优先队列）

```c++
/*
宽搜实现，先将所有入度为0的点入队，之后每遍历到一个点，更新其下一个节点的入度（入度减1），并将更新后入度为0的点入队
需要先遍历找出所有入度为0的点，再遍历每一条边，时间复杂度为 O(n+e)
如果遍历完队列，存在入度不为0的点，则不构成拓扑图
(d数组计数节点入度)
*/
vector<int> e[N];
int n, d[N];

void bfs() {
    queue<int> q;
    for (int i = 1; i <= n; i++)
		if (d[i] == 0) q.push(i);
    while (q.size()) {
        int t = q.front();
        q.pop();
        cout << t << " ";
        for (auto i : e[t]) {
			d[i]--;
			if (d[i] == 0) q.push(i);
        }
    }
    for (int i = 1; i <= n; i++)
        if (d[i]) cout << "error";
}
```



求有向无环图从每个点出发，可到达点的数量，可以将边逆过来，求逆拓扑序，需要注意的是，某些情况下不能直接贪心计算，比如a->b->c，a->c，不能f[c] += f[b], f[c] += f[a]，实际上a到c只经过三个点。可以用bitset位运算模拟遍历到的点的情况，f[c] |= f[b], f[c] |= f[a]，避免加入重复点。





### 传递闭包



### 最短路

#### 多源汇最短路

###### floyd

```c++
/*
dp实现，f(k,i,j)表示从i到j，只经过1~k作为中间点的所有路径的最短路
经过k号点：f[k][i][j] = f[k-1][i][k] + f[k-1][k][j]
不经过k号点：f[k][i][j] = f[k-1][i][j]
*/
for (int k = 1; k <= n; k++)
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= n; j++)
            f[k][i][j] = min(f[k-1][i][k] + f[k-1][k][j], f[k-1][i][j]);
```



```c++
/*
第k层由第k-1层转移过来，仅用f[i][j]表示经过1~n中间点，i到j的最短距离
状态转移：f[i][j] = min(f[i][j], f[i][k] + f[k][j])
*/
for (int k = 1; k <= n; k++)
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= n; j++)
            f[i][j] = min(f[i][j], f[i][k] + f[k][j]);
```



#### 单源最短路

###### dijkstra

基于贪心实现，不能用于负权边。

初始化起点到其他点的距离为正无穷，到源点的距离为0，循环n次，**每次找到未访问过的距离源点最近的点，并用该点更新起点到其他点的距离**。

朴素dijkstra时间复杂度O(n^2)

```c++
//以无重边，无自环为例
const int N = Max;
int n, m, d[N], f[N][N], x, y, z;
bool vis[N];

int dijkstra(void){
    memset(d, 127, sizeof(d));   //初始化
    d[1] = 0;
    for(int i = 1; i <= n; i++){
        int t = -1;  
        for(int j = 1; j <= n; j++) //找到未访问过的、距离源点最近的点t (可用堆优化)
            if(!vis[j] && (t == -1 || d[j] < d[t]))
                t = j;
       	vis[t] = true;
        for(int j = 1; j <= n; j++) //用点t更新源点到其他点的距离
            if(!vis[j] && f[t][j])
                d[j] = min(d[j], d[t] + f[t][j]);
    }
    return d[n] > 1e8 ? -1 : d[n];
}

int main()
{
    cin>>n>>m;
    while(m--){
        cin>>x>>y>>z;
        f[x][y] = z;
    }
    cout<<dijkstra()<<endl;
}
```

[最短路](http://oj.daimayuan.top/course/14/problem/655)



使用堆(优先队列)优化，时间复杂度降为O(mlogn)

```c++
const int N = 1e5 + 10;
typedef pair<int, int> PII;
int n, m, d[N], x, y, z;
bool vis[N];
struct Edge{
    int v, w;
};

vector<Edge> edge[N];  //连接表储存稀疏图

int dijkstra(void){
    memset(d, 127, sizeof(d));
    d[1] = 0;
    priority_queue<PII, vector<PII>, greater<PII>> heap;  //小根堆，储存距离源点最近的点(first储存距离，second储存该点，pair默认以first为排序标准)
    heap.push({0, 1});  //存入源点
    while(!heap.empty()){
        PII x = heap.top();
        heap.pop();
        int t = x.second;
        if(vis[t]) continue;   //访问过直接跳过
        vis[t] = true;
        for(auto i:edge[t])    //将未访问过的点添加到堆里，并更新最短距离
            if(!vis[i.v] && d[t] + i.w < d[i.v])
                d[i.v] = d[t] + i.w, heap.push({d[i.v], i.v});
    }
    return d[n] > 1e9 ? -1 : d[n];
}

int main()
{
    cin>>n>>m;
    while(m--){
        cin>>x>>y>>z;
        Edge e = {y, z};
        edge[x].push(e);
    }
    cout<<dijkstra();
}
```

[最短路2](http://oj.daimayuan.top/course/14/problem/662)



###### bellman-ford

循环n次，将所有可以松弛的点更新，注意每次更新用备份操作，防止串联更新

若源点到终点的途中存在负环，则无最短路。时间复杂度O(mn)

```c++
int n, m, x, y, z, d[N], backup[N];
struct Edge{
    int a, b, w;   //当前节点a，目标节点b，a到b的权重
}edge[M];

int bellman_ford(void){
	memset(d, 127, sizeof(d));
    d[1] = 0;
    for(int i = 0; i < n; i++){   //不超过n条边，到达终点所需要的最短距离
        memcpy(backup, d, sizeof(d));  //用备份更新d[b]，防止串联更新
        for(int j = 0; j < m; j++){
			int a = edge[i].a, b = edge[i].b, w = edge[i].w;
            d[b] = min(d[b], backup[a] + w);  //松弛操作
        }
    }
    return d[n];
}

int main()
{
    cin>>n>>m;
    for(int i = 0; i < m; i++){
        cin>>x>>y>>z;
        edge[i] = {x, y, z};
    }
    int res = bellman_ford();
    if(res > 1e9)
        cout<<"impossible";
   	else
        cout<<res;
}
```



###### SPFA

松弛操作中d[b] = min(d[b], d[a] + w) ，d[b]松弛前d[a]必定松弛，所以可以类似宽搜来优化bellman_ford，一般情况下时间复杂度优化成O(m)，极端情况下会退化成O(nm)

用spfa判断负权环：可用数组cnt[i]来表示到达i号点所经过的边。由于1~n个点，根据抽屉原理，两点之间不超过n-1条边，当cnt[i] == n时表明出现回路，出现回路即有负权边。

```c++
int n, m, x, y, z, d[N];
bool vis[N];   //vis[i]描述i号点是否已经在队列里
struct Edge{
    int v, w;
};
vector<Edge> edge[N];

void spfa(void){
    memset(d, 127, sizeof(d));
    d[1] = 0;
    queue<int> q;
    vis[1] = true;
    q.push(1);
    while(q.size()){
        int t = q.front();
        q.pop();
        vis[t] = false;   //t出队了，设置为false
        for(auto i:edge[t])   //遍历所有可以松弛的边
			if(d[t] + i.w < d[i.v]){
                d[i.v] = d[t] + i.w;
                if(!vis[i.v]){  //若不在队列里，则push进去
                    q.push(i.v);
                    vis[i.v] = true;
                }
			}
    }
}

int main()
{
    cin>>n>>m;
    while(m--){
        cin>>x>>y>>z;
        edge[x].push_back({y, z});
    }
    spfa();
}
```



### 最小生成树

###### 朴素prim

选取任意一点作为源点，然后找出距离集合最短的点，将其加入到集合，并用该点更新其他点到集合的最短距离。时间复杂度为**O(n^2)**，代码好写，适合稠密图

```c++
int n, m, a, b, c, f[N][N], d[N];  //d[i]表示i号点到集合(最小生成树)的最短距离
bool vis[N];

int prim(void) {
    memset(d, 127, sizeof(d));
    d[1] = 0;
    int res = 0;
    for (int i = 1; i <= n; i++) {   //循环n次，将n个点加入到集合
        int t = -1;
        for (int j = 1; j <= n; j++)  //找到距离集合最短的点
            if (!vis[j] && (t == -1 || d[j] < d[t])) t = j;
        if (d[t] > 1e9) return d[t];  //不连通，直接返回最大值
        res += d[t];
        vis[t] = true;  //标记点加入集合
        for (int j = 1; j <= n; j++)  //用该点更新其他点到集合的距离
            if (!vis[j]) d[j] = min(d[j], f[t][j]);
    }
    return res;
}

int main() {
    cin >> n >> m;
    memset(f, 127, sizeof(f));
    while (m--) {
		cin >> a >> b >> c;
        f[a][b] = f[b][a] = min(f[a][b], c);
    }
    int res = prim();
    if (res > 1e9)
        cout << "impossible";
    else
        cout << res;
}
```



###### kruskal

按边权从小到大遍历每一条边，如果a，b两点不在同一集合，则建边。并查集实现。时间复杂度为**O(mlog(m))** 

```c++
int n, m, a, b, c, f[N], res;
struct Edge {
    int a, b, c;
    bool operator < (const Edge &x) const {  //重载小于号，按边权排序
        return c < x.c;
    }
}edge[N];

int main() {
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        cin >> a >> b >> c;
        edge[i] = {a, b, c};
    }
    for (int i = 1; i <= n; i++) f[i] = i;  //并查集初始化
    sort(edge, edge+m);
    for (int i = 0; i < m; i++) {
        int fa = findset(edge[i].a);
        int fb = findset(edge[i].b);
        if (fa != fb) {
            f[fb] = fa;
            res += edge[i].c;
        }
    }
    int fx = findset(1);
    for (int i = 1; i <= n; i++)  //判断是否所有边都在同一集合 ---> 判断是否能构建最小生成树
        if (fx != findset(i)) {
            cout << "impossible";
            return 0;
        }
   	cout << res;
}
```



### 点（边）双边通分量

tarjan，割点、割边

### 二分图

###### 二分图判定(染色法)

一个图是二分图，当且仅当图中不含奇数环（二分图不一定是连通图，完全二分图为连通图）。二分图：将点分成两个集合，集合内部没有连边。时间复杂度**O(n+m)**

给每个点染色，确保相连两点染色不同。以下代码用dfs实现染色。

```c++
int n, m, a, b, cnt, color[N], f[N];  //color[i]记录染色情况，f[i]记录点
bool flag;  //标记染色时是否出现矛盾
vector<int> edge[N];

void dfs(int t, int k) {  //将点t染成k
	if (color[t] == 3-k) flag = true;  //出现染色冲突
    if (color[t] || flag) return;  //已经染色过了出现过冲突
    color[t] = k;
    for (auto i : edge[t]) dfs(i, 3-k);//将下一个点染成3-k (3-k实现1,2交替)
}

int main() {
	cin >> n >> m;
    for (int i = 1; i <= m; i++) {
		cin >> a >> b;
        edge[a].push_back(b);
        edge[b].push_back(a);
        f[cnt++] = a, f[cnt++] = b;
    }
    for (int i = 0; i < cnt; i++)
        if (!color[f[i]]) dfs(f[i], 1);
   	if (flag)
        cout << "No";
   	else
        cout << "Yes";
}
```

###### 匈牙利算法

### 欧拉回路

### 2-sat

### 竞赛图

### 差分约束



## 六、数学

### 数论

##### 1.埃氏筛

质数的倍数都是合数，通过倍数标记筛除合数，同一合数存在重复筛，时间复杂度为O(nlogn)

```c++
bool vis[N];

void solve(void) {
    f[1] = true;
    for (int i = 2; i <= n; i++)
        if (!vis[i]) {
            int cnt = i;
            while (cnt * i <= n) vis[cnt * i] = true, cnt++;
        }
}
```

##### 2.线性筛/欧拉筛

优化埃氏筛，每个合数都由最小的质因子筛除，时间复杂度O(nloglogn)

```c++
/*
若f[j]是i的倍数，则可以分解成(i * i) * (f[j]/i)，
那么可以直接退出了，因为i是逐个递增的，交由后面的i来筛除即可
比如i等于4时，若不退出，则筛除3 * 4 = 12，
而后续i等于6时，6 * 2 = 12则会重复筛除
*/

int f[N], cnt;
bool vis[N];

void solve(void) {
    vis[1] = true;
    for (int i = 2; i <= n; i++) {
        if (!vis[i]) f[cnt++] = i;   //f数组为当前筛选到的质数的集合
        for (int j = 0; j < cnt; j++) {
            if (i * f[j] > n) break; //超过筛选范围，没必要再筛了
            vis[i * f[j]] = true;
            if (i % f[j] == 0) break;
        }
    }
}
```



##### 3. gcd/lcm

gcd性质：

gcd(a, b) = gcd(b, a)

gcd(a, b) == gcd(a, b-a)   (a < b)

gcd(a1, a2, ......,an) == gcd(a1, gcd(a2,......,an))

```c++
int gcd(int a, int b){
    return a%b ? gcd(b, a%b) : b;
}

int lcm(int a, int b){
    return a * b / gcd(a, b);
}
```



##### 4.快速幂

```c++
//求a的b次幂，对mod取模
int qpow(int a, int b, int mod){
    int res = 1;
    while(b){
        if(b&1) res = res * a % mod;
        a = a * a % mod;
        b >>= 1;
	}
    return res;
}
```

```c++
/*
补充一条龟速乘，时间复杂度logn，
快速幂爆ll时结合龟速乘使用
*/
typedef long long ll;
ll qmul(ll a, ll b, ll p) {
    ll res = 0;
    while (b) {
		if (b & 1) res = (res + a) % p;
        a = (a + a) % p;
        b >>= 1;
    }
    return res;
}
```



##### 5.逆元

乘法逆元

数学上的乘法逆元就是指直观的倒数，如a的逆元就是1/a。 ax = 1, x就是a的逆元。
而关于取模运算的乘法逆元，就是满足ax mod b ≡ 1 (ax ≡ 1 mod b)时，称x为a关于模b的逆元，代码表示就是

a * x % b == 1。

```c++
/*求(a / b) % p;
设b的逆元为c，则：
	b * c ≡ 1 mod p
	(a / b) % p = (a * c) % p = ((a % mod) * (c % mod)) % mod
费马小定理：
	当b与p互质时：
	b^p ≡ b mod p
	b^(p-1) ≡ 1 mod p
令 c = b^(p-2)，则满足:
	b * c = b^(p-1) ≡ 1 mod p
	
	
所以b的逆元为b^(p-2) % p, 结合快速幂即可求出
*/
int solve(int b, int p){
    return qpow(b, p-2, p);
}
```



##### 6.扩展欧几里得

##### 7.费马定理/欧拉定理

##### 8.扩展欧拉定理

##### 9.同余方程组

扩展中国剩余定理

##### 10. Lucas定理

普通

##### 11. Miller Rubin素数检测法

不确定性算法

1.费马小定理：若a与p互质，则有a^(p-1)  ≡ 1 mod p
2.二次探测定理：0 < x < p，x^2 ≡ 1 mod p，若p为质数，则x的解只可能是1或-1

```c++
/*
x^2 - 1 ≡ 0 mod p
(x-1)(x+1) % p = 0
由于p为质数，不可能两个数都存在贡献价值，所以只能是(x-1)整除p，或者(x+1)整除p
（例如2 * 3 / 6可以整除，第一个数的贡献是2，第二个数的贡献是3，但只在合数意义下存在）
要求0 < x < p，则x的解为1或者-1(即 mod p 意义下的 p-1)
*/
```

费马探测法：

若对于任意a，都有 a^(p-1)  ≡ 1 mod p，则假定p是素数的可能性很大，而探测时，并不会真的遍历所有的a，而是取少数a探测，若对于这些a，若模数都为1，则p不是素数的概率很小很小，但仍存在一些合数能通过费马检测，我们将这些数称为**伪素数**，例如Carmichael数（卡迈尔数）。

而Miller Rubin通过二次探测，优化费马探测法，又筛除了一些伪素数，选取合适的参数，在一定范围内可以当做是准确性算法。

二次探测：

对于探求的p，若p为偶数，特判一下2，其他直接排除即可；而当p为奇数，则p-1为偶数，将p-1化成2m的形式，则 a ^ (p-1) ≡ 1 mod p 可以转化成 (a ^ m) ^ 2 ≡ 1 mod p，由二次探测定理可知，a ^ m % p的解只有1或-1 （即p-1），探测是否符合即可。（很多博客写的算法利用数字特性，为了“榨干”剩余价值，当m还是偶数时又继续探测，其实没有必要）

```c++
/*
以判断int范围内是否为素数为例，
网络上关于参数的数据并没有统一，
这里选取前几个质数作为参数，int范围内算是完全正确了
如果是long long范围内，除了考虑参数外，还需考虑乘法溢出，可用快速乘解决
*/

/*
如果你每次都用前7个素数(2, 3, 5, 7, 11, 13和17)进行测试，所有不超过341 550 071 728 320(即3.4e14)的数都是正确的。如果选用2, 3, 7, 61和24251作为底数，那么10^16内唯一的强伪素数为46 856 248 255 981(即4.6e13)。
原文链接：https://blog.csdn.net/a1307754356/article/details/97821076
*/

typedef long long ll;
ll qpow(ll a, ll b, ll p) {
    ll res = 1;
    while (b) {
        if (b & 1) res = res * a % p;
        a = a * a % p;
        b >>= 1;
    }
    return res;
}

bool Miller_Rubin(int n) {
    if (n == 2 || n == 3 || n == 5) return true;
    if (n % 2 == 0 || n % 3 == 0 || n % 5 == 0) return false;  //2，3，5可以筛除大批合数，小小优化
    int A[] = {2, 3, 5, 7, 11, 13, 17, 19, 23};
    for (auto a : A) {
        if (a == n) return true;
        ll x = qpow(a, n-1, n);   //进行费马检验
        if (x != 1) return false;
        x = qpow(a, (n-1)/2, n);  //通过费马检验后，进行二次探测
        if (x != 1 && x != n-1) return false;
    }
    return true;  //经过一组探测后，都符合要求，则大概率是素数，返回true
}
```



### 数论函数



##### 1.整除分块

##### 2.狄利科雷卷积

##### 3.莫比乌斯反演

### 线性代数

矩阵快速幂，高斯消元，行列式，线性基

### 组合数学

##### 1.组合数

杨辉三角，多重集组合数

##### 2.二项式定理

```c++
/*求Cm|n = m!  / ( n! * (m-n)!), mod (mod为质数)
结合乘法逆元求解：
*/
typedef long long LL;
LL N(LL n, LL mod){
    LL res = 1;
    for(int i = 1; i <= n; i++)
         res = res * i % mod;
   	return res;
}
LL solve(LL n, LL m, LL mod){
    LL t1 = N(m);
    LL t2 = N(n) * N(m-n) % mod;
    LL res = 1, p = mod - 2;
    while(p){
        if(p&1) res = res * t2 % mod;
        t2 = t2 * t2 % mod;
        p >>= 1;
	}
    return t1 * t2 % mod;
}
```

[完美数 ](http://oj.daimayuan.top/course/11/problem/669)



##### 3.概率论

期望的线性性

##### 4.容斥原理

基础

##### 5.常见数列

斐波那契数列，错排问题，卡特兰数，拆分数，斯特林数

##### 6.博弈论

SG函数，常见结论

##### 7.多项式

FFT/NTT

##### 8.群论

置换



## 七、几何

### 点类

点积，叉积，极角序

### 线段相关

相交判断，交点

### 多边形相关

凸包

### 圆相关

交点切线



## 八、杂

##### 离散

##### 前缀和/差分

###### 一维前缀和

快速求解某区间的元素和

```c++
//先预处理求以1起始的前缀和 (下标从1开始就不用担心溢出问题啦~)
for(int i = 1; i <= n; i++){
    scanf("%d",&a[i]);
    sum[i] = sum[i-1]+a[i];
}
int solve(int l, int r){
	return sum[r] - sum[l-1];
}
```

[前缀和](http://oj.daimayuan.top/problem/58)， [简单子段和](http://oj.daimayuan.top/problem/868)



###### 二维前缀和

计算矩阵内元素的和

sum i j 表示 由顶点坐标(0, 0)与(i, j)所构矩阵的元素和

利用左上顶点和右下顶点求子矩阵元素和

注意减去重叠部分以及边界

```c++
for(int i = 1; i <= n; i++)
    for(int j = 1; j <= m; j++){
        cin>>a[i][j];
        sum[i][j] = a[i][j] + sum[i-1][j]+sum[i][j-1]-sum[i-1][j-1];
    }
//求(x1, y1), (x2, y2)所构矩阵的元素和
int solve(int x1, int y1, int x2, int y2){
    if(x1 > x2) swap(x1, x2);
    if(y1 > y2) swap(y1, y2);
    return sum[x2][y2]-sum[x1-1][y2]-sum[x2][y1-1] + sum[x1-1][y1-1];
}
```

[二维前缀和](http://oj.daimayuan.top/course/22/problem/894)



###### 差分

对某一区间内所有元素进行相同增量操作，差分后求前缀和即为操作后的数组。时间复杂度O(n)

```c++
//先预处理求差值
for(int i = 1; i <= n; i++){
    scanf("%d",&a[i]);
    b[i] = a[i]-a[i-1];
}
void solve(int l, int r, int val){
    b[l] += val;
    b[r+1] -= val;
    long long ans;
    for(int i = 1; i <= n; i++)
        ans += b[i], printf("%lld ",ans);
}
```

[差分练习](http://oj.daimayuan.top/problem/285)



###### 二维差分

逐行进行差分即可，时间复杂度O(n^2)

```c++
void solve(int x1, int x2, int y1, int y2) {
    for (int i = x1; i <= x2; i++)
        f[i][y1]++, f[i][y2+1]--;
    for (int i = 1; i <= n; i++, cout << endl) {
        int sum = 0;
        for (int j = 1; j <= n; j++)
            sum += f[i][j], cout << sum << " ";
    }
}
```



##### 逆序对

a[i] > a[j] && i < j，则称a[i]与a[j]为一个逆序对；一个有序排列的逆序对总数，即逆序数。

```c++
/*
借助归并排序，时间复杂度O(nlogn)；
从小到大、从大到小均可，此处采用的从大到小排序；start到mid， mid+1到end均为从大到小排列。
用i，j分别指向两个部分起始端，比较a[i]与a[j],(i始终小于j,只需判断a[i]与a[j]的关系即可判断逆序对)
若a[i] <= a[j]，则a[i]与a[j]无法组成逆序对，j++;
若a[i] >  a[j]，则a[j]以及a[j]后面的数都比a[i]小，逆序对数 += r-j+1,同时i++，判断下一个a[i]。
求完两部分逆序对数，并成一部分，归到递归上一层，与同层的另一部分求逆序数。
*/
int a[Max], b[Max];
long long ans;

void solve(int start, int end){
    if(start >= end) return;
    int mid = start+end >> 1;
    sort(start, mid);
    sort(mid+1, end);
    int i = start, j = mid+1;
    while(i <= mid && j <= r)
        if(a[i] <= a[j]) j++;
    	else i++, ans += r-j+1;
   	merge(start, mid, end);
}

void merge(int start, int mid, int end){
    int i = start, j = mid+1, p = start;
    while(i <= mid && j <= end)
        if(a[i] > a[j]) b[p++] = a[i++];
    	else b[p++] = a[j++];
    while(i <= mid) b[p++] = a[i++];
    while(j <= end) b[p++] = a[j++];
    for(i = start; i <= end; i++) a[i] = b[i];
}
```

```c++
/*
树状数组 + 离散化实现，时间复杂度O(nlogn)
数据经过离散化后，用树状数组维护对于当前遍历到的前i个元素，小于a[i]的个数。
当前有i个数，若逆序数为0，则小于x的个数应为i-1；作差即可得到逆序数
*/
int lowbit(int x) {
    return x & (-x);
}

void add(int i, int x) {
    while (i <= n) {
        f[i] += x;
        i += lowbit(i);
    }
}

int ask(int i) {
    int res = 0;
    while (i) {
        res += f[i];
        i -= lowbit(i);
    }
    return res;
}

void solve(void) {
    int res = 0;
    for (int i = 1; i <= n; i++) {
        add(a[i], 1);  //维护当前a[i]出现的次数
        int cnt = i - ask(a[i] - 1);
        res += cnt;
    }
    return res;
}
```

[逆序对2](http://oj.daimayuan.top/course/22/problem/653)

##### 扫描线

##### 双指针（尺取法）

特定情况下可以将暴力的时间复杂度从O(n^2)优化成O(n)

前后指针

快慢指针

[统计子矩阵](http://oj.daimayuan.top/course/18/problem/752)



##### 倍增

##### 三分

求单峰极值：将答案区间分成三等份，确保答案在第二份区间内，再不断更新m1,m2，缩短区间范围。
注意：
1.确保一开始答案就在第二区间内，否则越缩越错。
2.更新秉持"更新最差，保留最优"
3.函数图像单峰且连续，且不能出现平滑部分

```c++
//例：求单峰极小值,f(x)为该函数的求值方式
double solve(void){
    double l = -1e9, r = 1e9, m1, m2;
    for(int i = 0; i < 100; i++){
        m1 = (r-l)/3 + l;
        m2 = (r-l)/3*2 + l;
        if(f(m1) < f(m2)) r = m2;
        else              l = m1;
	}
    return f(l);
}
```

[二次函数](http://oj.daimayuan.top/course/16/problem/651)



##### 枚举子集、超集

##### 搜索

[BFS练习0](http://oj.daimayuan.top/problem/146)

[BFS练习1](http://oj.daimayuan.top/problem/147)

[BFS练习1.5](http://oj.daimayuan.top/problem/150)

[BFS练习2](http://oj.daimayuan.top/problem/148)

[BFS练习3](http://oj.daimayuan.top/problem/149)

剪枝，折半搜索



##### *随机

爬山，模拟退火

##### *对拍

##### *常数优化

##### *读入优化

