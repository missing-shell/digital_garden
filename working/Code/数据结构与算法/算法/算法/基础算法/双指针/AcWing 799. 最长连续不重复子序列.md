* 仔细理解题意，相邻的数相同只是一种特殊情况，
* n=10
* 9 3 6 9 5 10 1 2 3 9这种同样需要考虑在内
* 数组s记录子序列a[j ~ i]中各元素出现次数

 ```c++
#include<iostream>
#include<algorithm>
#include<cstring>

using namespace std;

const int N = 1e5+10;

int a[N],s[N];

int main()
{
	int n,k=0;
	cin >> n;
	for (int i = 0; i < n; i++) cin >> a[i];

	for (int i = 0,j=0; i < n; i++)
	{
		s[a[i]]++;
		while (j<i&&s[a[i]]>1) {
			 
			s[a[j]]--;             //** s[a[j]]先减一，再右移j,确保j刚好移到位置i（如上例）;
			j++;                   //简化s[a[j++]]--;
		}
		k = max(k, i - j + 1);

	}
	cout << k << endl;
	return 0;
}
```