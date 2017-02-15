# 1 Introduction


# 2 Architecture

## 2.1 Principle
Flask framework使用session module 来管理用户会话信息。 session本质上是一个dict，用户的sign_in, sign_out通过更新 `session['user_id']`，来更新Cookie信息。 `SecureCookieSessionInterface`根据`session`是否更新，如何更新，来决定是否`set_cookie()`

**Sign-in API**  
```python
def login_user(user):
    ...
    user_id = getattr(user, current_app.login_manager.id_attribute)()
    session['user_id'] = user_id
    ...
```

**Sign-out API**  
```python
def logout_user():
    ...
    if 'user_id' in session:
        session.pop('user_id')
    ...
```

**Set-Cookie**  
```python
class SecureCookieSessionInterface():

    def save_session(self, app, session, response):
        ...
        if not self.should_set_cookie(app, session):
            return

        ...
        respone.set_cookie(...)
        ...
```


## 2.2 session Class  diagram

**Flask session module framework**    

![session.png](https://bytebucket.org/hotbaby/resource/raw/da7f226e5e9842ecdfaec267bd3076dbd3b448ab/flask/flask-session.png?token=af7615aaa8480b3cf74fa8bde4387db49328a572)

## 2.3 session Sequence diagram

**Flask session data flow**

![session-flow.png](https://bytebucket.org/hotbaby/resource/raw/da7f226e5e9842ecdfaec267bd3076dbd3b448ab/flask/flask-session-flow.png?token=d90303052b7b77896ed0f51600d2c3c499216a56)

# 4 References
