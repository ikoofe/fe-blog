---
title: JavaScript 代码整洁之道-异常处理篇
date: 2021-05-08 12:16:11
tags: JavaScript
---


> 本文源于翻译 [Clean Code Applied to JavaScript — Part V. Exceptions](https://dev.to/carlillo/clean-code-applied-to-javascript-part-v-exceptions-1k9d)


在软件开发中，异常处理是高质量不可或缺的一部分，这样我们才能对程序中一些意外的情况和未实现的逻辑进行有效的控制。但是，开发人员有时会将异常处理和软件的流程处理混为一谈。

异常应该用来处理软件中不受控制或未开发的情况，而不能像业务逻辑那样进行 `return`，然后在一个业务的分支流程去处理这种情况。

在这篇文章中，我们将提供一些与异常处理有关的建议，从而提高代码质量。

## 使用异常而非返回码

如果编程语言支持异常处理，那么建议优先使用异常处理。这个道理看起来十分简单，但是事实却并非如此。因为，有些开发语言可能不支持异常处理；另外，有些开发者则没有意识到异常处理所带来的好处，从而忽略这个功能，并不去使用它。然而，与返回码检查错误的方式相比，使用异常处理错误会更加简洁高效。

在下面的第一段代码中定义了一个类，然后在它实现的方法中，用 `if` 语句来检查返回值中是否存在不合法的返回码。这样做的问题在于调用者在接收函数的返回值之后要立即做错误检查，错误检查会使调用者的代码变得复杂，另外如果这个错误检查被遗忘会导致代码出现问题。我们应该把这些乏味和复杂的工作交给语言本身的异常处理。在第二段代码中，使用了异常处理隔离了两个不同的业务逻辑。这个代码有三个优势：

1. 隔离业务逻辑和错误处理，他们是两个不同的问题，必须要分开去处理和对待
2. 降低了代码的冗余程度更低，易于去阅读
3. 处理程序异常的责任交给编程语言处理

```JavaScript
// Dirty
class Laptop {
  sendShutDown() {
    const deviceID = getID(DEVICE_LAPTOP);
    if (deviceID !== DEVICE_STATUS.INVALID) {
      const laptop = DB.findOne(deviceID);

      if (laptop.getStatus() !== DEVICE_SUSPENDED) {
        pauseDevice(deviceID);
        clearDeviceWorkQueue(deviceID);
        closeDevice(deviceID);
      } else {
        logger.log('Device suspended. Unable to shut down');
      }
    } else {
      logger.log('Invalid handle for: ' + DEVICE_LAPTOP.toString());
    }
  }
}
```

```JavaScript
// Clean
/* 
   The code is better because the algorithm 
   and error handling, are now separated. 
*/
class Laptop {
  sendShutDown() {
    try {
      tryToShutDown();
    } catch (error) {
      logger.log(error);
    }
  }
  tryToShutDown() {
    const deviceID = getID(DEVICE_LAPTOP);
    const laptop = DB.findOne(deviceID);
    pauseDevice(deviceID);
    clearDeviceWorkQueue(deviceID);
    closeDevice(deviceID);
  }
  getID(deviceID) {
    ...
    throw new DeviceShutDownError('Invalid handle for: ' + deviceID.toString());
    ...
  }
}
```

## 不要忽视异常处理

**请不要做鸵鸟**，鸵鸟在遇到危险的时候会将头埋在地下。在进行异常处理的时候，我们不能像鸵鸟那样，在每次捕获到错误时都假装什么都没发生。对捕获的错误不做任何处理是没有意义的。

但是，如果我们仅仅通过 `console.log` 或者 `system.out.println` 作为错误处理，也同样意味着什么都没有做。在实践中眼睁睁的看着异常发生，而不采取任何措施去处理，这样做是非常危险的。因为这些异常通常是由意外情况引起的，从中能发现许多不易发现的问题。因此，请不要忽略对异常的处理。

> 译者注：`try ... catch` 能够让 JavaScript 代码出现问题也能顺利执行下去，避免页面因为报错导致白屏，但是对捕获到异常不做处理也是很可怕的一件事情。
> 在实践中，页面通常会接入错误上报工具（例如 Sentry），一旦页面的 JavaScript 代码出现错误，错误信息会第一时间上报到开发人员，开发人员可及时处理这些问题。一旦在代码中做了异常捕获，上报工具将不能主动捕捉到错误。如果这时不去处理捕获到的异常，会导致代码的错误逻辑不能及时被发现，因此哪怕是只对异常做日志上报处理也是好的。

在第一个代码中，这是初级开发者经常做的事情，对异常没有任何有效的处理。在第二个例子，在捕获错误后进行了必要的处理，虽然需要花费时间和精力，但这么做是值得的。

```JavaScript
try {
  functionThatMightThrow();
} catch (error) {
  console.log(error);
}
```

```JavaScript
try {
  functionThatMightThrow();
} catch (error){
  console.error(error);
  notifyUserOfError(error);
  reportErrorToService(error);
}
```

## 不要忽视 Promise reject

在 JavaScript 中，需要特别注意异步处理工具 `Promise`，同样也不能忽视 Promise 通过 reject 抛出的异常逻辑。

在这个例子中，我们能看到一个和之前差不多的例子，但它使用了 Promise。

```JavaScript
getData() 
 .then(data => functionThatMightThrow(data))
 .catch(error => console.log);
```

```JavaScript
getData()
 .then(data => functionThatMightThrow(data))
 .catch(error => {
   console.log(error);
   notifyUserOfError(error);
   reportErrorToService(error);
 });
```

## 定义异常层次结构（Exceptions Hierarchy）

每种编程语言都定义了一些基础的异常，比如 `NullPointerException` 或 `ArrayIndexOutOfBoundsException`，这些异常并不涉及我们的业务逻辑。使用这些异常来控制代码中的错误是毫无意义的，因为我们的代码是对业务逻辑的建模。因此，我们必须创建自己的异常层次结构，这些异常涉及到我们的业务逻辑，并在业务逻辑中发生意外情况时触发。

在下面的例子中，两个异常类型被创建，一个是用户异常 (UserException)，一个是管理员异常 (AdminException)，这两种异常发生在两种类型的用户上，而不是发生在数据结构上。现在，我们定义了代码的业务逻辑。

事实上，这两个异常的定义还是太过于笼统，我们可以这样定义：UserRepeatException, UserNotFoundException ...

我们需要对异常进行语义化，否则就算捕获了错误，也很难去分析。

```JavaScript
export class UserException extends Error {
  constructor(message) {
    super(`User: ${mesage}`);
   }
}

export class AdminException extends Error {
  constructor(message) {
    super(`Admin: ${message}`);
  }
}

// Client code
const id = 1;
const user = this.users.find({ id });
if(user){
 throw new UserException('This user already exists');
}
```

## 提供异常的上下文

在异常发生的时候，虽然我们可以通过堆栈跟踪和链式调用去查看发生错误的情况，但这样分析起来依然十分困难。因此，给异常情况添加备注和信息是十分必要的。比如，在捕获异常的地方，去添加一条信息去解释我们的意图。但请不要用特别复杂的语言去解释。需要注意的是，我们提供的信息最好不要被最终使用用户所能看到。因此，我们应该适当的去管理这些异常信息，让这些信息不会显示在用户的界面中，这样会更好。

如果我们定义了异常层次结构，我们还要为异常提供上下文。

## 总结

在这篇文章中，我们提出来一些异常处理的建议。异常处理是高质量软件开发中的一个基本部分，但是在许多情况下，它们会被忽略，或者是不正确的使用，只是保证代码流程不出错，去重定向到正确的程序流中。

无数例子告诉我们，如果编程语言提供了异常处理，我们必须使用它去处理异常，这样我们就能专注于业务本身的逻辑。

最后，我们再回顾一下本文的重点：
- 使用异常而非返回码
- 不要忽视异常处理
- 不要忽视 Promise reject
- 定义异常层次结构（Exceptions Hierarchy）
- 提供异常的上下文
