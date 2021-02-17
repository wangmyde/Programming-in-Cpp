### 1. `static_assert` and `decltype`   
`static_assert`: `static_assert(std::is_same<vector<int>::iterator ,decltype(it)>::value, "the type is wrong");`
`std::is_same<vector<int>::iterator ,decltype(it)>::value` is true, print out nothing. Otherwise, `the type is wrong`
`std::is_same<vector<int>::iterator ,decltype(it)>::value`: `std::is_same` decides whether 'vector<int>::iterator' and 'decltype(it)' are the same type. If yes, return Ture.  
`delctype(it)`: inspects the declared type of an entity or the type and value category of an expression.
```
int i = 33;
decltype(i) j = i * 2;
```
```
#include <iostream>
#include <type_traits>
#include <vector>
#include <typeinfo>
 
using std::cout;
using std::endl;
using std::vector;
 
// 1. placeholder type specifiers
 
template<class T, class U>
auto add(T t, U u) { return t + u; }
 
int main()
{
 
  auto res = add(2,3.0);
 
  cout << res << " has type " << typeid(res).name() << endl;
  static_assert(std::is_same<vector<int>::iterator ,decltype(it)>::value, "the type is wrong");
 
 
  return 0;
}
```
### 2. lambda
```
#include <iostream>
#include <type_traits>
#include <vector>
#include <typeinfo>

using std::cout;
using std::endl;
using std::vector;

int main()
{
    auto lambda1 = [](int x) { return 3 + x; };
    auto lambda2 = [](int x, int y) { return y + x; };
    cout << lambda1(3) << endl;
    cout << lambda2(3, 4) << endl;
    cout << "lambda1 has type: " << typeid(lambda1).name() << endl;
    cout << "lambda1 has type: " << typeid(lambda2).name() << endl;
    return 0;
}
 ```
 ```
 6
7
lambda1 has type: class <lambda_ef51e37f546878df1457b181f731e21d>
lambda1 has type: class <lambda_644be8c902fa9accc1260fbc4d95cba4>
```
```
#include <iostream>
#include <type_traits>
#include <vector>
#include <typeinfo>

using std::cout;
using std::endl;
using std::vector;

template<class T, class U>
auto add(T t, U u) { return t + u; }

int main()
{
    auto res = add(2.0, 3);
    cout << "res has type: " << typeid(res).name() << endl;
  
    return 0;
}
```
```
res has type: double
```
### 3. initialization
```
#include <iostream>
#include <initializer_list>
#include <vector>
#include <typeinfo>

using namespace std;

int main()
{
	int i = 1; // copy initialization
	/***
	int i(1); // direct initialization
	int i{ 1 }; // list initialization
	int i{}; // value initialization
	cout << i << endl;
	***/
	return 0;
}
```
```
#include <iostream>
#include <initializer_list>
#include <vector>
#include <typeinfo>

using namespace std;

struct X {
	X(int i) { cout << "X(int i)" << endl; }
	X(X&) { cout << "X(X&)" << endl; }
	X(initializer_list<int>) { cout << "X(initializer_list<int>)" << endl; }
	X& operator=(X) { cout << "X& operator=(X)" << endl; return *this; }
};

int main()
{
	cout << "x1(1): " << endl;
	X x1(1);
	cout << "x2{ 1 }: " << endl;
	X x2{ 1 };
	cout << "x3{1,2,3}: " << endl;
	X x3{ 1,2,3 };
	cout << "x4(x2): " << endl;
	X x4(x2);
	cout << "x5 = x1: " << endl;
	X x5 = x1;
	cout << "x2 = x1: " << endl;
	x2 = x1;
	cout << "x6({ 1,2,3 }): " << endl;
	X x6({ 1,2,3 });
	return 0;
}
```
```
x1(1):
X(int i)
x2{ 1 }:
X(initializer_list<int>)
x3{1,2,3}:
X(initializer_list<int>)
x4(x2):
X(X&)
x5 = x1:
X(X&)
x2 = x1:
X(X&)
X& operator=(X)
x6({ 1,2,3 }):
X(initializer_list<int>)
```
```
#include <iostream>
#include <initializer_list>
#include <vector>
#include <typeinfo>

using namespace std;

template <class T>
struct S {
	std::vector<T> v;
	S(std::initializer_list<T> l) : v(l) {
		std::cout << "constructed with a " << l.size() << "-element list\n";
	}
	void append(std::initializer_list<T> l) {
		v.insert(v.end(), l.begin(), l.end()); // // insert: (iterator position, InputIterator first, InputIterator last);
	}
	std::pair<const T*, std::size_t> c_arr() const {
		return { &v[0], v.size() };
	}

};
int main()
{
	S<int> s = { 1,2,3,4,5 }; // copy initialization
	s.append({ 6,7,8 }); // list-initialization in func call
	cout << "The vector size is now " << s.c_arr().second << " ints:\n";
	copy(s.v.begin(), s.v.end(), ostream_iterator<int>(cout, " "));
	cout << endl;

	for (int x : {-1, -2, -3})
		cout << x << ' ';
	cout << '\n';
	auto al = { 10, 11, 12 };
	cout << "al type: " << typeid(al).name() << '\n'; // al is not S class
	cout << "The list bound to auto has size() = " << al.size() << '\n';
	return 0;
}
```
```
constructed with a 5-element list
The vector size is now 8 ints:
1 2 3 4 5 6 7 8
-1 -2 -3
al type: class std::initializer_list<int>
The list bound to auto has size() = 3
```
```
#include <iostream>
#include <initializer_list>
#include <vector>
#include <typeinfo>

using namespace std;


int main()
{
	auto test = { 1,2,3,4 };
	cout << typeid(test).name() << '\n';
	return 0;
}
```
```
class std::initializer_list<int>
```
### 3. Lambda: 被捕获的变量的值是在创建时被copy，而不是调用时
```
int main()
{
	auto f = [] (int i) -> int {return i; };

	return 0;
}
```
```
#include <algorithm>
using std::sort; using std::for_each;
#include <functional>
using namespace std::placeholders;
#include <string>
using std::string;
#include <vector>
using std::vector;
#include <iostream>
using std::cin; using std::cout; using std::endl;

void fcn1()
{
	size_t v1 = 42;  // local variable
					 // copies v1 into the callable object named f
	auto f = [v1] { return v1; }; // v1在f创建时已经被copy了，故j为42
	v1 = 0;
	auto j = f(); // j is 42; f stored a copy of v1 when we created it
	cout << j << endl;
}

void fcn2()
{
	size_t v1 = 42;  // local variable
					 // the object f2 contains a reference to v1 
	auto f2 = [&v1] { return v1; };
	v1 = 0;
	auto j = f2(); // j is 0; f2 refers to v1; it doesn't store it
	cout << j << endl;
}

void fcn3()
{
	size_t v1 = 42;  // local variable
					 // f can change the value of the variables it captures
	auto f = [v1]() mutable { return ++v1; };
	v1 = 0;
	auto j = f(); // j is 43
	cout << j << endl;
}

void fcn4()
{
	size_t v1 = 42;  // local variable
					 // v1 is a reference to a nonconst variable
					 // we can change that variable through the reference inside f2
	auto f2 = [&v1] { return ++v1; };
	v1 = 0;
	auto j = f2(); // j is 1
	cout << j << endl;
}

void fcn5()
{
	size_t v1 = 42;
	// p is a const pointer to v1
	size_t* const p = &v1;
	// increments v1, the objet to which p points
	auto f = [p]() { return ++ * p; };
	auto j = f();  // returns incremented value of *p
	cout << v1 << " " << j << endl; // prints 43 43
	v1 = 0;
	j = f();       // returns incremented value of *p
	cout << v1 << " " << j << endl; // prints 1 1
}

int main()
{
	fcn1();
	fcn2();
	fcn3();
	fcn4();
	fcn5();

	return 0;
}
```
```
#include <algorithm>
using std::sort; using std::for_each;
#include <functional>
using std::bind;
using namespace std::placeholders;
#include <string>
using std::string;
#include <vector>
using std::vector;
#include <iostream>
using std::cin; using std::cout; using std::endl;

void print(const vector<string>& words)
{
    // uses a lambda function to print elements
    for_each(words.begin(), words.end(),
        [](const string& s) { cout << s << " "; });
    cout << endl;
}

int main()
{
    vector<string> words;

    // copy contents of each book into a single vector
    string next_word;
    while (cin >> next_word) {
        // insert next book's contents at end of words
        words.push_back(next_word);
    }
    print(words);

    // uses a lambda function to compare elements
    sort(words.begin(), words.end(), [](const string& s1, const string& s2) {return s1 < s2; });
    print(words);

    return 0;
}
```
