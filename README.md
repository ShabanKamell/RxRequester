RxReqeuster
==============

<img src="https://github.com/ShabanKamell/RxRequester/blob/master/blob/raw/logo.png" height="200">

A wrapper for RxJava abstracts away all those nasty details you don't really care about in most requests and provides default configurations for schedulers, loading indicators, and error handling. 

Using **RxRequester** you can:
- [ ] Make clean RxJava requests.
- [ ] Inline & Global error handling.
- [ ] Toggle loading indicators easily.

### Before RxRequester

``` kotlin
restaurantsRepo.all()
                .doOnSubscribe { showLoading() }
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.main())
                .doOnNext { hideLoading() }
                .subscribe( {

                }, { error ->

                })
```

### After RxRequester

``` kotlin
requester.request { restaurantsRepo.all() }.subscribe { }
```

#### Gradle:
```groovy
allprojects {
    repositories {
        ...
        maven { url "https://jitpack.io" }
    }
}

dependencies {
        implementation 'com.github.ShabanKamell:RxRequester:x.y.z'
}

```
(Please replace x, y and z with the latest version numbers:  [![](https://jitpack.io/v/ShabanKamell/RxRequester.svg)](https://jitpack.io/#ShabanKamell/RxRequester))

## Setup

``` kotlin
val presentable = object: Presentable {
    override fun showError(error: String) { .. }
    override fun showError(error: Int) { .. }
    override fun showLoading() { .. }
    override fun hideLoading() { .. }
    override fun onHandleErrorFailed(throwable: Throwable) { .. }
}

RxRequester.create(presentable) {
  resumableHandlers = listOf(TokenExpiredHandler())
  httpHandlers = listOf(ServerErrorHandler())
  throwableHandlers = listOf(IoExceptionHandler(), NoSuchElementHandler(), OutOfMemoryErrorHandler())
  serverErrorContract = ErrorContract::class.java
}

// Or provide error handlers this way:
 RxRequester.resumableHandlers = listOf(TokenExpiredHandler())
 RxRequester.httpHandlers =      listOf(ServerErrorHandler())
 RxRequester.throwableHandlers = listOf(OutOfMemoryErrorHandler())
```

## Error Handling
There're 3 types of error handlers in the library

### 1- Resumable Handler
There're cases where you want to handle the error and resume the current request as normal. Resumable handler provides the easiest solution for this problem!
Imagine you received `401 token expired` error and you want to refresh the token then resume the original request. This can be done as easy as like this!

```kotlin
class TokenExpiredHandler: ResumableHandler() {
     // check if the error code is 401
    override fun canHandle(info: ThrowableInfo): Boolean {
        return (throwable as? HttpException)?.errorCode() == 401
    }
    // retrun the API that refreshes the token
    override fun handle(throwable: Throwable, presentable: Presentable): Flowable<Any> {
        return info.requester.request{ ServiceApi.refreshToken() }
    }
}
```
Of course you can apply this for any error you want.

### 2- Retrofit Http Handler
Handles Retrofit's HttpException

``` kotlin
class ServerErrorHandler: HttpExceptionHandler() {

    override fun supportedErrors(): List<Int> {
        return listOf(500)
    }

    override fun handle(throwable: Throwable, presentable: Presentable, errorCode: Int, errorBody: String) {
        presentable.showError(R.string.oops_something_went_wrong)
    }
}
```

### 3- Throwable Handler
handles generic Throwables

``` kotin
class OutOfMemoryErrorHandler: ThrowableHandler<OutOfMemoryError>() {

    override fun supportedErrors(): List<Class<OutOfMemoryError>> {
        return listOf(OutOfMemoryError::class.java)
    }

    override fun handle(throwable: Throwable, presentable: Presentable) {
        presentable.showError(R.string.no_memory_free_up_space)
    }
}
```

## Error Handlers Priority
The library handles errors according to this priority
##### 1- Resumable Handlers
##### 2- HTTP Handlers
##### 3- Throwable Handlers

The library first asks Resumable handlers to handle the error, if can't handle it will be passed to HTTP handlers, if can't handle, the error will be passed to Throwable hanldlers. If no handler can handle the error, it will be passed to `Presentable.onHandleErrorFailed(Throwable)`

## Server Error Contract
RxRequester optionally parses server error for you and shows the error automatically. Just implement `ErrorMessage`
interface in your server error model and return the error message.

``` kotlin
data class ErrorContract(private val message: String): ErrorMessage {
    override fun errorMessage(): String {
        return message
    }
}
// Pass the contract
val requester = RxRequester.create(ErrorContract::class.java, presentable)
```

## Customizing Requests
RxRequester gives you the full controll over any request
- [ ] Inline error handling
- [ ] Enable/Disable loading indicators
- [ ] Set subscribeOn Scheduler
- [ ] Set observeOn Scheduler

``` kotlin
 val requestInfo = RequestOptions.Builder()
         .inlineErrorHandling { false }
         .showLoading(true)
         .subscribeOnScheduler(Schedulers.io())
         .observeOnScheduler(AndroidSchedulers.mainThread())
         .build()

 requester.request(requestOptions) { dm.restaurantsRepo.all() }
```

Here're all request options and default values

|          **Option**          |         **Type**          |          **Default**           |
| ---------------------------- | --------------------------|------------------------------- |
| **inlineHandling**           | ((Throwable) -> Boolean)? | null                           |
| **showLoading**              | Boolean                   | true                           |
| **subscribeOnScheduler**     | Scheduler                 | Schedulers.io()                |
| **observeOnScheduler**       | Scheduler                 | AndroidSchedulers.mainThread() |

### Best Practices
- [ ] Setup `RxRequester` only once in `BaseViewModel` and reuse in the whole app.
- [ ] Initialize error handlers only once. You can initialize then Application class as they are static properties.

#### Look at 'sample' module for the full code. For more advanced example, [Restaurants Modular Architecture](https://github.com/ShabanKamell/Restaurants)

### License

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
