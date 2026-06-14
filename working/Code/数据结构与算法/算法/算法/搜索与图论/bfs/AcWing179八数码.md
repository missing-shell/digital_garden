- 同[[AcWing845八位码]]

- 将845的计数更改为使用字符串表示

```c++
#include <iostream>
#include <string>
#include <queue>
#include <unordered_map>

using namespace std;

const int dx[] = {-1, 1, 0, 0}, dy[] = {0, 0, -1, 1};
bool st[3][3];

string res;

string check(int i)
{
    string op;

    if (0 == i)
    {
        op = 'u';
    }
    else if (1 == i)
    {
        op = 'd';
    }
    else if (2 == i)
    {
        op = 'l';
    }
    else if (3 == i)
    {
        op = 'r';
    }

    return op;
}
string bfs(string str)
{
    queue<string> ans;
    unordered_map<string,string> dist;

    ans.push(str);
    //dist[str] = "";

    while (ans.size())
    {
        auto t = ans.front();
        ans.pop();

        string distance = dist[t];

        if (t == "12345678x")
        {
            return distance;
        }
        int k = t.find('x');
        int x = k / 3, y = k % 3;

        for (int i = 0; i < 4; i++)
        {
            int a = x + dx[i], b = y + dy[i];
            if (a >= 0 && a < 3 && b >= 0 && b < 3)
            {
                swap(t[k], t[a * 3 + b]);
                if (!dist.count(t))
                {
                    dist[t] = distance +check(i);
                    ans.push(t);
                }
                swap(t[k], t[a * 3 + b]);
            }
        }
    }
    return "unsolvable";
}
int main()
{
    string c, str;
    for (int i = 0; i < 9; i++)
    {
        cin >> c;
        str += c;
    }

    cout << bfs(str) << endl;

    return 0;
}
```