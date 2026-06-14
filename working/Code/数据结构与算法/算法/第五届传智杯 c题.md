## 未通过

```c++
#include<iostream>
#include<algorithm>

using namespace std;

const int N = 1e6 + 10;
long long   n,w,sum=0;
long long int ans[N];

int main()
{
	int leap = 0;
	scanf_s("%lld", &n);
	for (long long int i = 0; i < n; i++) {
		scanf_s("%lld", &ans[i]);
		if (ans[i]) {
			leap = 1;
		}
	}
	sort(ans, ans + n);
	cin >> w;
	if (leap) {
		for (int i = n - 1; i >= 0; i--) {
			if (w >= ans[i]) { 
				w = ans[i];
				n = i;//此处出错！！！！！！！！！！！！！！
				break;
			}
		}
		long long int i = 0;
		while (i < n) {
			w -= ans[i];
			if (w >= 0) {
				sum++;
			}
			else
				break;
			i++;
		}
	}
	else sum = 0;
	cout << sum << endl;
}
```
### 边界问题
#### 更正
```c++
		for (int i = n - 1; i >= 0; i--) {
			if (w >= ans[i]) { 
				w = ans[i];
				n = i+1;// 更正，i先减一，所以第一次n就等于n-2,后续需要加1。其实这道题课以不用加这步条件，在累减操作中，w并不可能越界，
				break;
			}
		}
```
##### 优化
```c++
#include<iostream>
#include<algorithm>
using namespace std;

const int N = 1e6 + 10;
long long  int w,sum ;
int ans[N],n;

int main()
{
	cin >> n;
	for (int i = 0; i < n; i++) {
		scanf("%lld", &ans[i]);
	}

	sort(ans, ans + n);
	cin >> w;
	
	for (int i = n - 1; i >= 0; i--) {
		if (w >= ans[i]) {
			w = ans[i];
			break;
		}
	}
	for(int i=0;i<n;i++){
		w -= ans[i];
		if (w >= 0) {
			sum++;
		}
		else {
			cout << sum << endl;
			break;
		}
			 
	}
	return 0;
}
```