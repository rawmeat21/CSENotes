Ordered set

```cpp
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>

using namespace __gnu_pbds;

// for descending order replace less<int> with greater<int>
typedef tree<int, null_type, less<int>, rb_tree_tag, tree_order_statistics_node_update> oset;
// usage: oset::find by order(i) <-- ith element as iterator, oset::order_of_key(x) <-- index of x (or number of elements <x, also works for elements which dont exist in oset)

```

IMP builtin

```cpp
#define msb(mask) (63-__builtin_clzll(mask*1ll)) //highest set bit
#define lsb(mask) __builtin_ctzll(mask*1ll)
#define cntsetbits(mask) __builtin_popcountll(mask*1ll)

template<typename T>
using minpq=priority_queue<T,vector<T>,greater<T>>; //min to max
template<typename T>
using maxpq=priority_queue<T>; //max to min
```

string algos

```cpp
vector<int> lps(string& s)
{
    int n = s.size();
    vector<int> pi(n);// pi[i] = longest proper prefix in [0..i] which is also a suffix of [0..i]

    for(int i = 1; i < n; ++i)
    {
        int j = pi[i - 1];

        while(j && s[i] != s[j]) j = pi[j - 1];

        if(s[i] == s[j]) j++;

        pi[i] = j;
    }

    return pi;
}

vector<int> zarr(string& s)
{
    int n = s.size();

    vector<int> z(n);// z[i] = longest prefix starting at i which is also a prefix of original string s

    for(int i = 0, l = 0, r = 0; i < n; ++i)
    {
        if(i <= r)
        {
            z[i] = min(r - i + 1, z[i - l]);    
        }

        while(i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;

        if(i + z[i] - 1 > r) l = i, r = i + z[i] - 1;
    }

    return z;
}

template<typename T>
struct rollinghash
{
    long long p = 0;
    long long mod = 0;
    vector<long long> hash{};
    vector<long long> ppow{};

    rollinghash(T& s, long long p, long long mod)
    {
        int n = s.size();

        hash.resize(n + 1);
        ppow.resize(n + 1);

        this->p = p;
        this-> mod = mod;

        ppow[0] = 1;

        for(int i = 1; i <= n; ++i) (ppow[i] = ppow[i - 1] * p) %= mod;

        build(s);
    }

    void build(T& s)
    {
        int n = s.size();

        for(int i = 0; i < n; ++i)
        {
            (hash[i + 1] = hash[i] * p + (s[i] + 1)) %= mod;
        }
    }

    long long rmd(long long a, long long b)
    {
        return ((a % b) + b) % b; 
    }

    long long get(int l, int r)
    {
        return rmd(hash[r + 1] - hash[l] * ppow[r - l + 1], mod);
    }
};
```

