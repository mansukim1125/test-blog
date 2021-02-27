### Router 만들기

아래와 같이 생긴 URL구조가 있다.

```javascript
const routes = [
    {
        path: '/',
        component: NavBarComponent,
        redirect: '/home',
        children: [
            {
                path: '/home',
                component: HomeComponent
            },
            {
                path: '/projects',
                component: ProjectsComponent
            }
        ]
    },
    {
        path: '/blog',
        component: BlogNavBarComponent,
        redirect: '/blog/posts',
        children: [
            {
                path: '/posts',
                component: PostsComponent
            },
            {
                path: '/posts/:id',
                component: PostComponent,
                param: {
                    type: String,
                    name: 'postId'
                }
            }
        ]
    }
];
```

다음을 실행하면,

```javascript
const router = new Router(routes);

router.push('/');
```

`router.push('/')`의 결과값은 

```
[
    {
        path: '/',
        component: NavBarComponent { param: null, parent: null, mountPosId: 'app' }
    },
    {
        path: '/home',
        component: HomeComponent {
            param: null,
            parent: [Object],
            mountPosId: 'main-container'
    	}
    }
]
```

이다. `parent`의 `[Object]`는 바로 위의 `path: '/'`인 객체를 참조한다. 그럼 지난 글에서 말했던 알고리즘 설계에 대한 이야기를 해보자.

### 특정 URL 접속

예를 들어 `/blog/posts/45`로 이동한다고 해보자. 큰 흐름은 `routes`를 순회하면서 조건에 맞는 Component를 찾아 Stack 형식으로 만드는 것이다. 진행 순서는

1. `routes`순회 알고리즘 설계
2. 1.을 수행하면서 `path`를 Stack에 쌓기.
3. 2.의 결과를 정규표현식으로 변환.
4. 3.의 결과와 접속하고자 하는 URL간 일치 여부 확인.

이다.

#### `routes`순회 알고리즘

```javascript
function traverseRoutes(routes) {
    const routesLength = routes.length;
    for (let i = 0; i < routesLength; ++i) {
        console.log(routes[i]);
        if (routes[i].children) traverseRoutes(routes[i].children);
    }
}
```

이 코드는 `routes`배열을 순회하는 알고리즘이다. 여기에 살을 붙혀보자.

#### `path` Stack에 쌓기

```javascript
let pathStack = [];
function traverseRoutes(routes) {
    const routesLength = routes.length;
    for (let i = 0; i < routesLength; ++i) {
        pathStack.push(routes[i].path);
        console.log(pathStack.join(''));
        if (routes[i].children) traverseRoutes(routes[i].children);
    	pathStack.pop();
    }
}
```

Stack에 저장된 값을 정규표현식으로 변환해 보자.

#### Stack을 정규표현식으로 변환

```javascript
function convertToRegex(joinedStr) {
    const path = joinedStr.replace(/\/+/, '/').replace(/:\w+/, '(.+)');
    return new RegExp(path);
}

let pathStack = [];

function traverseRoutes(routes) {
    const routesLength = routes.length;
    for (let i = 0; i < routesLength; ++i) {
        pathStack.push(routes[i].path);
        const joinedPath = pathStack.join('');
        console.log(convertToRegex(joinedPath));
        if (routes[i].children) traverseRoutes(routes[i].children);
    	pathStack.pop();
    }
}
```

먼저 순회하면서 저장된 Stack의 구성을 합친 후, 그 문자열을 정규표현식으로 변환한다.

1. `/`가 연달아 나오는 경우, 1개의 `/`로 대치.
2. `:<some-parameter>`가 나오는 경우, `<any-charactor>`로 대치.

그 결과는 다음과 같다.

```
/\//
/\/home/
/\/projects/
/\/blog/
/\/blog\/posts/
/\/blog\/posts\/(.+)/
```

#### 이동하고자 하는 URL과 정규표현식의 일치 여부

`이동하고자 하는 URL과 정규표현식이 일치한다.`는 문장의 의미가 무엇인지 생각해 볼 필요가 있다. 우리의 목적은 URL과 정규표현식이 100% 일치하는 경우를 찾는 것이다. 100% 일치한다는 것은 무엇인가? **URL의 시작과 끝이 생성된 정규표현식과 일치한다는 것**을 의미한다.

1. URL이 정규표현식의 패턴으로 시작하는지?
   1. 시작하면 2.로
   2. 시작하지 않으면 일치 실패
2. URL이 정규표현식의 패턴으로 끝나는지?
   1. 끝나면 일치 성공
   2. 끝나지 않으면 children이 있는지 확인
      1. 있으면 `traverseRoutes(children)`호출
      2. 없으면 일치 실패

이를 알고리즘으로 구현해 보자.

