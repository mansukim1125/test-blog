---
description: 블로그에서 이동하고자 하는 URL로 이동하는 방법을 알아봅니다.
---

우리가 하고자 하는 것은 이동하고자 하는 URL과 이미 JSON 형식으로 정의된 URL구조 중 완벽히 일치하는 URL구조를 만들어내는 것이다. 이는 다음의 내용으로 이어진다.

### 이동하고자 하는 URL과 생성된 정규표현식의 일치 여부

`traverseRoutes(routes)`함수의 흐름:

1. URL이 정규표현식의 패턴으로 시작하는지?
   1. 시작하면 2.로
   2. 시작하지 않으면 일치 실패
2. URL이 정규표현식의 패턴으로 끝나는지?
   1. 끝나면 일치 성공
   2. 끝나지 않으면 children이 있는지 확인
      1. 있으면 `traverseRoutes(children)`호출
      2. 없으면 일치 실패
3. 이를 `routes`만큼 반복.

이를 알고리즘으로 구현해 보자.

### 여러가지 필요한 함수

#### `pathToRegex`

`pathObj`를 정규표현식으로 변환하는 함수이다.

```javascript
function pathToRegex(pathObj) {
    let path = pathObj.path;
    if (pathObj.param) {
        if (pathObj.param.type === 'Number') {
            path = path.replace(/:\w+/, '(\\d+)');
        } else {
            path = path.replace(/:\w+/, '(.+)');
        }
    }
    return path;
}
```

#### `regexReducer`

순회하면서 Stack에 누적된 정규표현식을 하나로 합치는 함수이다.

```javascript
const regexReducer = (pathStack) => new RegExp(pathStack.join('').replace(/\/+/, '/'));
```

#### `startsWith`

이동하고자 하는 경로(`str`)가 `regexReducer`에 의해 하나로 합쳐진 정규표현식으로 시작하는지를 확인하는 함수이다.

```javascript
function startsWith(str, regex) {
    const reg = new RegExp(/^/.source + regex.source);
    return str.match(reg) ? true : false;
}
```

#### `endsWith`

이동하고자 하는 경로(`str`)가 `regexReducer`에 의해 하나로 합쳐진 정규표현식으로 끝나는지를 확인하는 함수이다.

```javascript
function endsWith(str, regex) {
    const reg = new RegExp(regex.source + /$/.source);
    return str.match(reg) ? true : false;
}
```

#### URL이 정규표현식의 패턴으로 시작하는지?

```javascript
const regexStr = this.pathToRegex(routes[i]); // 현재 순회하고 있는 경로의 정규표현식
this.pathStack.push(regexStr); // children까지 확인하기 위해 Stack에 추가
const reducedRegex = regexReducer(this.pathStack); // Stack에 저장된 정규표현식을 하나로 합침

if (!startsWith(this.targetPath, reducedRegex)) {
    // 이동하고자 하는 경로가 하나로 합친 경로로 시작되지 않으면 일치 실패
    // algorithm here.
}
```

##### 시작하지 않으면 일치 실패

```javascript
if (!startsWith(this.targetPath, reducedRegex)) {
    // 이동하고자 하는 경로가 하나로 합친 경로로 시작되지 않으면 일치 실패
    this.pathStack.pop();
    continue;
}
```

#### 시작하면, URL이 정규표현식의 패턴으로 끝나는지?

```javascript
if (endsWith(this.targetPath, reducedRegex)) {
    // algorithm here.
}
```

##### 끝나면 일치 성공

```javascript
if (endsWith(this.targetPath, reducedRegex)) {
    return true;
}
```

##### 끝나지 않으면 children이 있는지 확인

```javascript
if (routes[i].children) {
    const result = this.traverseRoutes(routes[i].children);
    if (result) return result;
}
```

##### 없으면 일치 실패

```javascript
this.pathStack.pop();
```

이러한 과정을 통해 알고리즘을 구성할 수 있다.
