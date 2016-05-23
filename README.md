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

