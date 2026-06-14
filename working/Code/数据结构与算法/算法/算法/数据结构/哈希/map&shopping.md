```c++

#include<string>
#include<map>
using namespace std;

int main()
{
	int n;
	while (scanf("%d", &n) != EOF) {
		string s;
		for (int i = 0; i < n; i++)cin >> s;
		int times;
		scanf("%d", &times);
		map<string, int>shop;
		int x;
		string name;
		while (times--) {
			for (int i = 0; i < n; i++) {
				cin >> x >> name;
				shop[name] += x;
			}
			int ans = 1;
			map<string, int>::iterator it;
			for (it = shop.begin(); it != shop.end(); it++) {
				if (it->second > shop["memory"]) {
					ans++;
				}
			}
			printf("%d\n", ans);
		}
	}
	return 0;


}