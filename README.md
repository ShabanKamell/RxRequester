# RxRequester
A simple wrapper for RxJava that helps you:
- [ ] Make clean RxJava requests
- [ ] Handle errors in a clean and effective way.

RxRequester does all the dirty work for you!

### Before RxRequester
```kotlin
dm.restaurantsRepo.all()
                .doOnSubscribe { showLoading() }
                .subscribeOn(Schedulers.io())
                .observeOn(Schedulers.io())
                .doOnNext { hideLoading() }
                .subscribe( {

                }, { error ->

                })
```

### After RxRequester
```kotlin
requester.request { dm.restaurantsRepo.all() }.subscribe { }
```

### Usage
#### Setup

``` kotlin
val presentable = object: Presentable {
            override fun showError(error: String) { showError.value = error }
            override fun showError(error: Int) { showErrorRes.value = error }
            override fun showLoading() { toggleLoading.value = true }
            override fun hideLoading() { toggleLoading.value = false }
            override fun onHandleErrorFailed() { showErrorRes.value = R.string.oops_something_went_wrong }
        }

       val requester = RxRequester.create(ErrorContract::class.java, presentable)
```
#### Handle Errors
```
            RxRequester.nonHttpHandlers = listOf(
                    IoExceptionHandler(),
                    NoSuchElementHandler(),
                    OutOfMemoryErrorHandler()
            )
            RxRequester.httpHandlers = listOf(
                    TokenExpiredHandler(),
                    ServerErrorHandler()
            )
```

#### Error Handlers
Error handler is a class that extends
`HttpExceptionHandler`
```kotlin
class ServerErrorHandler : HttpExceptionHandler() {

    override fun supportedExceptions(): List<Int> {
        return listOf(500)
    }

    override fun handle(info: HttpExceptionInfo) {
        info.presentable.showError(R.string.oops_something_went_wrong)
    }
}
```

Or `NonHttpExceptionHandler`
```kotin
class OutOfMemoryErrorHandler : NonHttpExceptionHandler<OutOfMemoryError>() {

    override fun supportedThrowables(): List<Class<OutOfMemoryError>> {
        return listOf(OutOfMemoryError::class.java)
    }

    override fun handle(info: NonHttpExceptionInfo) {
        info.presentable.showError(R.string.no_memory_free_up_space)
    }
}
```

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
