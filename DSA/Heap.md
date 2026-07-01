
A **Heap** is a complete binary tree data structure that satisfies the heap property:

In a min-heap, the value of a node is always less than or equal to its children.

In a max-heap, the value of a node is always greater than or equal to its children.


### Implementation

```cpp
class PriorityQueue
{
    int sz = 0;
    int heap[1001]{};

private:

    void heapify(int i)
    {
        while(2 * i <= sz || 2 * i + 1 <= sz)
        {
            int j = i;

            if(2 * i <= sz && heap[2 * i] < heap[j]) j = 2 * i;
            if(2 * i + 1 <= sz && heap[2 * i + 1] < heap[j]) j = 2 * i + 1;

            if(j == i) break;

            swap(heap[i], heap[j]);

            i = j;
        }
    }

public:

    /* push */
    void push(int x)
    {   
        sz++;

        heap[sz] = x;

        int i = sz;

        while(i != 1)
        {
            if(heap[i / 2] > heap[i])
            {
                swap(heap[i / 2], heap[i]);
                i /= 2;
            }
            else break;
        }
    }

    /* pop */   

    void pop()
    {
        if(!sz) return;
        
        heap[1] = heap[sz];
        sz--;

        heapify(1);
    }

    /* top */
    int top()
    {
        if(!sz) return -1e9;
        return heap[1];
    }
};
```

### Build Heap from array

Algorithm: Iterate right to left. each time, apply heapify function on `a[i]`.

```cpp
static PriorityQueue buildHeap(vector<int> arr)
{
	int n = arr.size();

	PriorityQueue pq{};

	pq.sz = n; 
	for(int i = 0; i < n; ++i) pq.heap[i + 1] = arr[i]; 

	for(int i = pq.sz / 2; i >= 1; --i)
	{
		heapify(i);
	}
	
	return pq;
}
```

This is **O(N)** amortized.

### Heapsort : Sort an array using heapsort

Step 1: Convert array to heap. For ascending order, convert to MAX HEAP. For descending order, convert to MIN HEAP.

Step 2: Iterate right to left. Each time, remove the top and heapify.

```cpp
static void heapify(int i, vector<int>& arr, int n)
{
	while(2 * i + 1 < n || 2 * i + 2 < n)
	{
		int j = i;

		if(2 * i + 1 < n && arr[2 * i + 1] > arr[j]) j = 2 * i + 1;
		if(2 * i + 2 < n && arr[2 * i + 2] > arr[j]) j = 2 * i + 2;

		if(i == j) break;

		swap(arr[i], arr[j]);

		i = j;
	}
}

static void  makeHeap(vector<int>& arr)
{
	int n = arr.size();

	for(int i = arr.size() / 2 - 1; i >= 0; --i)
	{
		heapify(i, arr, n);
	}
}

static void heapSort(vector<int>& arr)
{
	makeHeap(arr); // converts to max heap

	for(int i = (int) arr.size() - 1; i >= 0; --i)
	{
		swap(arr[0], arr[i]);
		heapify(0, arr, i);
	}
}
```

For 0 indexed heap:

children: 2i + 1 and 2i + 2
internal nodes: 0 to (n - 1)/2 -1

For 1 indexed heap:

children: 2i and 2i + 1
internal nodes: 1 to n/2


