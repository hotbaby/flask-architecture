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

![session.png](https://bitbucket.org/repo/zp95RA/images/2819649157-session.png)

## 2.3 session Sequence diagram

**Flask session data flow**

![session-flow.png](https://bitbucket.org/repo/zp95RA/images/3022005732-session-flow.png)

# 4 References