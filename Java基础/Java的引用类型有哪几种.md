# Java的引用类型有哪几种

强引用，软引用，弱引用，虚引用

## 强引用
**通过对象类型的变量直接引用**,例如`Object o1=new Object()`,</br>
o1为强引用，引用堆内存中Object对象</br>
o1这个变量是存在于栈的(`方法里面的局部变量都是存在于栈`),Object类里的属性都是存在于堆里(`对象的属性都在堆里`)</br>
缓存满了GC会回收对象，而`这类强引用对象无法无法被回收，最终会被放到幸存区或者老年代`。
只要这个强引用没被释放，那么这个对象就不会被回收。

## 软引用(`SoftReference<T>`)
用SoftReference包裹的对象，如果一个对象只存在一个软引用，在内存溢出之前，将会被回收。</br>
软引用主要用于实现内存敏感的高速缓存。当内存不够用时，sf对象会被回收，sf.get()会返回null。</br>
`内存不足时,软引用引用的对象会被回收，不会幸存或放到老年代`

## 弱引用(`(WeakReference<T>`)
用WeakReference包裹的对象，如果一个对象只存在一个弱引用，在垃圾回收时这个对象会被回收。</br>
`只要有GC,弱引用引用的对象就会被回收`</br>
>WeakHashMap就是用的WeakReference来保存对象。WeakReference的Entry继承了WeakReference类。

## 虚引用(`PhantomReference<T>`)
Phantomreference,垃圾回收时回收，永远无法通过引用取到对象值。</br>
pf.get();//永远返回null</br>
pf.isEnQueued();//返回对象是否已被回收</br>
`虚引用主要用于检测对象是否已经从内存中删除`，可以通过这个通知机制来做额外的清场工作。 因此有些情况可以用PhantomReference 代替finalize()，做资源释放更明智。</br>
`当对象被回收时,可以得到通知记录被回收的对象`.