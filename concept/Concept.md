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

std::cout << mymax<7, 33.4>

std::complex<double> c1, c2;
std::cout << mamax(c1, c2);  //no < supported

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