#JetPack
## SomeThings
1. Support V7 包的27版本,集成了LifeCycle 和LifeOwner 无需自己去手动导入相关的依赖.
2. 使用android.arch.lifecycle:reactivestreams库,可以将RxJava2 与LiveData集成使用
3. tips:guide to app architecture 中的webservice直接返回一个MutableLiveData,然后异步执行请求去设置数据并无问题,因为LiveData设置数据时才会去更新界面.
## ViewModel
1. ViewModel的创建是通过ViewModelProvider.Factory接口的实例创建的,该接口的实现目前有两种:AndroidViewModelFactory和NewInstanceFactory.
继承自AndroidViewModel的ViewModel使用AndroidViewModelFactory进行创建.
继承自ViewModel的ViewModel通过NewInstanceFactory进行创建.

2. ViewModel是与组件的生命周期无关的控件,当Fragment或Activity因为Configuration变化销毁重建之后,ViewModel还是没有改变的,能立刻刷新数据用于显示.
   *?????但是实际查看源码发现ViewModel其实是存储与Fragment实例的ViewModelStore内部*
   FragmentActivity会在FragmentActivity Configuration变化时进行 ViewModelStore状态的存储
   
3. ViewModel不要直接持有View,因为ViewModel的生命周期通常比View的生命周期更加长.


## LiveData
1. LiveData在观察的时候需要传入一个LifeCycleOwner 实例便于观察组件的生命周期.

## LifeCycle
1. 27版本的Support v4包中,Fragment 以及FragmentActivity 均实现了LifeCycleOwner接口,getLifeCycle返回的均为LifecycleRegistry的实例用于管理(LiveData,ViewModel的生命周期)
   v4包中的Fragment Activity的start stop resume等生命周期均会回调给LifeCycleRegistry
