# Flask ctx stack

## Flask Local Stack 

Flask 通过LocalStack()创建_request_ctx_stack, _app_ctx_stack两个栈帧实例。

```
_request_ctx_stack = LocalStack()
_app_ctx_stack = LocalStack()
```

操作的对象分别是`RequestContext`和`AppContext`

### Local

Local 使用dict缓存所有的上下文，通过greenlet包的getcurrent获取当前线程ID
```
def __init__(self):
    object.__setattr__(self, '__storage__', {})
    object.__setattr__(self, '__ident_func__', get_ident)
```

Local 使用运算符重载定制属性访问，设置、获取当前线程的栈
```
def __getattr__(self, name):
    try:
        return self.__storage__[self.__ident_func__()][name]
    except KeyError:
        raise AttributeError(name)

def __setattr__(self, name, value):
    ident = self.__ident_func__()
    storage = self.__storage__
    try:
        storage[ident][name] = value
    except KeyError:
        storage[ident] = {name: value}

def __delattr__(self, name):
    try:
        del self.__storage__[self.__ident_func__()][name]
    except KeyError:
        raise AttributeError(name)
```

Local 数据结构
```
{
    “thread_id1”: {
        "stack":  [ctx,]
    },
    "thread_id2": {
        "stack": [ctx,]
    },
    ...
}
```

### Stack

Local stack 利用Local存储上下文，并模拟栈的push, pop, top 操作。

Local stack初始化

```
def __init__(self):
    self._local = Local()
```

push 操作

```
def push(self, obj):
    """Pushes a new item to the stack"""
    rv = getattr(self._local, 'stack', None)
    if rv is None:
        self._local.stack = rv = []
    rv.append(obj)
    return rv
```

pop 操作

```
def pop(self):
    """Removes the topmost item from the stack, will return the
    old value or `None` if the stack was already empty.
    """
    stack = getattr(self._local, 'stack', None)
    if stack is None:
        return None
    elif len(stack) == 1:
        release_local(self._local)
        return stack[-1]
    else:
        return stack.pop()
```

top 操作

```
@property
def top(self):
    """The topmost item on the stack.  If the stack is empty,
    `None` is returned.
    """
    try:
        return self._local.stack[-1]
    except (AttributeError, IndexError):
        return None
```

## Local Proxy

Flask 中的LocalProxy实例:

运行时当前app的代理 `current_app`

```
def _find_app():
    top = _app_ctx_stack.top
    if top is None:
        raise RuntimeError(_app_ctx_err_msg)
    return top.app

current_app = LocalProxy(_find_app)
```

运行时当前request代理`request`

```
def _lookup_req_object(name):
    top = _request_ctx_stack.top
    if top is None:
        raise RuntimeError(_request_ctx_err_msg)
    return getattr(top, name)

request = LocalProxy(partial(_lookup_req_object, 'request'))
```

运行时当前session 代理`session`

```
def _lookup_req_object(name):
    top = _request_ctx_stack.top
    if top is None:
        raise RuntimeError(_request_ctx_err_msg)
    return getattr(top, name)

session = LocalProxy(partial(_lookup_req_object, 'session'))
```

运行时g的代理

```
def _lookup_app_object(name):
    top = _app_ctx_stack.top
    if top is None:
        raise RuntimeError(_app_ctx_err_msg)
    return getattr(top, name)

g = LocalProxy(partial(_lookup_app_object, 'g'))
```

## LocalProxy 原理

LocalProxy 本质上讲还是一个代理，通过运算符重载，定制几乎所有的属性设置、获取。

初始化时，设置代理对象
```
def __init__(self, local, name=None):
    object.__setattr__(self, '_LocalProxy__local', local)
    object.__setattr__(self, '__name__', name)
```

运行时动态获取代理对象

```
def _get_current_object(self):
    """Return the current object.  This is useful if you want the real
    object behind the proxy at a time for performance reasons or because
    you want to pass the object into a different context.
    """
    if not hasattr(self.__local, '__release_local__'):
        return self.__local()
    try:
        return getattr(self.__local, self.__name__)
    except AttributeError:
        raise RuntimeError('no object bound to %s' % self.__name__)
```

运算符重载

```
    @property
    def __dict__(self):
        try:
            return self._get_current_object().__dict__
        except RuntimeError:
            raise AttributeError('__dict__')

    def __repr__(self):
        try:
            obj = self._get_current_object()
        except RuntimeError:
            return '<%s unbound>' % self.__class__.__name__
        return repr(obj)

    def __bool__(self):
        try:
            return bool(self._get_current_object())
        except RuntimeError:
            return False

    def __unicode__(self):
        try:
            return unicode(self._get_current_object())  # noqa
        except RuntimeError:
            return repr(self)

    def __dir__(self):
        try:
            return dir(self._get_current_object())
        except RuntimeError:
            return []

    def __getattr__(self, name):
        if name == '__members__':
            return dir(self._get_current_object())
        return getattr(self._get_current_object(), name)

    def __setitem__(self, key, value):
        self._get_current_object()[key] = value

    def __delitem__(self, key):
        del self._get_current_object()[key]
```