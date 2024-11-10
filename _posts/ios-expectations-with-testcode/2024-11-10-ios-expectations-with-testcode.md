---
title: What's Expectation? study on iOS Project TestCode (with RxSwift, ReSwift)
date: 2024-11-10 15:55:00 +07:00
modified: 2024-11-10 15:55:00 +07:00
tags: [ios, RxSwift, ReSwift, in-english]
description: Let's see the testcode of a ThunkCreator which has API calls and business logic about the API response.
---

There's this testcode of iOS project and I’d like to focus on the expectation part.


```swift

import XCTest  
  
func commonSetup() {  
    // set store, thunkCreator...  
}  
  
func useCase1pattern() {  
    let condition1 = "something differs"  
    let condition2 = "something differs"  
      
    commonExecutor(apiParameter1: condition1, apiParameter2: condition2, expectResult: "result", testDescription: "should be something")  
}  
  
  
func commonExecutor(apiParameter1: String, apiParameter2: String, expectResult: String, testDescription: String) {  
    commonSetup()  
      
    let expectation = Expectation  
    expectation.expectedFulfillmentCount = 1  
      
    store.state.afterResult  
        .subscribe(onNext: { afterResult in  
            XCTAssertEqual(expectResult, afterResult, testDescription)  
            expectation.fulfill()  
        }).disposed(by: disposeBag)  
      
    let apiParameter = Parameter(apiParameter1, apiParameter2)  
    state.dispatch(thunkCreator.send(element: apiParameter))  
      
    waitForExpectations(timeout: 1, handler: nil)  
}

```
  

#### Swift document
[<img src="[Screenshot 2024-11-10 at 15.01.51.png]">](https://developer.apple.com/documentation/xctest/asynchronous_tests_and_expectations)


It’s used for asynchronous code because asynchronous code could be not executed on main thread.

Two approaches for testing asynchronous code
1. For async/await code, `func testBlah() async throws {}`
2. Not async/await code but also asynchronous code, use `expectations` and control the `waiting/fulfillment`

<hr>

  

#### Usage

```swift
// Create an expectation for an asynchronous task.  
let expectation = XCTestExpectation(description: "Open a file asynchronously.")  
  
let fileManager = ExampleFileManager()  
  
// Perform the asynchronous task.  
fileManager.openFileAsync(with: "exampleFilename") { file, error in  
in  
    // Assert that the asynchronous task worked.  
    XCTAssertNotNil(file, "Expected to load a file.")  
  
  
    // Assert that no errors occurred opening the file asynchronously.  
    XCTAssertNil(error, "Expected no errors loading a file.")  
      
    // Fulfill the expectation.  
    expectation.fulfill()  
}  
  
  
// Wait for the expectation to fulfill or time out.  
wait(for: [expectation], timeout: 10.0)
```
  

<hr>

I quite got it, but also got a question why it doesn’t use async throws test function because our asynchronous API request is using async code like below.


```swift
extension GETAsyncRequestable where Self: AsyncRequestBase {  
    func send(with parameter: Parameter) async throws -> Response {  
        createRequest(...)  
          
        return try await requestJson()  
    }  
}
```




Then I figured out that our ThunkCreator’s send function(the one be tested up there) is not async function and it coud be refactored. I’m gonna try to refactor and upload Pull Request about that in couple days 😙


#### Resourcees

- [SwiftDoc asynchronous tests and expectations](https://developer.apple.com/documentation/xctest/asynchronous_tests_and_expectations)  
