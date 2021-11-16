# 函数实现

```c++
void *my_memcpy_byte(void *dst, const void *src, int n)
{
	if (dst == NULL || src == NULL || n <= 0)
		return NULL;

	char * pdst = (char *)dst;
	char * psrc = (char *)src;

	if (pdst > psrc && pdst < psrc + n)//有内存覆盖
	{
		pdst = pdst + n - 1;
		psrc = psrc + n - 1;
		while (n--)
			*pdst-- = *psrc--;
	}
	else
	{
		while (n--)
			*pdst++ = *psrc++;
	}
	return dst;
}
```

# 类

## 静态变量、常量、指针的构造、析构、拷贝构造、移动构造的初始化

```C++
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;

class A {
public:
	A(int y_,char* p_) :y(y_){
		cout << "constructor" << endl;
		int len = strlen(p_);
		p = new char[len + 1];
		memset(p, len + 1, 0);
		strcpy(p, p_);
	}
	~A() {
		delete[] p;
	}
	A(const A& a2):y(a2.y) {
		cout << "copy constructor" << endl;
		int len = strlen(a2.p);
		p = new char[len + 1];
		memset(p, len + 1, 0);
		strcpy(p, a2.p);
	}

	A(A&& a2) :y(a2.y), p(a2.p) {
		cout << "move constructor" << endl;
		a2.p = nullptr;
	}
private:
	static int x;
	const int y;
	char* p;
};
int A::x = 10;

int main()
{
	char *p_ = "abc";
	A a(1, p_);
	A b(a);
	A c(move(a));
}
```



# 字符串

## 构建回文

```c++
// 部分测试用例没过
#include <string>
#include <iostream>
using namespace std;
bool isHW(char* str, int start, int end){
    for (int i=start,j=end; i<j; i++,j--) {
        if(str[i]!=str[j])
            return false;
    }
    return true;
}
int main() {
    char str[2001];
    cin>>str;
    int count=0;
    for (int i=0; str[i]!='\0'; i++) {
        count++;
    }
    int idx=count-1;
    for (int i=0; i<count-1; i++) {
        if(str[i]==str[count-1] && isHW(str, i, count-1)){
            idx=i-1;
            break;
        }
    }
    // 将[0,idx]添加到末尾
    for (int j=idx; j>=0; j--) {
        str[count++]=str[j];
    }
    str[count]='\0';
    cout<<str<<endl;
    return 0;
}
```

string反转字符串的做法

```c++
链接：https://www.nowcoder.com/questionTerminal/8eaa64ad417f4deeb25fbd350f2273fb
来源：牛客网

#include<bits/stdc++.h>
using namespace std;

const int MAXL = 3000;
int main()
{
    string s;
    cin>>s;
    int l = s.size();
    for (int i = 0; i < l; i++)
    {
        string temp = s;
        reverse(temp.begin(), temp.end());
        if (s == temp) break;
        s.insert(s.begin()+l, s[i]);
    }
    cout<<s;
    return 0;
}
```



# 排序

## 快速排序

```c++
void quickSort(vector<int>& arr, int l, int r) {
    // 子数组长度为 1 时终止递归
    if (l >= r) return;
    // 哨兵划分操作（以 arr[l] 作为基准数）
    int pivot=arr[l];
    int i = l, j = r;
    while (i < j) {
        while (i < j && arr[j] >= pivot) j--;
        while (i < j && arr[i] <= pivot) i++;
        swap(arr[i], arr[j]);
    }
    swap(arr[i], pivot);
    // 递归左（右）子数组执行哨兵划分
    quickSort(arr, l, i - 1);
    quickSort(arr, i + 1, r);
}
```

## 归并排序

- 思想：分治法，将数组分成子数组，对两份子数组合并成有序的数组

```c++
void mergeSortCore(int a[], int b[], int start, int end)
{
	if (start >= end)
		return;
	// 继续拆分子数组
	int mid = (start + end) / 2;
	int start1 = start, end1 = mid;
	int start2 = mid + 1, end2 = end;
	mergeSortCore(a, b, start1, end1);
	mergeSortCore(a, b, start2, end2);

	// 合并子数组
	int k = start;
	while (start1 <= end1 && start2 <= end2)
	{
		b[k++] = a[start1] < a[start2] ? a[start1++] : a[start2++];
	}
	// 把剩下的元素补齐
	while (start1 <= end1)
		b[k++] = a[start1++];
	while (start2 <= end2)
		b[k++] = a[start2++];
	// 更新原始数组
	for (k = start; k <= end; k++)
		a[k] = b[k];
}
void mergeSort(int a[], const int n)
{
	int *b = new int[n];
	mergeSortCore(a, b, 0, n-1);
	delete [] b;
}
```

## 冒泡排序

```c++
void bubbleSort(int a[], int len)
{
	for (int i = 0; i < len - 1;i++)
	{
		for (int j = 0; j < len - i - 1;j++)
		{
			if (a[j] > a[j + 1])
			{
				int temp = a[j];
				a[j] = a[j + 1];
				a[j + 1] = temp;
			}
		}
	}
}
```

## 选择排序

```c++
void selectSort(int[] arr, int len){
    for(int i = 0; i < len - 1; i++){
        int min = i;
        for(int j = i + 1; j < len; j ++){
            if(arr[min] > arr[j]){
                min = j;
            }
        }
        swap(arr, i, min);
    }
}
```



## 堆排序

思路

- 建立大根堆
- 交换堆顶与堆尾元素（堆尾元素为最大元素）
- 重新调整成大根堆

```c++
// 递归方式构建大根堆(len是arr的长度，index是第一个非叶子节点的下标)
void adjust(vector<int> &arr, int len, int index)
{
	int left = 2 * index + 1; // index的左子节点
	int right = 2 * index + 2;// index的右子节点

	int maxIdx = index;
	if (left<len && arr[left] > arr[maxIdx])     maxIdx = left;
	if (right<len && arr[right] > arr[maxIdx])     maxIdx = right;

	if (maxIdx != index)
	{
		swap(arr[maxIdx], arr[index]);
		adjust(arr, len, maxIdx);
	}

}

// 堆排序
void heapSort(vector<int> &arr, int size)
{
	// 构建大根堆（从最后一个非叶子节点向上）
	for (int i = size / 2 - 1; i >= 0; i--)
	{
		adjust(arr, size, i);
	}

	// 调整大根堆
	for (int i = size - 1; i >= 1; i--)
	{
		swap(arr[0], arr[i]);           // 将当前最大的放置到数组末尾
		adjust(arr, i, 0);              // 将未完成排序的部分继续进行堆排序
	}
}
```

参考：

https://www.cnblogs.com/wanglei5205/p/8733524.html



## 插入排序

```c++
void InsertSort(vector<int>& a, int len) {
	for (int i = 1; i < len; i++)
	{
		int key = a[i];
		int j = i - 1;
		while (j>=0 && key<a[j])
		{
			a[j + 1] = a[j];
			j--;
		}
		a[j + 1] = key;
	}
}
```

# 容器

## unordered_map

字典，unordered_map不会根据key的大小进行排序，存储时是根据key的hash值判断元素是否相同

```c++
#include<string>  
#include<iostream>  
#include<unordered_map>
using namespace std;  
  
int main()
{
	unordered_map<string, int> dict; // 声明unordered_map对象
	//unordered_map<string, int> dict = {{"apple",2}, "orange",3}; // 带初值
    
	// 插入数据的四种方式
	dict.insert(pair<string,int>("apple",2));
	dict.insert(make_pair("orange",3));
    dict.insert({ "tomato", "1" });
	dict["banana"] = 6;
	
	// 判断是否有元素
	if(dict.empty())
		cout<<"该字典无元素"<<endl;
	else
		cout<<"该字典共有"<<dict.size()<<"个元素"<<endl;
	
	// 遍历
	unordered_map<string, int>::iterator iter;
	for(iter=dict.begin();iter!=dict.end();iter++)
		cout<<iter->first<<" "<<iter->second<<endl;
	
	// 查找
	if(dict.count("boluo")==0)
		cout<<"can't find boluo!"<<endl;
	else
		cout<<"find boluo!"<<endl;
	
	if((iter=dict.find("banana"))!=dict.end())
		cout<<"banana="<<iter->second<<endl;
	else
		cout<<"can't find boluo!"<<endl;
	
    // 通过迭代器删除
	mymap5.erase(mymap5.begin());
	// 通过 Key 值删除
	mymap5.erase("France");
	// 通过迭代器范围删除
	mymap5.erase(mymap5.find("China"), mymap5.end());

	return 0;
}
```

- `map`和`unordered_map`区别

  - map：对应红黑树（具有自动排序功能），查找时间复杂度**O(logN)**

  - unordered_map：对应哈希表，查找时间复杂度**O(1)**

- `hash_map`和`unordered_map`区别
  - 本质都是哈希表，只不过 `unordered_map`被纳入了C++标准库标准。

## priority_queue

大根堆、小根堆

```c++
#include <functional>
priority_queue<int> q // 默认大根堆
priority_queue<int, vector<int>, greater<int>> q // 小根堆
```



注意：

- emplace_back(type) 对应 push_back(type)
- emplace(i, type) 对应于 insert(type, i)
- emplace_front(type) 对应于 push_front()

但是！对于stack 和 queue，只有push操作，所以也只有emplace操作，此时它们是相对应的。



## 手写最大堆

```c++
//
//  main.cpp
//  MyHeap
//
//  Created by jinye on 2021/7/7.
//

#include <iostream>
using namespace std;
class MaxHeap{
public:
    //数组索引为0的位置不放元素
    MaxHeap(int maxSize){
        data = new int[maxSize+1];
        size = 0;
        capacity = maxSize;
    }
    
    
    void push(int d){
        if(size == capacity){
            std::cout<<"堆已满！"<<std::endl;
            return;
        }
        //索引为0的位置不存放元素
        data[size+1] = d;
        size++;
        //插入在最后的元素上移方法
        shiftUp(size);
    }
    //堆插入元素时的元素上移
    void shiftUp(int i){
        //数组可能越界问题始终不能忽视
        //当此元素比父元素大时，交换这两个元素位置
        while(i > 1 && data[i] > data[i/2]){
            int t = data[i];
            data[i] = data[i/2];
            data[i/2] = t;
            i /= 2;
        }
    }
    
    //删除堆的最大元素
    int deleteMax(){
        if(size == 0){
            std::cout<<"堆已经是空的了！"<<std::endl;
            return -1;
        }
        int t = data[1];
        //将最后一个元素放到第一个元素位置
        data[1] = data[size];
        size--;
        //然后将第一个元素下移到适当位置
        shiftDown(1);
        return t;
    }

    //堆删除元素时的元素下移
    void shiftDown(int i) {
        while(2*i <= size){
            // 将要将data[i]与data[j]交换
            int j = 2*i;
            // 让j指向他的孩子结点中的大的那一个
            if(j+1 <= size && data[j] < data[j+1]){
                j += 1;
            }
            if(data[i] > data[j])
                break;
            //元素下移
            int t = data[i];
            data[i] = data[j];
            data[j] = t;
            i = j;
        }
    }
    
    MaxHeap(int arr[], int length, int maxSize){
        data = new int[maxSize+1];
        capacity = maxSize;
        for(int i = 0; i < length; i++){
            data[i+1] = arr[i];
        }
        size = length;
        for(int i = size; i >= 1; i--){
            shiftDown(i);
        }
    }
    
    
    
private:
    int* data;
    int size;       //当前堆的大小
    int capacity;   //堆的最大容量
};
void heapSort(int arr[], int length){
    MaxHeap maxHeap(arr, length, 100);
    for(int i=length-1; i >= 0 ; i--){
        arr[i] = maxHeap.deleteMax();
    }
}
int main() {
    // insert code here...
//    MaxHeap mh(10);
//    mh.push(1);
//    mh.push(2);
//    mh.push(3);
//    mh.push(4);
//    int max=mh.deleteMax();
    int arr[4]={4,2,1,3};
    for(auto n:arr)
        cout<<n<<"\t";
    cout<<endl;
    heapSort(arr, 4);
    for(auto n:arr)
        cout<<n<<"\t";
    cout<<endl;
    return 0;
}
```



# 并查集

```c++
class UnionFind {
public:
	UnionFind(int _n) :sets(_n), n(_n), rank(_n), parent(_n) {
		for (int i = 0; i < n; i++)
			parent[i] = i;
	}
	void merge(int x, int y) {
		int rootX = find(x);
		int rootY = find(y);
		if (rootX != rootY) {
			if (rank[rootX] > rank[rootY]) {
				parent[rootY] = rootX;
			}
			else if (rank[rootX] < rank[rootY]) {
				parent[rootX] = rootY;
			}
			else {
				parent[rootY] = rootX;
				rank[rootX]++;
			}
			sets--;
		}
	}
	int find(int x) {
		if (x == parent[x])
			return x;
		return parent[x] = find(parent[x]);
	}

	int getSets() {
		return sets;
	}
private:
	int sets, n; // 集合数量, 点数量
	vector<int> parent, rank;
};

class Solution {
public:
	int findCircleNum(vector<vector<int>>& isConnected) {
		int n = isConnected.size();
		UnionFind uf(n);
		for (int i = 0; i < n; i++)
			for (int j = 0; j < i; j++)
				if (isConnected[i][j])
					uf.merge(i, j);
		return uf.getSets();
	}
};
```



# 回溯法

## 思想

- 画出递归树，找到状态变量（回溯函数的参数）
- 根据题意，确立结束条件
- 找准选择列表（与函数参数相关）
- 判断是否需要剪枝
- 作出选择，递归调用，进入下一层
- 撤销选择

链接：https://leetcode-cn.com/problems/subsets/solution/c-zong-jie-liao-hui-su-wen-ti-lei-xing-dai-ni-gao-/

## 子集组合

### 子集（元素不重复）

```c++
/**
 * Leetcode_78
 * 给你一个整数数组 nums ，数组中的元素 互不相同 。返回该数组所有可能的子集（幂集）
 */
class Solution {
public:
	vector<vector<int>> subsets(vector<int>& nums) {
		vector<vector<int>> res;
		vector<int> cur;
		backtrack(nums, res, cur, 0);
		return res;
	}
	
	void backtrack(vector<int>& nums, vector<vector<int>>& res, vector<int> cur, int start)
	{
		res.push_back(cur);
		for (int i = start; i < nums.size(); i++)
		{
			cur.push_back(nums[i]);
			backtrack(nums, res, cur, i + 1);
			cur.pop_back();
		}
	}
};
```

### 子集（元素重复）

```c++
/**
 * Leetcode_90
 * 给你一个整数数组 nums ，其中可能包含重复元素，请你返回该数组所有可能的子集（幂集）。
 */
class Solution {
public:
	vector<vector<int>> subsetsWithDup(vector<int>& nums) {
		vector<vector<int>> res;
		vector<int> cur;
		sort(nums.begin(), nums.end()); // 排序 比较前后元素进行剪枝
		backtrack(nums, res, cur, 0);
		return res;
	}

	void backtrack(vector<int>& nums, vector<vector<int>>& res, vector<int> cur, int start)
	{
		res.push_back(cur);
		for (int i = start; i < nums.size(); i++)
		{
			if(i>start && nums[i]==nums[i-1]) //剪枝
				continue;
			cur.push_back(nums[i]);
			backtrack(nums, res, cur, i + 1);
			cur.pop_back();
		}
	}
};
```



### 组合

```c++
/**
 * Leetcode_77
 * 给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。
 */
class Solution {
public:
    vector<vector<int>> res;
	vector<int> cur;
	vector<vector<int>> combine(int n, int k) {		
		backtrack(n, 1, k);
		return res;
	}

	void backtrack(int n, int start, int k)
	{
		if (cur.size() + (n - start+1) < k) //剪枝
			return;
		if (cur.size() == k)
		{
			res.push_back(cur);
			return;
		}
		for (int i = start; i <= n; i++)
		{
			cur.push_back(i);
			backtrack(n, i+1, k);
			cur.pop_back();
		}
	}
};
```



### 组合总和（每个数字可以重复被选取）

```c++
/**
 * Leetcode_39
 * 给定一个无重复元素的数组 candidates 和一个目标数 target ，
 * 找出 candidates 中所有可以使数字和为 target 的组合
 */
class Solution {
public:	
	vector<vector<int>> res;
	vector<int> cur;
	vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
		int sum = 0;
		sort(candidates.begin(), candidates.end());
		backtrack(target, sum, candidates, 0);
		return res;
	}
	void backtrack(int target, int sum, vector<int>& candidates, int start) {
		if (sum == target)
		{
			res.push_back(cur);
			return;
		}
		else if (sum > target)
			return;
		for (int i = start; i < candidates.size();i++)
		{
			if(sum+candidates[i]>target) //剪枝
				break;
			cur.push_back(candidates[i]);
			sum += candidates[i];
			backtrack(target, sum, candidates, i);
			cur.pop_back();
			sum -= candidates[i];
		}
	}
};
```



### 组合总和2（每个数字只能使用一次）

```c++
/**
 * Leetcode_40
 * 给定一个数组 candidates 和一个目标数 target ，
 * 找出 candidates 中所有可以使数字和为 target 的组合
 */
#include <vector>
#include <algorithm>
using namespace std;
class Solution {
public:
	vector<vector<int>> res;
	vector<int> cur;
	vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
		int sum = 0;
		sort(candidates.begin(), candidates.end());
		backtrack(target, sum, candidates, 0);
		return res;
	}
	void backtrack(int target, int sum, vector<int>& candidates, int start) {
		if (sum == target)
		{
			res.push_back(cur);
			return;
		}
		else if (sum > target)
			return;
		for (int i = start; i < candidates.size(); i++)
		{
			// 剪枝
			if (sum + candidates[i]>target) // 后面更大的元素和更大
				break;
			// 重复子集
			if (i - start > 0 && candidates[i] == candidates[i - 1])
				continue;
			cur.push_back(candidates[i]);
			sum += candidates[i];
			backtrack(target, sum, candidates, i+1);
			cur.pop_back();
			sum -= candidates[i];
		}
	}
};
```



## 排列

### 全排列

```c++
/**
 * Leetcode_46
 * 给定一个不含重复数字的数组 nums ，返回其 所有可能的全排列 。
 * 你可以 按任意顺序 返回答案。
 */
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
		vector<vector<int>> res;
		if (nums.size() == 0)
			return{};
		dfs(res, nums, 0);
		return res;
	}
	void dfs(vector<vector<int>>& res, vector<int>&nums, int first)
	{
		if (first == nums.size())
		{
			res.push_back(nums);
			return;
		}

		for (int i = first; i < nums.size(); i++)
		{
			// 选择
			if(first !=i)
				swap(nums[first], nums[i]);
			// 回溯
			dfs(res, nums, first+1);
			// 撤销选择
			if (first != i)
				swap(nums[first], nums[i]);
		}
	}
};


```



### 全排列2

```c++
/**
 * Leetcode_47
 * 给定一个可包含重复数字的序列 nums ，按任意顺序 返回所有不重复的全排列
 */
// swap
class Solution0 {
public:
	vector<vector<int>> res;
	vector<vector<int>> permuteUnique(vector<int>& nums) {
		sort(nums.begin(), nums.end());
		backtrack(nums, 0);
		return res;
	}

	void backtrack(vector<int>& nums, int start) {
		if (start == nums.size())
		{
			res.emplace_back(nums);
			return;
		}
		for (int i = start; i < nums.size(); i++)
		{
			// 交换后，后面的元素可能无序
			sort(nums.begin() + start, nums.end());
			if (i > start && nums[i] == nums[i - 1])
				continue;
			if (i != start)
				swap(nums[i], nums[start]);
			backtrack(nums, start + 1);
			if (i != start)
				swap(nums[i], nums[start]);
		}
	}
	
};
// 使用visited数组
class Solution {
	vector<vector<int>> res;
	vector<int> path;
	vector<int> vis;
public:
	
	vector<vector<int>> permuteUnique(vector<int>& nums) {
		sort(nums.begin(), nums.end());
		vis.resize(nums.size());
		backtrack(nums);
		return res;
	}

	void backtrack(vector<int>& nums) {
		if (path.size() == nums.size())
		{
			res.emplace_back(path);
			return;
		}
		for (int i = 0; i < nums.size(); i++)
		{
            // 1、这个数字选过了跳过
			// 2、剪枝：当前数字和前一个数字相等，且前一个数字未被选过
			if (vis[i] || (i> 0 && nums[i] == nums[i - 1] && !vis[i-1]))
				continue;
			path.emplace_back(nums[i]);
			vis[i] = true;
			backtrack(nums);
			path.pop_back();
			vis[i] = false;
		}
	}
};
```



### 字符串的全排列

```c++
/**
 * 剑指Offer_387
 * 输入一个字符串，打印出该字符串中字符的所有排列
 */
// swap

class Solution {
	vector<string> res;
public:
	vector<string> permutation(string s) {
		backtrack(s, 0);
		return res;
	}
	void backtrack(string s, int start) {
		if (start == s.length())
		{
			res.emplace_back(s);
			return;
		}
		for (int i = start; i < s.length();i++)
		{
			sort(s.begin()+start, s.end());
			if(i>start && s[i]==s[i-1])
				continue;
			if (i != start)
				swap(s[i], s[start]);
			backtrack(s, start + 1);
			if (i != start)
				swap(s[i], s[start]);
		}
	}
};

// 使用visited数组
class Solution {
	vector<string> res;
	string path;
	vector<int> vis;
public:
	vector<string> permutation(string s) {
		vis.resize(s.length());
		sort(s.begin(), s.end());
		backtrack(s);
		return res;
	}
	void backtrack(string s) {
		if (path.length() == s.length())
		{
			res.emplace_back(path);
			return;
		}
		for (int i = 0; i < s.length(); i++)
		{
			if (vis[i] || (i>0 && s[i]==s[i-1] && !vis[i-1])) 
				continue;
			path.push_back(s[i]);
			vis[i] = true;
			backtrack(s);
			path.pop_back();
			vis[i] = false;
		}
	}
};
```



### 字母大小写全排列

```c++
/**
 * Leetcode_784
 * 给定一个字符串S，通过将字符串S中的每个字母转变大小写，
 * 我们可以获得一个新的字符串。返回所有可能得到的字符串集合
 */
class Solution {
private:
	vector<string> res;
	void backtrack(string s, int start) {
		if (start == s.length())
		{
			res.push_back(s);
			return;
		}

		if (isalpha(s[start])) { // 字母
			if(isupper(s[start]))
				s[start] = tolower(s[start]);
			backtrack(s, start + 1);
			if (islower(s[start]))
				s[start] = toupper(s[start]);
			backtrack(s, start + 1);
		}
		else {
			backtrack(s, start + 1);
		}
	}
public:
	vector<string> letterCasePermutation(string s) {
		backtrack(s, 0);
		return res;
	}	
};
```



## 搜索

### 单词搜索

```c++
/**
 * Leetcode_79
 * 给定一个 m x n 二维字符网格 board 和一个字符串单词 word 。
 * 如果 word 存在于网格中，返回 true ；否则，返回 false 。
 * 单词必须按照字母顺序，通过相邻的单元格内的字母构成，
 * 其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。
 * 同一个单元格内的字母不允许被重复使用。
 */
#include <vector>
using namespace std;
class Solution {
public:
	bool exist(vector<vector<char>>& board, string word) {
		if (board.size() == 0 || board[0].size() == 0 || word.length() == 0)
			return false;
		for (int i = 0; i < board.size();i++)
		{
			for (int j = 0; j < board[0].size();j++)
			{
				if (board[i][j] == word[0] && dfs(board, word, 0, i, j))
					return true;
			}
		}
		return false;
	}
private:
	bool dfs(vector<vector<char>>& board, string word, int start, int i, int j) {
		if (i < 0 || i >= board.size() || j<0 || j>=board[0].size() || board[i][j] != word[start])
			return false;
		if (start == word.size()-1) {
			return true;
		}
		char temp = board[i][j];
		board[i][j] = '\0';
		bool res = dfs(board, word, start + 1, i+1, j)
			|| dfs(board, word, start + 1, i-1, j)
			|| dfs(board, word, start + 1, i, j+1)
			|| dfs(board, word, start + 1, i, j-1);
		board[i][j] = temp;
		return res;
	}
};
```



### N皇后

```c++
/**
 * Leetcode_51
 * n 皇后问题 研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。
 * 给你一个整数 n ，返回所有不同的 n 皇后问题 的解决方案。
 * 每一种解法包含一个不同的 n 皇后问题 的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。
 */
class Solution {
public:
	vector<vector<string>> solveNQueens(int n) {
		string a;
		for (int i = 0; i < n; i++)
			a.push_back('.');
		for (int i = 0; i < n; i++)
			path.push_back(a);

		dfs(0, n);

		return res;
	}
private:
	vector<vector<string>> res;
	vector<string> path;
	bool col[9], gl[9], ugl[9]; // 列、斜线、斜线
	void dfs(int start, int n) {
		if (start == n)
		{
			res.push_back(path);
			return;
		}

		// 第start个皇后从n列中选位置
		for (int i = 0; i < n; i++)
		{
			if (!col[i] && !gl[start + i] && !ugl[n - start + i])
			{
				col[i] = gl[start + i] = ugl[n - start + i] = true;
				path[start][i] = 'Q';
				dfs(start + 1, n);
				path[start][i] = '.';
				col[i] = gl[start + i] = ugl[n - start + i] = false;
			}
		}
	}
};
```



### 分割回文串

```c++
/*
 * 给定一个字符串 s，将 s 分割成一些子串，使每个子串都是回文串。
 * 返回 s 所有可能的分割方案。
*/
#include <vector>
using namespace std;
class Solution {
public:
	vector<vector<string>> partition(string s) {
		len = s.length();
		dp.assign(len, vector<int>(len, true));
		for (int i = len-1; i >=0;i--)
		{
			for (int j = i+1; j < len; j++)
			{
				dp[i][j] = (s[i] == s[j]) && dp[i + 1][j - 1];				
			}
		}
		dfs(s, 0);
		return res;
	}
private:
	vector<vector<string>> res;
	vector<string> path;
	vector<vector<int>> dp; // dp[i][j]: s[i:j]是否是回文串
	int len;
	void dfs(string s, int start) {
		if (start == len) {
			res.emplace_back(path);
			return;
		}
		for (int i = start; i < s.length();i++)
		{
			if(!dp[start][i])
				continue;
			string subS = s.substr(start, i - start + 1);
			path.emplace_back(subS);
			dfs(s, i + 1);
			path.pop_back();
		}
	}
};
```



# 背包问题

![416.分割等和子集1](images/bagProblemRelationship.png)

## 经典01背包问题（二维数组）

```c++
void backpack01(){
    vector<int> weight = {1, 3, 4};
    vector<int> value = {15, 20, 30};
    int bagWeight = 4;

    int len=weight.size();
    // 二维数组
    vector<vector<int>> dp(len, vector<int>(bagWeight + 1, 0));

    // 初始化
    for (int j = weight[0]; j <= bagWeight; j++) {
        dp[0][j] = value[0];
    }

    // weight数组的大小 就是物品个数
    for(int i = 1; i < len; i++) { // 遍历物品
        for(int j = 0; j <= bagWeight; j++) { // 遍历背包容量
            if (j < weight[i])
                dp[i][j] = dp[i - 1][j];
            else
                dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);

        }
    }

    // 输出dp数组
    for (int i=0; i<len; i++) {
        for (int j=0; j<=bagWeight; j++) {
            cout<<dp[i][j]<<"\t";
        }
        cout<<endl;
    }
}
```

## 经典01背包问题（滚动数组）

```c++
void backpack01() {
    vector<int> weight = {1, 3, 4};
    vector<int> value = {15, 20, 30};
    int bagWeight = 4;

    // 初始化
    vector<int> dp(bagWeight + 1, 0);
    for(int i = 0; i < weight.size(); i++) { // 遍历物品
        for(int j = bagWeight; j >= weight[i]; j--) { // 遍历背包容量
            dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
        }
    }
    for(int x:dp)
        cout<<x<<"\t";
    cout<<endl;
}
```



## 完全背包问题

- 01背包内嵌的循环是从大到小遍历，为了保证每个物品仅被添加一次
- 完全背包问题遍历物品的顺序为从前往后，这样子就可以选择多个物品

```c++
// 先遍历物品，在遍历背包
void test_CompletePack() {
    vector<int> weight = {1, 3, 4};
    vector<int> value = {15, 20, 30};
    int bagWeight = 4;
    vector<int> dp(bagWeight + 1, 0);
    for(int i = 0; i < weight.size(); i++) { // 遍历物品
        for(int j = weight[i]; j <= bagWeight; j++) { // 遍历背包容量
            dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
        }
    }
    cout << dp[bagWeight] << endl;
}
```



### 完全背包的组合和排列问题

- 如果求组合数就是外层for循环遍历物品，内层for遍历背包。
- 如果求排列数就是外层for遍历背包，内层for循环遍历物品。

# 动态规划

## 目标和

```c++
/*
Leetcode_494
给你一个整数数组 nums 和一个整数 target 。

向数组中的每个整数前添加 '+' 或 '-' ，然后串联起所有整数，可以构造一个 表达式 ：

例如，nums = [2, 1] ，可以在 2 之前添加 '+' ，在 1 之前添加 '-' ，然后串联起来得到表达式 "+2-1" 。
返回可以通过上述方法构造的、运算结果等于 target 的不同 表达式 的数目。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/target-sum
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
*/
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum=0;
        for(int n:nums)
            sum+=n;
        if(target>sum || (sum+target)&1)
            return 0;
        int bagW=(sum+target)/2;
        int len=nums.size();
        vector<int> dp(bagW+1);
        dp[0]=1;
        for (int n:nums) {
            for (int j=bagW; j>=n; j--) {
                dp[j]+=dp[j-n];
            }
        }
        return dp[bagW];
    }
};
```

## 一和零

背包维度是2维的01背包问题

```c++
/*给你一个二进制字符串数组 strs 和两个整数 m 和 n 。

请你找出并返回 strs 的最大子集的大小，该子集中 最多 有 m 个 0 和 n 个 1 。

如果 x 的所有元素也是 y 的元素，集合 x 是集合 y 的 子集 。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/ones-and-zeroes
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
*/
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        vector<vector<int>> dp(m+1, vector<int>(n+1));
        int zerosNum,onesNum;
        for (string s : strs) {
            zerosNum=onesNum=0;
            for(char c:s){
                if (c=='0')
                    zerosNum++;
                else
                    onesNum++;
            }
                
            for (int i=m; i>=zerosNum; i--) {
                for (int j=n; j>=onesNum; j--) {
                    dp[i][j]=max(dp[i][j], dp[i-zerosNum][j-onesNum]+1);
                }
            }
        }
        return dp[m][n];
    }
};
```



并查集

```C++
class UnionFind {
public:
	UnionFind(int _n) :sets(_n), n(_n), rank(_n), parent(_n) {
		for (int i = 0; i < n; i++)
			parent[i] = i;
	}
	void merge(int x, int y) {
		int rootX = find(x); 
		int rootY = find(y);
		if (rootX != rootY) {
			if (rank[rootX] > rank[rootY]) {
				parent[rootY] = rootX;
			}
			else if (rank[rootX] < rank[rootY]) {
				parent[rootX] = rootY;
			}
			else {
				parent[rootY] = rootX;
				rank[rootX]++;
			}
			sets--;
		}
	}
	int find(int x) {
		if (x == parent[x])
			return x;
		return parent[x]=find(parent[x]);
	}

	int getSets() {
		return sets;
	}
private:
	int sets, n; // 集合数量, 点数量
	vector<int> parent, rank;
};


class Solution {
public:
	int findCircleNum(vector<vector<int>>& isConnected) {
		int n = isConnected.size();
		UnionFind uf(n);
		for (int i = 0; i < n; i++)
			for (int j = 0; j < i; j++)
				if (isConnected[i][j])
					uf.merge(i, j);
		return uf.getSets();
	}
};
```



# 笔试题

## 迷宫问题

迷宫从左上角到右下角

```c++
/*
链接：https://www.nowcoder.com/questionTerminal/cf24906056f4488c9ddb132f317e03bc?answerType=1&f=discussion
来源：牛客网

定义一个二维数组N*M（其中2<=N<=10;2<=M<=10），如5 × 5数组下所示：

int maze[5][5] = {
0, 1, 0, 0, 0,
0, 1, 0, 1, 0,
0, 0, 0, 0, 0,
0, 1, 1, 1, 0,
0, 0, 0, 1, 0,
};


它表示一个迷宫，其中的1表示墙壁，0表示可以走的路，只能横着走或竖着走，不能斜着走，
要求编程序找出从左上角到右下角的最短路线。入口点为[0,0],既第一空格是可以走的路。
*/
// 存储路径的容器可以用vector或者stack
#include <iostream>
#include <vector>
#include <stack>
using namespace std;
bool dfs(vector<vector<int>>& a, vector< pair<int, int> >& path, int x, int y);
void printStackReverse(stack< pair<int, int> > stk);

int main()
{
	int N, M;
	//stack< pair<int, int> > stk;
	vector< pair<int, int> >path;
	cin >> N >> M;
	
	vector<vector<int>> a(N, vector<int>(M, 1));

	for (int i = 0; i < N; i++)
	{
		for (int j = 0; j < M; j++)
		{
			cin >> a[i][j];
		}
	}
	
	//dfs(a, stk, 0, 0);
	dfs(a, path, 0, 0);
}

bool dfs(vector<vector<int>>& a, vector< pair<int, int> >& path, int x, int y)
{
	if (x<0 || x>a.size() - 1 || y<0 || y>a[0].size() - 1 || a[x][y] != 0)
		return false;
	if (x == a.size() - 1 && y == a[0].size() - 1)
	{
		//stk.push(make_pair(x, y));
		//printStackReverse(stk);
		path.push_back(make_pair(x, y));
		for (auto p:path)
		{
			cout << "(" << p.first << "," << p.second << ")" << endl;
		}
        // 如果结果需要保存的话可以再定义一个vector，将最终的路径赋值给它
		return true;
	}
	a[x][y] = 2; //走过的路用2表示
	//stk.push(make_pair(x, y));
	path.push_back(make_pair(x, y));

	/*bool res = dfs(a, stk, x, y + 1)
		|| dfs(a, stk, x + 1, y)
		|| dfs(a, stk, x, y-1)
		|| dfs(a, stk, x - 1, y);*/

	bool res = dfs(a, path, x, y + 1)
		|| dfs(a, path, x + 1, y)
		|| dfs(a, path, x, y - 1)
		|| dfs(a, path, x - 1, y);

	// 撤销选择
	a[x][y] = 0;
	//stk.pop();
	path.pop_back();
	return res;
}

// 从底向上输出栈
void printStackReverse(stack< pair<int, int> > stk)
{
	if (stk.empty())
		return;
	else {
		pair<int, int> p = stk.top();
		stk.pop();
		printStackReverse(stk);
		cout << "("<<p.first<<","<<p.second<<")" << endl;
	}
}
```



## 迷宫摸墙走

```c++
/*
作者：斯皮尔伯格·渣渣辉
链接：https://www.nowcoder.com/discuss/643474?type=0&order=0&pos=7&page=0&channel=-1&source_id=discuss_center_0_nctrack
来源：牛客网

迷宫左手法则多少步能走出迷宫。
左手法则：沿着墙壁走，如果左边没墙壁，就往左走，否则看前面有没有墙壁，没有往前走，
否则右面有没有墙壁往右走，否则往后走。
#为墙壁，.为路，S为开始位置，E为结束位置。
*/
#include <iostream>
#include <vector>
using namespace std;
const int dir[4][2] = { {-1, 0}, {0, 1}, {1, 0}, {0, -1} }; // 上右下左
bool dfs(vector<vector<char>>& a, int x, int y, int step, int& step_final, int cur_dir);

int main()
{
	int T, w, h;
	int s1, s2; // 起点坐标
	int step, step_final; //步数
	cin >> T;
	vector< pair<int, int> >path;	

	while (T--)
	{
		/*cin >> w >> h;
		vector<vector<char>> a(h, vector<char>(w));
		for (int i = 0; i < w; i++)
		{
			for (int j = 0; j < h; j++)
			{
				cin >> a[i][j];
				if (a[i][j] == 'S')
				{
					s1 = i;
					s2 = j;
				}
			}
		}*/
		w = 5; h = 5;
		vector<vector<char>> a = {
			{ '#', '#', '#', '#', '#' },
			{ '#', '.', '.', '.', '#' },
			{ '#', '.', '.', '.', '#' },
			{ '#', '.', '#', '.', '#' },
			{ '#', 'E', '#', 'S', '#' }
		};
		step = 0;
		s1 = 4; s2 = 3;
		int cur_dir;
		if (s1 == 0)
			cur_dir = 2;
		else if (s1 == h-1)
			cur_dir = 0;
		else if (s2 == 0)
			cur_dir = 3;
		else
			cur_dir = 1;
		
		dfs(a, s1, s2, step, step_final, cur_dir);
		cout << step_final << endl;
	}
	
}

bool dfs(vector<vector<char>>& a, int x, int y, int step, int& step_final, int cur_dir)
{
	if (x<0 || x>a.size() - 1 || y<0 || y>a[0].size() - 1 || a[x][y] == '#')
		return false;
	if (a[x][y]=='E')
	{
		step++;
		step_final = step;
		return true;
	}
	step++;
	a[x][y] = '#'; //标记走过的路	

	// 三个方向的坐标都不一样
	int next1_dir = (cur_dir + 3) % 4; // 左
	int next2_dir = cur_dir; // 前
	int next3_dir = (cur_dir + 1) % 4; // 右
	int next4_dir = (cur_dir + 2) % 4;; // 后

	bool res = dfs(a, x + dir[next1_dir][0], y + dir[next1_dir][1], step, step_final, next1_dir)
		|| dfs(a, x + dir[next2_dir][0], y + dir[next2_dir][1], step, step_final, next2_dir)
		|| dfs(a, x + dir[next3_dir][0], y + dir[next3_dir][1], step, step_final, next3_dir)
		|| dfs(a, x + dir[next4_dir][0], y + dir[next4_dir][1], step, step_final, next4_dir);

	// 撤销选择
	a[x][y] = '.';

	return res;
}
```



## 旅行商问题

这个代码超时了！！！

```c++
/*
旅行商问题 
*/
#include <iostream>
#include <vector>
using namespace std;

void dfs(vector<int>& res, vector<int> path, vector<bool> used, int &minLen, int len, vector<vector<int>> a, int pre)
{

	if (path.size() == used.size()-1)
	{
		// 返回原点
		len+= a[pre][0];
		path.push_back(0);

		if (len < minLen)
		{
			minLen = len;
			res = path;
		}
		return;
	}

	for (int i = 1; i < used.size(); i++)
	{
		if (used[i])
			continue;

		// 选择
		path.push_back(i);
		used[i] = true;
		len += a[pre][i];
		// 剪枝
		if (len > minLen)
			continue;
		//pre = i;
		// 回溯
		dfs(res, path, used, minLen, len, a, i);

		// 撤销选择
		path.pop_back();
		used[i] = false;
		len -= a[pre][i];
	}
}

int main()
{
	int n; // 城市个数
	//int a[100][100]; // 距离矩阵
	vector<int> path, res; //路径
	
	int minLen = 1000; //路径长度
	cin >> n;
	//n = 4;

	vector<vector<int>> a(n, vector<int>(n, 0)); // 距离矩阵
	vector<bool> used(n, false);
	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < n; j++)
		{
			cin >> a[i][j];
		}
	}
	

	/*a = {
		{0, 2, 6, 5},
		{2, 0, 4, 4},
		{6, 4, 0, 2},
		{5, 4, 2, 0} 
	};*/
	dfs(res, path, used, minLen, 0, a, 0);
	cout << minLen << endl;
}
```



## 赏金猎人

```c++
#include<bits/stdc++.h>
using namespace std;

int main() {
	int n, k;
	cin >> n >> k;
	vector<vector<int> > arr(n, vector<int>(3));
	for (int i = 0; i < n; i++) {
		arr[i][2] = i;  // 记录编号
		cin >> arr[i][0];  // 输入attack
	}
	for (int i = 0; i < n; i++) {
		cin >> arr[i][1];  // 输入money
	}
	sort(arr.begin(), arr.end());  // 按战斗力排序
	int sum = 0;
	vector<int> res(n);
	priority_queue<int, vector<int>, greater<int> > Q;
	for (auto& it : arr) {  // 后面的肯定能战胜前面所有的
		int money = it[1], i = it[2];
		sum += money;
		res[i] = sum;
		Q.push(money);
		if (Q.size() > k) {
			sum -= Q.top();
			Q.pop();
		}
	}
	for (auto x : res) {
		cout << x << " ";
	}
	cout << endl;
	return 0;
}
```



- 卖鸡蛋饼更换纸币（5、10、20元，找不开钱收摊）
- 链表相减
  - 计算两个代表整数的链表参与减法运算后的结果，通过链表形式返回
- 链表子序列最小
  - 2->2->2->5->2 的最小子序列是 2->2->2->2->5

## 华为08.25笔试

### 第一题

```c++
// 简单的写法
#include <bits/stdc++.h>
using namespace std;

int main()
{
	/*vector<vector<int>> nums = {
		{1, -1,2,3},
		{-3,2,-4,5},
		{6,8,-1,6}
	};
	int n = nums.size(), m = nums[0].size();*/
	
	int n, m;
	cin >> n >> m;
	vector<vector<int>> nums(n, vector<int>(m));
	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < m; j++)
		{
			cin >> nums[i][j];
		}
	}
	int ma=INT_MIN;
	for (int i = 0; i < n; i++)
	{
		vector<int> sums(m); // 第i层到第j层的和
		for (int j = i; j < n; j++)
		{
			int curSum = 0;
			for (int k = 0; k < m; k++)
			{
				sums[k] += nums[j][k];
				if (curSum > 0)
					curSum += sums[k];
				else
					curSum = sums[k];
				ma = max(ma, curSum);
			}
		}
	}
	cout << ma << endl;
}
```



### 第二题

```c++
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> dirs = { {1,0},{0,1},{-1,0},{0,-1} };
int main()
{
	int m, n;
	cin >> m >> n;
	vector<vector<int>> nums(m, vector<int>(n));
	for (int i = 0; i < m; i++) {
		for (int j = 0; j < n; j++)
		{
			cin >> nums[i][j];
		}
	}
	vector<vector<bool>> vi(m, vector<bool>(n));

	int t = 0;
	queue<vector<int>> que;
	que.push({ 0,0 });
	int flag = false; // 到达出口
	while (!que.empty() && !flag)
	{
		int size = que.size();
		for (int i = 0; i < size; i++)
		{
			auto xy = que.front();
			que.pop();
			int x = xy[0], y = xy[1];
			vi[x][y] = true;
			if (x == m - 1 && y == n - 1) {
				flag = true;
				break;
			}
			// 看四个方向
			for (auto& dir : dirs) {
				int nx = x + dir[0], ny = y + dir[1];
				if(nx<0 || ny<0 || nx>=m || ny>=n || vi[nx][ny])
					continue;
				if (t >= nums[nx][ny]) {
					continue;
				}
				que.push({ nx, ny });
			}
		}
		t++;
	}
	t--;
	if (flag)
		cout << t << endl;
	else
		cout << -1 << endl;
}
```

### 第三题

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{
	int n;
	cin >> n;
	vector<vector<int>> adj(n);
	vector<int> times(n);
	
	for (int i = 0; i < n; i++)
	{
		vector<int> data;
		int tmp;
		while (cin>>tmp)
		{
			data.push_back(tmp);
			if (cin.get() == '\n')
				break;
		}
		for (int j = 0; j < data.size()-1; j++)
		{
			if (data[j] != -1)
				adj[data[j]].push_back(i);
		}
		times[i] = data[data.size() - 1];
	}

	

	vector<int> indeg(n);
	
	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < adj[i].size();j++)
			indeg[adj[i][j]]++;
	}
	vector<int> cost(times); // 每个任务完成的时间
	queue<int> que;
	for (int i = 0; i < n; i++)
	{
		if (indeg[i] == 0) {
			que.push(i);
		}			
	}

	while (!que.empty())
	{
		int p1 = que.front();
		que.pop();
		for (int& p2 : adj[p1])
		{
			indeg[p2]--;
			// 通过p1前往p2
			cost[p2] = max(cost[p2], cost[p1] + times[p2]);
			if (indeg[p2] == 0) {
				que.push(p2);
			}
		}
	}
	for (int& in:indeg)
	{
		if (in != 0) {
			cout << -1 << endl;
			return 0;
		}
	}
	int res = *max_element(cost.begin(), cost.end());
	cout << res << endl;
}
```



## 阿里08.02笔试

- 迷宫，每个点有进入时间限制

```c++
#include <bits/stdc++.h>
using namespace std;
struct Point {
	int x;
	int y;
	int t;
};
const int dir[4][2] = { {1,0},{0,1},{-1,0},{0,-1} };

int main()
{
	int T;
	scanf("%d", &T);
	while (T--)
	{
		int n, m;
		scanf("%d%d", &n,&m);
		Point startP;
		vector<vector<int>> board= {
			{0,1,2},{1,2,2},{2,2,4}
		};
		vector<vector<bool>> visited(n, vector<bool>(m));

		for (int i = 0; i < n; i++)
		{
			for (int j = 0; j < m; j++)
			{
				scanf("%d", &board[i][j]);
				if (board[i][j] == 0) {
					startP = {i, j, 0};
				}
			}
		}
		queue<Point> que;
		que.push(startP);
		bool flag = false;
		visited[0][0] = true;
		while (!que.empty())
		{
			Point p = que.front();
			que.pop();
			if (p.x == n - 1 && p.y == m - 1) {
				flag = true;
				break;
			}
			for (auto d:dir)
			{
				Point p2 = { p.x+d[0], p.y+ d[1], p.t+1 };
				if (p2.x >= 0 && p2.x < n && p2.y >= 0 && p2.y < m 
					&& p2.t <= board[p2.x][p2.y] && !visited[p2.x][p2.y]) {
					que.push({p2.x, p2.y, p2.t});
					visited[p2.x][p2.y] = true;
				}
			}
		}
		printf("%s\n", flag ? "YES" : "NO");
	}
	
}
```

## 腾讯09.05笔试

第一题

```c++
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
	/**
	* 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
	*
	* @param a ListNode类vector 指向这些数链的开头
	* @return ListNode类
	*/
	ListNode* solve(vector<ListNode*>& a) {
		// write code here
		int n = a.size();
		ListNode* head=new ListNode(0);
		ListNode* h = head;
		
		vector<bool> isempty(n, false);
		for (int i = 0; i < n; i++)
		{
			if (a[i] == nullptr)
				isempty[i] = true;
		}
		int count = 0; //统计为空的个数
		while (true)
		{
			count = 0;
			for (int i = 0; i < n; i++)
			{
				if (isempty[i]) {
					count++;
					continue;
				}
				if (a[i] != nullptr) {
					ListNode* tmp = new ListNode(a[i]->val);
					h->next = tmp;
					h = h->next;
					a[i] = a[i]->next;
					if (a[i] == nullptr) {
						isempty[i] = true;
					}
				}				
			}
			if(count==n)
				break;
		}
		
		return head->next;
	}
};
```

第二题

```c++
#include <bits/stdc++.h>
using namespace std;


int getFactorNums(int x) {
	if (x == 1) return 1;
	if (x == 2) return 2;
	int k = 1;
	for (int i = 2; i <= x; i++)
	{
		if (x%i == 0)
			k++;
	}
	return k;
}
int main()
{
	int n;
	cin >> n;
	vector<int> a(n);
	vector<int> b(n);
	vector<int> nums1(n);
	vector<int> nums2(n);

	for (int i = 0; i < n;i++)
	{
		cin >> a[i];
		nums1[i] = getFactorNums(a[i]);
	}
	for (int i = 0; i < n; i++)
	{
		cin >> b[i];
		nums2[i] = getFactorNums(b[i]);
	}

	sort(nums1.begin(), nums1.end());
	sort(nums2.begin(), nums2.end());
	int idx = -1; // a中第一支比b大的
	for (int i = 0; i < n; i++)
	{
		if (nums1[i] > nums2[0]) {
			idx = i;
			break;
		}
	}
	if (idx == -1) {
		cout << 0 << endl;
		return 0;
	}

	int count = 0;
	int j = 0;
	for (int i = idx; i < n; i++)
	{
		if (nums1[i]>nums2[j]) {
			count++;
			j++;
		}
	}
	cout << count << endl;
}
```

第三题

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{
	int n;
	cin >> n;
	string s;
	cin >> s;
	
	vector<int> a(n + 1);
	for (int i = 1; i <=n; i++)
	{
		a[i] = s[i-1] - '0';
	}
	vector<vector<int>> l(2, vector<int>(n + 2)), r(2, vector<int>(n + 2));
	for (int i = 1; i <=n; i++)
	{
		l[0][i] += l[0][i - 1];
		l[1][i] += l[1][i - 1]; 
		l[a[i]][i] += 1;
	}

	for (int i = n; i >=1; i--)
	{
		r[0][i] += r[0][i + 1];
		r[1][i] += r[1][i + 1];
		r[a[i]][i] += 1;
	}

	int res = 0;
	for (int i = 0; i <=n; i++)
	{
		res = max(res, (l[0][i] + 1)*l[0][i] + (r[1][i + 1] + 1)*r[1][i + 1]);
		res = max(res, (l[1][i] + 1)*l[1][i] + (r[0][i + 1] + 1)*r[0][i + 1]);
	}
	cout << res/2 << endl;
}
```

第四题

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{
	long long n;
	int l, r;
	cin >> n >> l >> r;
	vector<int> nums(1);
	nums[0] = n;
	for (int i = 0; i < nums.size();)
	{
		if (nums[i] > 1) {
			int tmp1 = nums[i] / 2;
			int tmp2 = nums[i] % 2;
			nums.erase(nums.begin() + i);
			nums.insert(nums.begin() + i, { tmp1, tmp2, tmp1 });
		}
		else {
			i++;
		}
	}
	int count = 0;
	for (int i = l-1; i <=r-1; i++)
	{
		if (nums[i] == 1)
			count++;
	}
	cout << count << endl;
}

```

第五题

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{
	int n;
	cin >> n;
	vector<int> a(n+1);

	for (int i = 1; i <= n; i++)
	{
		cin >> a[i];
	}

	vector<vector<int>> mi(n + 1, vector<int>(n + 1));
	for (int i = 1; i <=n; i++)
	{
		int pre = a[i];
		for (int j = i; j <=n; j++)
		{
			pre = min(pre, a[j]);
			mi[i][j] = pre;
		}
	}
	int count = 0;
	for (int i = 1; i <=n; i++)
	{
		for (int j = i + 1; j <= n;j++)
		{
			int ma = max(a[i], a[j]);
			if (i + 1 == j)
				count++;
			else if (ma <= mi[i + 1][j - 1])
				count++;
		}
	}
	cout << count << endl;
}
```

# 面试题

## 网易雷火

- 随机打乱数组并证明正确性

- k大小滑动窗口中最大数和最小数的差的最大值（两个单调队列维护一下）

- 手写最小堆

- 给一个凸多边形，求三角剖分的最小边长（简单dp）

  https://blog.csdn.net/Jayphone17/article/details/102581362

## 华为

- 有序数组删除重复的值 √ LeetCode_26
- 删除链表倒数第k个√ LeetCode_26
- 最长无重复字串 × LeetCode_3
- 字符串最长回文数 × LeetCode_5
- 两个链表节点值相加 √ leetcode 445
- 大意是给N，找出1到N中能被N整除的数。一开始直接就循环直接看余数是否等于零。面试官提示减少循环次数，答一次找出两个数 例如N是9 一次性找出1和9 √ Moni-01
- 排序两个有序链表 √ LeetCode_21 
- 有效的括号 √ LeetCode_20 
- 二叉树的最小深度 √ LeetCode_111
- 作者：我会个锤子的算法
  链接：https://www.nowcoder.com/discuss/643910?type=2&order=3&pos=11&page=1&channel=-1&source_id=discuss_tag_nctrack
- 快排
- 给一个字符串 判断其作为密码的安全性。就是根据一些字母个数啊 数字个数啊 特殊字符来打分
- 是m*n网格的寻路问题，网格的每个位置有对应的cost，问找出最接近cost t的路线（即时间不能超过t，但需要最大），动态规划应该可解
- 写一个算法判断某个数是2的n次方
- 编程题：一个单词数组，再给你一个长字符串，要求找出字符串能够拼成的单词。(比如appabcabcle里面有个apple)
- 算法题是重组一个字符串。给一个字符串，按照里面字符出现的频率排序这个字符串，频数相同顺序随意。
- 最大公共前缀 （一开始用前缀树做，后续调试出了问题。时间快到改用循环做）
- 寻找数组里等于目标值的最长连续字串
- 拼接最大字符串，比如：1, 20, 9　-> 9, 20, 1
- 手撕代码：反转链表三种方法，求二叉树的深度两种方法。要求先说思路，然后写代码，写完代码再照着代码讲一遍思路。
- 股票的最大利润 剑指offer63

## 字节

- 最长无重复子串（最长重复字串）
- 升序序列的查找出现目标target第一次出现的位置
- 删除一个数组中指定出现的某个数