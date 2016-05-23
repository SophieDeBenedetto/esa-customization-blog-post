```javascript
import Ember from 'ember';
import Base from 'ember-simple-auth/authenticators/base';
import config from '../config/environment';

const { RSVP: { Promise }, $: { ajax }, run } = Ember;

export default Base.extend({
  tokenEndpoint: `${config.host}/knock/auth_token`,

  restore(data) {
    debugger;
    return new Promise((resolve, reject) => {
      if (!Ember.isEmpty(data.token)) {
        resolve(data);
      } else {
        reject();
      }
    });
  },

  authenticate(creds) {
    debugger;
    const { identification, password } = creds;
    const data = JSON.stringify({
      auth: {
        email: identification,
        password
      }
    });

    const requestOptions = {
      url: this.tokenEndpoint,
      type: 'POST',
      data,
      contentType: 'application/json',
      dataType: 'json'
    };

    return new Promise((resolve, reject) => {
      debugger;
      ajax(requestOptions).then((response) => {
        debugger;
        const { jwt } = response;
        // Wrapping aync operation in Ember.run
        run(() => {
          debugger;
          resolve({
            // store token in localStorage
            token: jwt
          });
        });
      }, (error) => {
        // Wrapping aync operation in Ember.run
        run(() => {
          reject(error);
        });
      });
    });
  },

  invalidate(data) {
    debugger;
    return Promise.resolve(data);
  }
});
```

### Promise

Allows you to write asynchronous code in a synchronous fashion. a `Promise` object represents the data that will be returned by the web service in future. In the meantime the Promise object acts like a proxy to the actual data. Furthermore, you can attach callbacks to the Promise object which will be called once the actual data is available.

Instantiate a new `Promise` object, giving it an anonymous function that takes two arguments, `resolve` and `reject`, which are both functions. Your anych code goes in the body of that parent anon function--i.e. making your ajax request. Then, if you get a successful response, invoke the resolve function, passing it an argument of the response, if you don't get a successful response back, invoke the reject function, passing it an argument of the error you got back from your web request. 

You can then chain functions onto your promise. The `then` function gets invoked if `resolve` was invoked, getting passed an argument of the return value of `resolved`. The `catch` function gets invoked if the `reject` function was invoked, getting passed an argument of the return value of  `reject`. 

* run loop : https://guides.emberjs.com/v2.5.0/applications/run-loop/

### The Authenticate Function

Called by the session when the session needs to authenticate (okay but actually when is that?).Gets the credentials (INVOKED BY YOU, WHEN YOU CALL authenticate in your login or signin controller dummy), uses them to structure the ajax request. Creates new Promise that makes the ajax request, if success, resolve function sets token in localStorage to response, which is encrypted jwt token. If failure, passed error response to reject function. That error then can get passed to a catch function, if you use it. If success, pass return of resolve function to then function (optional. Our then functions don't really use that response. We are only trying to get the token into localStorage, not get a reasl response from API to display to user.)

Where do we call it???

```javascript
# login controller
import Ember from 'ember';

export default Ember.Controller.extend({
  session: Ember.inject.service(),

  actions: {
    authenticate: function() {
      var credentials = this.getProperties('identification', 'password'),
        authenticator = 'authenticator:jwt';
      this.get('session').authenticate(authenticator, credentials).catch((reason)=>{
        this.set('errorMessage', reason.error || reason);
      });
    }
  }
});

```

Right here! when we call `this.get('session').authenticate(<credentials>)`. 

### Invalidate Function

```javascript
invalidate(data) {
    return Promise.resolve(data);
  }
```

This method is invoked as a callback when the session is invalidated. While the session will invalidate itself and clear all authenticated session data, it might be necessary for some authenticators to perform additional tasks (e.g. invalidating an access token on the server side).

This method returns a promise. A resolving promise will result in the session becoming unauthenticated. A rejecting promise will result in invalidation being intercepted and the session remaining authenticated.

The BaseAuthenticator's implementation always returns a resolving promise and thus never intercepts session invalidation. This method doesn't have to be overridden in custom authenticators if no actions need to be performed on session invalidation.

Where do we call it???

```javascript
# application route

actions: {
    invalidateSession: function() {
        this.get('session').invalidate();
    }
  }
```

Remember: It clears the session BY ITSELF. ooooo magic!

### Restore

```javascript
restore(data) {
    debugger;
    return new Promise((resolve, reject) => {
      if (!Ember.isEmpty(data.token)) {
        resolve(data);
      } else {
        reject();
      }
    });
  }
```

Restores the session from a session data object. This method is invoked by the session either on application startup if session data is restored from the session store or when properties in the store change due to external events (e.g. in another tab) and the new session data needs to be validated for whether it constitutes an authenticated session.

This method returns a promise. A resolving promise results in the session becoming or remaining authenticated. Any data the promise resolves with will be saved in and accessible via the session service's data.authenticated property (see data). A rejecting promise indicates that data does not constitute a valid session and will result in the session being invalidated or remaining unauthencicated.

The BaseAuthenticator's implementation always returns a rejecting promise. This method must be overridden in subclasses.