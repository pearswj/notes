# OAuth web app flow

Developer guide: https://developer.github.com/v3/oauth/#web-application-flow

Sample web app (Ruby): https://developer.github.com/guides/basics-of-authentication

## 1. Prerequisites

[Register an application](https://github.com/settings/applications/new) and make a note of the `client ID` and `client secret`. Set  the callback URL to `your.web.app/callback`.

## 2. Redirect users to request GitHub access

Assuming the environment variables `CLIENT_ID` and `CLIENT_SECRET` exist...

```
$ open https://github.com/login/oauth/authorize?client_id=${CLIENT_ID}&scope=(no scope)
```

## 3. GitHub redirects back to your site

GitHub redirects back to your site (via the callback URL) with a temporary code in a `code` parameter.

```python
from Flask import Flask, redirect, request, session
from os import environ

app = Flask(__name__)
app.config['SECRET_KEY'] = 'Waffling on is really great for these secrets brandy cactus beartrap'

@app.route('/callback')
def callback():
    if 'code' in request.args:
        url = 'https://github.com/login/oauth/access_token'
        payload = {
            'client_id': environ.get('CLIENT_ID'),
            'client_secret': environ.get('CLIENT_SECRET'),
            'code': request.args['code']
        }
        headers = {'Accept': 'application/json'}
        r = requests.post(url, params=payload, headers=headers)
        response = r.json()
        # get access_token from response and store in session
        if 'access_token' in response:
            session['access_token'] = response['access_token']
        else:
            app.logger.error('github didn\'t return an access token, oh dear')
        # send authenticated user where they're supposed to go
        return redirect(url_for('index'))
    return '', 404
```

## 4. Use the access token to access the API

```python
@app.route('/')
def index():
    # authenticated?
    if not 'access_token' in session:
        return 'Never trust strangers', 404
    # get username from github api
    url = 'https://api.github.com/user?access_token={}'
    r = requests.get(url.format(session['access_token']))
    try:
        login = r.json()['login']
    except AttributeError:
        app.logger.debug('error getting username from github, whoops')
        return 'I don't know who you are; I should, but regretfully I don't', 500
    return 'Hello {}!'.format(login), 200
```


[1]: https://developer.github.com/v3/oauth/#redirect-users-to-request-github-access
[2]: https://developer.github.com/v3/oauth/#github-redirects-back-to-your-site
