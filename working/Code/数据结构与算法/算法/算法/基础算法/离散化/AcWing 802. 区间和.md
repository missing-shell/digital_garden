### 代码
 ```c++
 #include<iostream>
#include<algorithm>
#include<vector>

using namespace std;
typedef pair<int, int>PII;
const int N = 3e5 + 10;//n次插入和m次查询的上界

int n, m;
int a[N];//存储坐标插入的值
int s[N];//数组a的前缀和
vector<int>alls;//存储所有插入和查询的坐标
vector<PII>add, query;//存储插入和查询的原始数据

int find(int x)//返回相应坐标对应离散化后的坐标 
{
	int l = 0, r = alls.size() - 1;
	while (l < r)
	{
		int mid = l + r >> 1;
		if (alls[mid] >= x)r = mid;
		else l = mid + 1;
	}
	return r + 1;
}

int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++)
	{
		int x, c;
		cin >> x >> c;
		add.push_back({ x,c });
		alls.push_back(x);//将插入所用坐标放在alls中
	}
	for (int i = 1; i <= m; i++)
	{
		int l, r;
		cin >> l >> r;
		query.push_back({ l,r });
		alls.push_back(l);//查询的坐标
		alls.push_back(r);
	}
	sort(alls.begin(), alls.end());//排序
	alls.erase(unique(alls.begin(), alls.end()), alls.end());//清除重复元素
	/*unique返回的无重复数字的新数组的末尾*/
	for (auto item : add)
	{
		int x = find(item.first);
		a[x] += item.second;
	}
	for (int i = 1; i <= alls.size(); i++)
	{
		s[i] = s[i - 1] + a[i];
	}
	for (auto item:query)
	{
		int l = find(item.first);
		int r = find(item.second);
		printf("%d\n", s[r] - s[l - 1]);
	}
	return 0;
}
```

### unique函数的实现方式
```c++
vector<int>::iterator uniqure(vector<int>& a)
{
	int j = 0;
	for (int i = 0; i < a.size(); i++)
	{
		if (!i || a[i] != a[i - 1]) {
			a[j++] = a[i];
		}
	}
	//a[0]~a[j-1]是所有a中不重复的数
	return a.begin() + j;
}


alls.erase(unique(alls), alls.end());//删除重复元素

```


[参考](https://www.acwing.com/solution/content/13511/)
![8021.png](https://cdn.acwing.com/media/article/image/2020/05/22/38626_05e3618e9b-8021.png)

![8022.png](https://cdn.acwing.com/media/article/image/2020/05/22/38626_08fb69ca9b-8022.png)
