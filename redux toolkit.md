### createAsyncThunk

#### 总览

这个函数接收Redux action类型字符串和返回promise的回调函数。它根据你传入的action类型的前缀生成promise生命周期的action类型，并返回一个thunk action的创建者，该创建者将运行promise回调并根据返回的promise dispatch这个生命周期actions。

抽象了推荐的处理异步请求生命周期的标准方法。

它不生成任何reducer方法，因为它不知道你要获取什么数据，也不知道你想怎么跟踪加载状态，或者如何处理返回的数据。你应该编写自己的reducer逻辑来处理这些action。并使用适合你自己的应用程序的加载状态和处理逻辑。

用法示例：