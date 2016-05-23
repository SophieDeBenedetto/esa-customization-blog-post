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