 

本文来自CSDN博客，转载请标明出处：http://blog.csdn.net/ciahi/archive/2009/08/09/4429103.aspx

# STL 的使用





条款1：仔细选择你的容器

deque是唯一一个“在迭代器失效时不会使它的指针和引用失效”的标准STL容器。

条款2：小心对“容器无关代码”的幻想

既要和序列容器又要和关联容器一起工作的代码并没有什么意义。很多成员函数只存在于其中一类容器中，比如，只有序列容器支持push_front或push_back，只有关联容器支持count和lower_bound。 在不同的类中，相同的操作名称，但在意义上是天差地别的。

条款3：使容器里对象的拷贝操作轻量而正确

拷贝对象是STL的方式。即，比如向容器中添加对象，都是通过值来传递的，即都要对对象进行拷贝。

还有其它的操作，如排序算法，删除元素等，都要进行大量的拷贝动作。（通过容器里面的对象的拷贝构造函数和拷贝赋值操作符来完成的）

如果你以基类对象建立一个容器，而你试图插入派生类对象，那么当对象（通过基类的拷贝构造函数）拷入容器的时候对象的派生部分会被删除。

一个使拷贝更高效、正确而且对分割问题免疫的简单的方式是建立指针的容器而不是对象的容器。

（但指针的容器有它们自己STL相关的头疼问题。智能指针的容器是一个吸引人的选择）



条款4：用empty来代替检查size()是否为0

对于所有的标准容器，empty是一个常数时间的操作，但对于一些list实现，size花费线性时间。

条款5：尽量使用区间成员函数代替它们的单元素兄弟

因为区间成员函数一般针对特定的容器进行了优化，要比“通用”版本的操作效率高。

无论何时你必须完全代替一个容器的内容，你就应该想到赋值。

几乎所有目标区间被插入迭代器指定的copy的使用都可以用调用的区间成员函数的来代替。(尽量用成员函数来代替copy)

● 一般来说使用区间成员函数可以输入更少的代码。

● 区间成员函数会导致代码更清晰更直接了当。

条款6：警惕C++最令人恼怒的解析

ifstream dataFile("ints.dat");

list<int> data(istream_iterator<int>(dataFile), // 警告！这完成的并不

istream_iterator<int>()); // 是像你想象的那样

编译器可能会将它解析为一个函数的声明。

改用如下代码来代替：（即在第一个参数外面加上括号）

list<int> data((istream_iterator<int>(dataFile)),

istream_iterator<int>());

不过这种方法可能并不是所有编译器都支持的。

所以改成下面的方法可以保证所有编译器支持：

istream_iterator<int> dataBegin(dataFile);

istream_iterator<int> dataEnd;

list<int> data(dataBegin, dataEnd);

条款7：当使用new得指针的容器时，记得在销毁容器前delete那些指针

当你要删除指针的容器时要避免资源泄漏，你必须用智能引用计数指针对象（比如Boost的shared_ptr）来代替指针，或者你必须在容器销毁前手动删除容器中的每个指针。

条款8：永不建立auto_ptr的容器

auto_ptr的容器（COAPs）是禁止的。试图使用它们的代码都不能编译。

主要是因为auto_ptr会传递所有权，所以再对容器操作的时候，很可能产生一些非预期的行为。

条款9：在删除选项中仔细选择

● 去除一个容器中有特定值的所有对象：

如果容器是vector、string或deque，使用erase-remove惯用法。

如果容器是list，使用list::remove。

如果容器是标准关联容器，使用它的erase成员函数。

● 去除一个容器中满足一个特定判定式的所有对象：

如果容器是vector、string或deque，使用erase-remove_if惯用法。

如果容器是list，使用list::remove_if。

如果容器是标准关联容器，使用remove_copy_if和swap，或写一个循环来遍历容器元素，当你把迭代器传给erase时记得后置递增它。

● 在循环内做某些事情（除了删除对象之外）：

如果容器是标准序列容器，写一个循环来遍历容器元素，每当调用erase时记得都用它的返回值更新你的迭代器。

如果容器是标准关联容器，写一个循环来遍历容器元素，当你把迭代器传给erase时记得后置递增它。

条款10：注意分配器的协定和约束

条款12：对STL容器线程安全性的期待现实一些

你能从已有的实现里确定的最多是下列内容：

● 多个读取者是安全的。多线程可能同时读取一个容器的内容，这将正确地执行。当然，在读取时不能有任何写入者操作这个容器。

● 对不同容器的多个写入者是安全的。多线程可以同时写不同的容器。

所以要实现线程安全，必须自己来处理代码，将要加锁部分的代码lock住。

条款13：尽量使用vector和string来代替动态分配的数组

无论何时，你发现你自己准备动态分配一个数组（也就是，企图写“new T[...]”），你应该首先考虑使用一个vector或一个string。(这样就可以避免管理资源,省去了new及delete所可能造成的资源泄漏)

如果你在多线程环境中使用引用计数字符串，就应该注意线程安全性支持所带来的的性能下降问题。

如果你用的string实现是引用计数的，而且要在多线程环境中使用，可以用如下的方法尝试：

一，看看库是否可以关闭引用计数。

二，寻找或开发一个不使用引用计数的string实现。

三，考虑使用vector<char>来代替string。

条款14：使用reserve来避免不必要的重新分配

因为vector和string空间不足时(且小于max_size)，每次会以2为因数增长。而且每次都会申请新内存，将旧数据拷贝到新内存，销毁旧内存的对象，释放旧内存。所以消耗很大。

用reserve来保存足够多的容量。如果确切的知道有多少元素，就可以使用reserve，或者保留你可能需要的最大空间。将数据全部添加完后，再修整掉多余的内容。

条款15：小心string实现的多样性

因为string的各实现不同，可能造成string特性的一些差异，如sizeof(string)就可能大小不一致。

新字符串值的建立可能需要0、1或2次动态分配。

string对象可能是或可能不共享字符串的大小和容量信息。

string可能是或可能不支持每对象配置器。

不同实现对于最小化字符缓冲区的配置器有不同策略。

条款16: 如何将vector和string的数据传给遗留的API

如果你有一个vector对象v，而你需要得到一个指向v中数据的指针，以使得它可以被当作一个数组，只要使用&v[0]就可以了。

让C风格API把数据放入一个vector，然后拷到你实际想要的STL容器中的主意总是有效的。

条款17：使用“交换技巧”来修整过剩容量

当vector擦除了很多元素之后，想要把它的大小缩减，以节省空间。

reserve和resize都没法减少vector的占用空间。只能用swap来做。

如下：

vector<Contestant>(contestants).swap(contestants);

即，用目前contestants的有效元素来初始化一个临时vector，然后再将两个vector的内容互换。并且当这个临时对象消失的时候，那个vector的所有空间都被释放了。

string(s).swap(s); // 在s上进行“收缩到合适”

vector<Contestant>().swap(v); // 清除v而且最小化它的容量

string().swap(s); // 清除s而且最小化它的容量

条款18：避免使用vector<bool>

第一，它不是一个STL容器。

第二，它并不容纳bool。

(虽然vector<bool>满足大部分STL容器的必要条件，但仍然不能完全满足。)

可以用deque<bool>来代替(deque<bool>的存储并不连续)。或者用bitset来代替。(bitset是标准库的一部分，但不是STL容器。)

bitset在编译期间固定大小，所以不支持插入和删除元素。也不支持iterator，使用一个压缩的表示法，使得它包含的每个值只占用一比特。它提供vector<bool>特有的flip成员函数及其它位集操作函数。

条款19：了解相等和等价的区别

a==b表示a和b相等。

!(a<b) && !(b<a) 即，a<b为假，且 b<a 也为假，则a和b等价。

只所以在关联容器中使用等价，没有使用相等的原因是：

如果关联容器使用相等来决定两个对象是否有相同的值，但因为关联容器要决定元素间的顺序，所以还是要有用来比较元素大小的运算符，这样多个运算符，在关联容器中容易造成混乱。

通过只使用一个比较函数并使用等价作为两个值“相等”的意义的仲裁者，标准关联容器避开了很多会由允许两个比较函数而引发的困难。

条款20：为指针的关联容器指定比较类型

在对关联容器进行默认排序时，会默认采用指针的值来做为排序对象。所以此时要自定义排序规则。

条款21: 永远让比较函数对相等的值返回false

如果对相等的值返回true的时候，则在关联容器里面用operator <= 来比较两个元素的时候，则会将两个不相等的元素判断为不相等。(因为!(a<b) && !(b<a)为true才相等)

而且对multiset及multimap等，使用equal_range来得到等价的值的范围，a==b，但并不能得到a与b等价，所以它们两个不可能都在equal_range得到的范围内。

从技术上说，用于排序关联容器的比较函数必须在它们所比较的对象上定义一个“严格的弱序化(strict weak ordering)”。任何一个定义了严格的弱序化的函数都必须在传入相同的值的两个拷贝时返回false。

条款22：避免原地修改set和multiset的键

set和multiset保持它们的元素有序，这些容器的正确行为依赖于它们保持有序。

如果要修改的话，将原有元素拷贝出来，然后修改这个拷贝值。删除容器中的原元素，再将新的拷贝元素插入到容器里面。

条款23：考虑用有序vector代替关联容器

有序的vector与关联容器相比，从算法上来看，查找速度可能不会快，但会节省空间，因为关联容器要保存一些指针。

但实际上，因为关联容器存储元素的时候，元素是分散的，就可能存储在多个内存页面上，或者是存储在虚拟内存中，所以用到元素的时候，会经常发生缺页错误，从而导致页面更频繁的换入换出，影响查找速度。

用有序vector来模拟map或multimap时，map<const K, Val> 中的K不能为const，因为要对vector进行排序的时候，要改变K的值，来达到排序的效果。(模拟map的话，因为vector存储的是pair，所以要自定义排序函数)

条款24：当关乎效率时应该在map::operator[]和map-insert之间仔细选择

map::operator[]被设计为简化“添加或更新”功能，与vecotr、string等的operator[]无关，也与内建数组没有联系。

map<int, Widget> m;

m[1] = 1.50; //使用operator[]

m.insert(map<int, Widget>::value_type(1, 1.50)); //使用insert

使用：m[1] = 1.50; 时，如果1还未在map中出现，就会插入这个元素。

当插入元素的过程中(即1还未在map中出现)，第一种方法会比第二种方法多出如下的三步：

建立临时的默认构造Widget对象。

对Widget的赋值操作。

销毁这个临时Widget对象。

当1已经在map中出现，就会更新这个值。

这个时候，第一种方法反而会更高效。原因如下：

在insert方法中出现的map<int, Widget>是一个pair对象，在这个地方就会构造和析构pair，而且还会构造Widget，所以此时效率就比operator[]低。

不过我们自己可以实现一个算法，针对不同的情况调用不同的函数来提高效率，使其总是高效的。

条款25：熟悉非标准散列容器

散列容器是关联容器。

事实上的标准名字：hash_set、hash_multiset、hash_map和hash_multimap。(目前还未标准化，但下个版本会加入标准库。不同的厂家实现原理、细节可能会不同)

 

条款26：尽量用iterator代替const_iterator，reverse_iterator和const_reverse_iterator

有些函数只接受iterator类型的参数。

const_iterator不能隐式转换成iterator，即使用变通的办法，也不通用，且不能保证高效。

从reverse_iterator转换而来的iterator在转换之后可能需要相应的调整。

还有种情况，如果const_iterator将operator==实现为了成员函数的话，则在==的左边必须是const_iterator才能编译通过。

不过，const_iterator可以防止所指元素被改变，有时候也是有用的。

条款27：用distance和advance把const_iterator转化成iterator

//ci是const_iterator，Iter i是iterator

Iter i(const_cast<Iter>(ci)); // 错误！不能从const_iterator映射为iterator

只所以错误，是因为const_iterator与iterator是完全不同的类。(如果是vector和string容器的迭代器的话，有可能可以通过编译，因为这些容器通常用真实的指针作为它们的迭代器。)

用如下的方法来转换：

Iter i(d.begin()); // 初始化i为d.begin()

advance(i, distance<ConstIter>(i, ci)); // 把i移到指向ci位置

条款28：了解如何通过reverse_iterator的base得到iterator

vector<int>::reverse_iterator ri;

vector<int>::iterator i(ri.base());

实际上得到的i是指向ri的下一个位置的元素，而并不是ri所指的元素。

这样做的好处是在插入元素的时候，并不需要考虑它们的位置，不管用i还是ri来插入元素，都会插入到相同的位置。

但是，在擦除元素的时候就会出现问题。因为erase是插除迭代器本身所指的元素，所以i和ri会擦除不同的元素。

解决方法如下：

v.erase((++ri).base()); // 删除ri指向的元素；

条款29：需要一个一个字符输入时考虑使用istreambuf_iterator

istream_iterators所依靠的operator>>函数进行的是格式化输入，这意味着每次你调用的时候它们都必须做大量工作。如它们必须检查影响它们行为的流标志，读取错误检查等等。所以它的速度相对比较慢。

你可以像istream_iterator一样使用istreambuf_iterator，但istream_iterator<char>对象使用operator>>来从输入流中读取单个字符。，istreambuf_iterator不忽略任何字符。它们只简单地抓取流缓冲区的下一个字符。

条款30：确保目标区间足够大

如果目标区间并不足够大，用back_inserter或front_inserter或inserter来插入。

赋值只在两个对象之间操作时有意义，而不是在一个对象和一块原始的比特之间。

results.reserve(results.size() + values.size());

transform(values.begin(), values.end(),

results.end(), // 到未初始化的内存

transmogrify); // 行为未定义！

因为reserve出来的空间只是原始内存，里面并不包含对象。

条款31：了解你的排序选择

partial_sort只排序前N个元素。

nth_element本质上等价于partial_sort，但是它并不在这前N个元素内部排序。但前N个是所有元素里面N个最大的。 它也可以将指定位置的单一元素设成与相应位置相符的大小。

partition排序指定范围内的元素。

条款32：如果你真的想删除东西的话就在类似remove的算法后接上erase

唯一从容器中除去一个元素的方法是在那个容器上调用一个成员函数，而且因为remove无法知道它正在操作的容器，所以remove不可能从一个容器中除去元素。从一个容器中remove元素不会改变容器中元素的个数，它只是将相应位置后面的元素向前移，而且不删除最后面的元素。 用size()取得的大小也不会变化。

所以要想删除元素，调用完remove后一定要调用erase。

v.erase(remove(v.begin(), v.end(), 99), v.end()); // 真的删除所有等于99的元素。

(不过list的成员函数remove可以真正的删除元素。)

条款33：提防在指针的容器上使用类似remove的算法

容器存储的是指针的时候，调用remove的时候，会使一些内存空间不被指针所指向，从而无法再引用这段内存空间，从而造成内存泄漏。

如果你无法避免在那样的容器上使用remove，排除这个问题一种方法是在应用erase-remove惯用法之前先删除指针并设置它们为空，然后除去容器中的所有空指针。

条款34：注意哪个算法需要有序区间

binary_search

lower_bound

upper_bound

equal_range

set_union

set_intersection

set_difference

set_symmetric_difference

merge

inplace_merge

includes

要注意容器的排序方式与算法的排序方式是否一致，如果不一致，可能导致一些未定义的行为。

条款35：通过mismatch或lexicographical比较实现简单的忽略大小写字符串比较

mismatch，用来返回两个字符串第一个不匹配的位置。只要为它指定比较规则即可，第一个区间要小于第二个区间的长度。

lexicographical_compare用来进行字典排序。

条款36：了解copy_if的正确实现

条款37：用accumulate或for_each来统计区间

条款38：把仿函数类设计为用于值传递

c/c++准则：函数指针是值传递。

STL算法库一般要按值传递来接收函数对象。不过可以通过显示指定算法的模板参数类型，来传递引用。不过传递引用时，有时候会发生编译错误。

用值传递函数对象时，函数对象要尽可能的小，否则它们的拷贝会很昂贵。函数对象必须是单态的，不能用虚函数。因为派生类以值传递的时候可能造成切割问题。

如果要传递的函数对象很大，或者要使用到多态，用Bridge模式可以解决。 实现方式即：

带着你要放进你的仿函数类的数据和/或多态，把它们移到另一个类中（即实现类）。然后给你的仿函数一个指向这个新类的指针。建立一个包含一个指向实现类的指针的小而单态的类，然后把所有数据和虚函数放到实现类。

条款39：用纯函数做判断式

判断式是返回bool（或者其他可以隐式转化为bool的东西）。

纯函数是返回值只依赖于参数的函数。且它的内部不会改变参数的值，由纯函数引用的所有数据不是作为参数传进的就是在函数生存期内是常量。所以纯函数没有状态。

一个判断式类是一个仿函数类，它的operator()函数是一个判断式，也就是，它的operator()返回true或false（或其他可以隐式转换到true或false的东西）。

如果在判断式中改变了所引用的数据，即不是纯函数了，这时候，以值来传递判断式的时候，可能会出现问题，比如说判断式本应该只有一次返回true，但两次使用这个判断式时，两次被重新构造，有两次返回true了。结果就会有问题了。

最简单的使你自己不摔跟头而进入语言陷阱的方法是在判断式类中把你的operator()函数声明为const。这样，编译器不会让你在里面改变任何类数据成员。

不管怎么写判断式，他们都应该是纯函数。

条款40：使仿函数类可适配

提供必要的typedef的函数对象称为可适配的。(typedef是：argument_type、first_argument_type、second_argument_type和result_type)

ptr_fun做的唯一的事是使一些typedef有效。

只要使我们的类继承自std::unary_function或是std::binary_function模板类，就可以自动使这个函数对象变为可适配的。

在仿函数类中，只能提供一个operator()来适配一元或二元的仿函数，如果提供两个，它只能适配一个！

条款41：了解使用ptr_fun、mem_fun和mem_fun_ref的原因

这些函数的主要任务之一是覆盖C++固有的语法矛盾之一。

它们被称为函数对象适配器。

在将成员函数传递给STL组件时，一定得使用mem_fun和mem_fun_ref，否则不能通过编译。

ptr_fun是在函数对象没有提供一些typedef时，要使用的。(参看条款40)

条款42：确定less<T>表示operator<

条款43：尽量用算法调用代替手写循环

调用算法通常比手写的循环更优越。原因：

● 效率：算法通常比程序员产生的循环更高效。

● 正确性：写循环时比调用算法更容易产生错误。

● 可维护性：算法通常使代码比相应的显式循环更干净、更直观。

条款44：尽量用成员函数代替同名的算法

首先，成员函数更快。其次，比起算法来，它们与容器结合得更好

因为成员函数就是针对某容器提供的特化版本算法。

list成员函数的行为和它们的算法兄弟的行为经常不相同。如调用通用的remove、remove_if和unique算法后，必须紧接着调用erase函数才能清除对象，但list的remove等成员函数后面并不需要erase函数，就能清除对象。

条款45：注意count、find、binary_search、lower_bound、upper_bound和equal_range的区别

如果迭代器定义了一个有序区间，则可以通过binary_search、lower_bound、upper_bound和equal_range来加速。如果没有有序区间，则只能用count、find线性时间的算法。count经常用来检查元素是否存在。但count找到存在的元素后，仍会继续查找，直到将所有元素遍历。但find找到后就会结束，所以find的效率可能略优一点。

要测试在有序区间中是否存在一个值，使用binary_search，但这个算法只返回一个bool值，表明是否找到了，要想得到更多的信息，则这个算法无能为力了。如果想知道存在的元素在哪儿，可以用equal_range。

条款46：考虑使用函数对象代替函数作算法的参数

函数作为算法的参数的时候，会传递函数的指针作为参数，而且函数指针会抑制内联。函数对象的operator()被声明为内联的时候，在速度上，会高于以函数作为算法的参数进行传递。

有时候，用函数作为参数的时候，可能会编译不过，但用函数对象，一般都不会出问题。

条款47：避免产生只写代码

此处的“只写”代码是指，容易写，但是不容易读和理解的代码。

代码的读比写更经常，这是软件工程的真理。

条款48：总是#include适当的头文件

 


条款49：学习破解有关STL的编译器诊断信息

 


条款50：让你自己熟悉有关STL的网站

