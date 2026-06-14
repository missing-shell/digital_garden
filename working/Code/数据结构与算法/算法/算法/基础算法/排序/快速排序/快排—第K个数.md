-  重点在于对K的判断，优化时间
```c++
#include<iostream>
#include<stdio.h>
#include<algorithm>
using namespace std;
const int M = 5000010;
int n,k; 
int ans[M];
void quick_sort(int L, int R)
{
	if (L == R) return;
	int i = L-1, j = R+1, x = ans[(L + R)/2];
	while (i < j)
	{
		while (ans[++i] < x);
		while (ans[--j] > x);
		if(i<j)swap(ans[i], ans[j]);
	}
	if (k <= i) {
		quick_sort(L, j);
	}
	else{ 
		quick_sort(j + 1, R); 
	}
	 
}
int main()
{   
	scanf("%d %d", &n, &k);
	for (int i = 0; i < n; i++) {
		scanf("%d", &ans[i]);
	}
	quick_sort(0, n - 1);
	printf("%d\n", ans[k]);
	return 0;
}

```


