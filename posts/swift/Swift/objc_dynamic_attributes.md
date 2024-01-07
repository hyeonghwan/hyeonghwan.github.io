---
title: '@Objc @dynamic Attributes'
date: '2024-01-11 16:40:30'
tags: 'Swift, Objc, Dynamic'
desc: 'Objc Dynamic 속성에 관한 글입니다.'
---


## @Objc, dynamic 속성은 어떻게 작동하는가

> Objective-C runtime visibility and the depths of dynamic dispatch in the modern Swift
>
> Objective C 와 Swift 간의 상호운용성을 고려할 때는 두가지 요소가 있습니다. 

- @objc 키워드는 Swift코드를 Objective-C 에 노출하는 역할을 합니다.
- dynamic 키워드는 Objective-C의 dynamic Dispatch 를 사용하기위해 명시해야 합니다. 

### 내가 알고 있는것

> Swift 는 기본적으로 정적 Dispatch를 사용하지만 Dynamic Dispatch 로 작동을 할때 Vtable을 만들어서 사용하는 것으로 알고 있습니다.
> Swift 는 기본적으로 Native Language 이기 때문에 Objective-C 와는 별개로 작동을 하고 Objective-C 와의 상호운용성을 위해서 이러한 지시자들이 있는것으로 알고있습니다.

- **Vtable**: 객체 지향 프로그래밍에서 다형성을 지원하기 위해서 지원하는 함수의 포인터 배열이라고 생각하면 됩니다. -> 예를 들어 부모클래스 에서 debugPrint 라는 함수를 서브클래스에서 override 해서 재정의 했을때 함수를 호출할때 부모 또는 자식 어떤 클래스의 함수를 호출할지 결정하기 위해서 사용됩니다. 
```swift
class TestObjcDynamic {
    func perform() -> Int {
        return 42
    }

    @objc func performOBJC() -> Int {
        return 42
    }

    @objc dynamic func performDynamic() -> Int {
        return 42
    }
}
```

1. perform 은 Swift 기본 함수
2. performOBJC 은 Objective-C Run time 에 노출하도록 (가시성?ㅇ Objective C에서 코드를 호출하도록 노출하는 의미로 생각하고 있습니다.)  @objc 지시자를 선언하였습니다.
3. performDynamic 은 dynamic Dispatch 를 하도록 dynamic 키워드를 선언하였습니다. 이 함수도 2번과 마찬가지로 Objective-C 에 노출됩니다. 


## Swift Compilation

> Swift 는 컴파일 언어로써 Sil -> IR -> .O 파일로 컴파일 됩니다. 자세한 내용은 추후 업로드를 해보겠습니다.
> 중간 과정에 SIL 을 살펴볼 예정이므로
``` swift
swiftc -emit-sil Objc_dynamic.swift
// 이 명령어를 사용하여 Swift Intermediate Language 로 컴파일하여 한번 살펴보겠습니다.
```


```swift
// TestObjcDynamic.perform()
sil hidden @$s12Objc_dynamic04TestA7DynamicC7performSiyF : $@convention(method) (@guaranteed TestObjcDynamic) -> Int {
// %0 "self"                                      // user: %1
bb0(%0 : $TestObjcDynamic):
  debug_value %0 : $TestObjcDynamic, let, name "self", argno 1, implicit // id: %1
  %2 = integer_literal $Builtin.Int64, 42         // user: %3
  %3 = struct $Int (%2 : $Builtin.Int64)          // user: %4
  return %3 : $Int                                // id: %4
} // end sil function '$s12Objc_dynamic04TestA7DynamicC7performSiyF'
```
좀 난해해 보이지만
> name mangling 을 통해서 클래스와 함수의 identity를 설정합니다.

> 정의한 클래스와 함수는 mangling 된 식별자를 자세히 보면 선언했던 클래스와 함수의 이름이 섞여 들어가 있는걸 볼수 있습니다.

> 42를 리턴하기 위해서 적당히 많은 일들이 일어나게 됩니다... ㅎ (대충 이해만 하고 넘어가보자)

> 이 외에도 함수를 호출하기 위해서 Class Type을 인자로 받는 부분이 인상적이지만 Objc 와 dynamic 부분만 집중적으로 살펴보도록 합시다.

```swift
// TestObjcDynamic.performOBJC()
sil hidden @$s12Objc_dynamic04TestA7DynamicC11performOBJCSiyF : $@convention(method) (@guaranteed TestObjcDynamic) -> Int {
// %0 "self"                                      // user: %1
bb0(%0 : $TestObjcDynamic):
  debug_value %0 : $TestObjcDynamic, let, name "self", argno 1, implicit // id: %1
  %2 = integer_literal $Builtin.Int64, 42         // user: %3
  %3 = struct $Int (%2 : $Builtin.Int64)          // user: %4
  return %3 : $Int                                // id: %4
} // end sil function '$s12Objc_dynamic04TestA7DynamicC11performOBJCSiyF'

// @objc TestObjcDynamic.performOBJC()
sil private [thunk] @$s12Objc_dynamic04TestA7DynamicC11performOBJCSiyFTo : $@convention(objc_method) (TestObjcDynamic) -> Int {
// %0                                             // users: %4, %3, %1
bb0(%0 : $TestObjcDynamic):
  strong_retain %0 : $TestObjcDynamic             // id: %1
  // function_ref TestObjcDynamic.performOBJC()
  %2 = function_ref @$s12Objc_dynamic04TestA7DynamicC11performOBJCSiyF : $@convention(method) (@guaranteed TestObjcDynamic) -> Int // user: %3
  %3 = apply %2(%0) : $@convention(method) (@guaranteed TestObjcDynamic) -> Int // user: %5
  strong_release %0 : $TestObjcDynamic            // id: %4
  return %3 : $Int                                // id: %5
} // end sil function '$s12Objc_dynamic04TestA7DynamicC11performOBJCSiyFTo'

// @objc TestObjcDynamic.performDynamic()
sil private [thunk] @$s12Objc_dynamic04TestA7DynamicC07performD0SiyFTo : $@convention(objc_method) (TestObjcDynamic) -> Int {
// %0                                             // users: %4, %3, %1
bb0(%0 : $TestObjcDynamic):
  strong_retain %0 : $TestObjcDynamic             // id: %1
  // function_ref TestObjcDynamic.performDynamic()
  %2 = function_ref @$s12Objc_dynamic04TestA7DynamicC07performD0SiyF : $@convention(method) (@guaranteed TestObjcDynamic) -> Int // user: %3
  %3 = apply %2(%0) : $@convention(method) (@guaranteed TestObjcDynamic) -> Int // user: %5
  strong_release %0 : $TestObjcDynamic            // id: %4
  return %3 : $Int                                // id: %5
} // end sil function '$s12Objc_dynamic04TestA7DynamicC07performD0SiyFTo'

// TestObjcDynamic.performDynamic()
sil hidden @$s12Objc_dynamic04TestA7DynamicC07performD0SiyF : $@convention(method) (@guaranteed TestObjcDynamic) -> Int {
// %0 "self"                                      // user: %1
bb0(%0 : $TestObjcDynamic):
  debug_value %0 : $TestObjcDynamic, let, name "self", argno 1, implicit // id: %1
  %2 = integer_literal $Builtin.Int64, 42         // user: %3
  %3 = struct $Int (%2 : $Builtin.Int64)          // user: %4
  return %3 : $Int                                // id: %4
} // end sil function '$s12Objc_dynamic04TestA7DynamicC07performD0SiyF'
```
- objc 함수와 objc dynamic 함수 모두 Swift 에서 사용될 sil을 생성하고
- Objective-C 에서 사용할수 있도록 함수 포인터를 전달하는걸 볼 수 있습니다.
-> **dynamic 을 붙인 함수와 그냥 objc 함수의 sil이 동일한 것을 볼 수 있습니다.**
> ㅇ ? 동일하면 똑같은 건가 ?

### Vtable 을 살펴보자
```swift
sil_vtable TestObjcDynamic {
  #TestObjcDynamic.perform: (TestObjcDynamic) -> () -> Int : @$s12Objc_dynamic04TestA7DynamicC7performSiyF	// TestObjcDynamic.perform()
  #TestObjcDynamic.performOBJC: (TestObjcDynamic) -> () -> Int : @$s12Objc_dynamic04TestA7DynamicC11performOBJCSiyF	// TestObjcDynamic.performOBJC()
  #TestObjcDynamic.init!allocator: (TestObjcDynamic.Type) -> () -> TestObjcDynamic : @$s12Objc_dynamic04TestA7DynamicCACycfC	// TestObjcDynamic.__allocating_init()
  #TestObjcDynamic.deinit!deallocator: @$s12Objc_dynamic04TestA7DynamicCfD	// TestObjcDynamic.__deallocating_deinit
}
```
1. perform
2. performOBJC
3. init
4. deinit 
**이 정의 되어있는데 performDynamic 이 보이지 않습니다. ??**
> 호출하는 코드를 작성하고 다시 sil을 생성해보겠습니다.

```swift
let test = TestObjcDynamic()
let p1 = test.perform()
let p2 = test.performOBJC()
let p3 = test.performDynamic()

// 일부만 가져왔습니다.
%11 = class_method %10 : $TestObjcDynamic, #TestObjcDynamic.perform : (TestObjcDynamic) -> () -> Int, $@convention(method) (@guaranteed TestObjcDynamic) -> Int // user: %12
 %17 = class_method %16 : $TestObjcDynamic, #TestObjcDynamic.performOBJC : (TestObjcDynamic) -> () -> Int, $@convention(method) (@guaranteed TestObjcDynamic) -> Int // user: %18
%23 = objc_method %22 : $TestObjcDynamic, #TestObjcDynamic.performDynamic!foreign : (TestObjcDynamic) -> () -> Int, $@convention(objc_method) (TestObjcDynamic) -> Int // user: %24
```
perform, performObjc 는 Vtable을 조회하여 class_method 로 호출합니다.
반면에 performDynamic 은 objc_method 방식 Objective-c의 메서드로 호출됩니다.

dynamic 키워드를 사용하면 Swift 메서드 호출방식이 아닌 Objective-c 로 호출로 강제되는 것을 볼 수 있습니다.


### 성능
성능상으로는 Objective-C의 동적 디스패치로 작동하게 되면 성능이 많이 하락하게 됩니다.

- static > swift dynamic using Vtable > objc_msgSend

### 사용처
@objc dyanmic 은 주로 KVO 특정 프로퍼티의 값을 Objective-C 의 힘을 빌려 값이 런타임에 변경되었을때 관찰하도록 지시자를 추가하고 observe 함수로 관찰할수 있습니다. 

- dynamic 키워드는 Objective-C의 동적 디스패치를 하도록 하기 때문에 런타임 상에서 값이 변경되었을때 값을 관찰할 수 있습니다. KVO 나 method Swizzling (런타임 상에서 메서드를 exchange) 할때 많이 쓰입니다.
- @objc 는 Objective - C에 노출되어 Swift 코드와 Objc 간의 상호운용성을 지원하기 위한 지시자 입니다. 주로 button 의 이벤트 호출할때 많이 작성했죠, objective-c에서 swift 코드를 호출하기 위해서는 이 지시자가 필요합니다.(@objc 지시자를 쓰기 위해서는 NSObject 를 상속 받아야 합니다.)
