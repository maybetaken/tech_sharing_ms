```c++
template <typename T>
T mymax(T a, T b){
    return b < a ? a : b;
}
```
**Implicit requirements for T:**
* operator < (returning bool)
* copy/move constructor

```c++
int i1 = 42, i2 = 77;
std::cout << mymax(i1, i2);

std::cout << mymax(7, 33.4);

std::complex<double> c1, c2;
std::cout << mymax(c1, c2);  //no < supported

std::atomic<int> a1{8}, a2{10};
std::cout << mymax(a1, a2); // copying disabled
```

## **Concepts(since c++20)**
 ***To formulate formal constraints for generic code***

 ```c++
template <typename T>
requires std::copyable<T> && SupportsLessThan<T>
T mymax(T a, T b){
    return b < a ? a : b;
}
 ```
Here is the SupportsLessThan concept:

```c++
template <typename T>
concept SupportsLessThan = requires (T x) {x < x ;} ;
```

But how can we implement this in c++ 17(maybe lower version)
```c++
template <typename T>
std::enable_if_t<std::copyable<T> && SupportsLessThan_v<T>, T> mymax(T a, T b){
    return b < a ? a : b;
}
```

We have to implement the SupportsLessThan_v ourself like:
```c++
template <typename T, typename = void>
struct SupportsLessThan : std::false_type {}

template <typename T>
struct SupportsLessThan<T, std::void_t<decltype(std::declval<T>() < std::declval<T>()>)>> : std::true_type{}

template <typename T>
constexpr bool SupportsLessThan_v = SupportsLessThan<T>::value;
```

We have another complicated implementation for this
```c++
template <typename T>
struct SupportsLessThan{
private:
    template <typename, typename=decltype(std::declval<T>()<std::declval<T>())>
    static char test(void*);

    template <typename>
    static long test(...);
public:
    static constexpr bool value = is_same<decltype(test<T>(nullptr)),char)>::value;
}
template <typename T>
constexpr bool SupportsLessThan_v SupportsLessThan<T>::value;
```

Assume that we have an function add, it adds values to a collection

```c++
template <typename Coll, typename T>
void add(Coll& coll, const T& val){
    coll.push_back(val);
}

// for the below caller, it works
std::vector<int> coll;
add(coll, 42); // OK
```

For c++ 20, you can use auto as function parameters, which is still a function template
```c++
void add(auto& coll, const auto& val){
    coll.push_back(val)
}

// for the below caller, it works also
std::vector<int> coll;
add(coll, 42); // OK
```

But here in other cases, we have another function
```c++
void add(auto& coll, const auto& val){
    coll.push_back(val);
}

void add(auto& coll, const auto& val){ // compile time error
    coll.insert(val);
}

// for the below caller, it works also
std::vector<int> coll1;
std::set<int> coll2;
add(coll1, 42); // ERROR: ambiguous
add(coll2, 42); // ERROR: ambiguous
```

emm... how can we fix it..

We can use **Concepts** as type constraints 

```c++
void add(HasPushBack auto& coll, const auto& val){ // this function is still a template
    coll.push_back(val);
}

/*
This is also equivalent to:
template<HasPushBack T1, typename T2>
void add(T1& coll, const T2& val){
    coll.push_back(val);
}
*/

void add(auto& coll, const auto& val){ // compile time error
    coll.insert(val);
}

// for the below caller, it works also
std::vector<int> coll1;
std::set<int> coll2;
add(coll1, 42); // OK, use first add()
add(coll2, 42); // OK, use second add()
```
***Overload resolution perfers more specialized template***

how can we implement HasPushBack in template,
```c++
template <typename Coll>
concept HasPushBack = requires (Coll c, Coll::value_type v){
    c.push_back(v);
}
```
Also it can be implemented with requires and Compile-time if

```c++
void add(auto& coll, const auto& val){
    if constexpr (requires {coll.push_back(val);}){
        coll.push_back(val);
    }else{
        coll.insert(val);
    }
}
```
***A requirement is a boolean expression***
