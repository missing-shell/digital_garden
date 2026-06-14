- 参考[四谷夕雨](https://www.acwing.com/solution/content/15149/)

- 拓展[[AcWing179八数码]]

- 思想：将每一种情况作为1个节点，目标情况即为终点
	
	从初始状况移动到目标情况 —> 求最短路

- 问题
	
	怎么表示一种情况使其能作为节点？
	如何记录每一个状态的“距离”（即需要移动的次数）？
	队列怎么定义，dist数组怎么定义？

- **解决方案**
	
	将`3*3`矩阵转换为字符串
	队列使用`queue<string>`存储转换后的字符串
	`dist`数组用`unordered_map<string,int>`将字符串和数字联系在一起，字符串表示状态，数组表示距离

- 矩阵与字符串的转换关系

	字符串下标`i=x*+y`
	矩阵`x=i/3,y=i%3`

### `bfs`
```c++
#include <iostream>
#include <algorithm>
#include <string>
#include <queue>
#include <unordered_map>

using namespace std;

const int dx[] = {-1, 1, 0, 0}, dy[] = {0, 0, -1, 1};
int bfs(string str)
{
    string end = "12345678x";//定义目标状态
    queue<string> q;//队列
    unordered_map<string, int> dist;//dist数组
    q.push(str);
    dist[str] = 0;
    while (q.size())
    {
        auto t = q.front();
        q.pop();
        int distance = dist[t];//记录当前状态，如果为目标状态则返回距离
        if (t == end)
        {
            return distance;
        }
        int k = t.find('x');
        int x = k / 3, y = k % 3;//将x在字符串中的下标转换为矩阵坐标

        for (int i = 0; i < 4; i++)
        {
            int a = x + dx[i], b = y + dy[i];
            if (a >= 0 && a < 3 && b >= 0 && b < 3)
            {
                swap(t[k], t[a * 3 + b]);
                if(!dist.count(t))//如果当前状态是第一次遍历，记录距离，入队
                {
                    dist[t] = distance + 1;
                    q.push(t);
                }
                swap(t[k], t[a * 3 + b]);//还原状态，为下一次转换情况做准备
            }
        }
    }
    return -1;
}
int main()
{
    string c, str;
    for (int i = 0; i < 9; i++)
    {
        cin >> c;//cin滤除空格
        str += c;
    }

    cout << bfs(str) << endl;
    return 0;
}
```
