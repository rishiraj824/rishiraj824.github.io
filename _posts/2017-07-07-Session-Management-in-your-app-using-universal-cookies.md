---
title: Minimal User Sessions and Management Using Universal-Cookies 
desc: Manage Cookies using universal-cookies
---


If you login to [SUSI Web Chat](http://github.com/fossasia/chat.susi.ai), and come back again after some days, you find that you didn’t have to login and all your previous sent messages are in there in the message pane. To achieve this, SUSI Web Chat uses cookies stored in your browser which is featured in this blog.  

In ReactJS, it’s highly misleading and extensive to use the conventional Javascript methodology of saving tokens and deleting them. However, [universal-cookie](https://www.npmjs.com/package/universal-cookie), a node package allows you to store and get cookies with the least possible confusion. In the following examples, I have made use of the _get_, _set_ and _remove_ functions of the universal-cookie package which have documentations easily available at [this link.](https://github.com/reactivestack/cookies/blob/master/packages/universal-cookie/README.md) The basic problem one finds while setting cookies and maintaining sessions is the time through which it should be valid and to secure the routes. We shall look at the steps below to figure out how was it implemented in [SUSI Web Chat](http://github.com/fossasia/chat.susi.ai).

1\. The first step is to install the packages by using the following command in your project’s root-
```
npm install universal-cookie --save
```
2\. Import the Cookies Class, where-ever you want to create a Cookie object in your repository.
```
import Cookies from 'universal-cookie';
```
Create a Cookie object at the same time in the file you want to use it,
```
const cookies = new Cookies();
```
3\. We make use of the _set_ function of the package first, where we try to set the cookie while the User is trying to login to the account.

**Note** - The cookie value can be set to any value one wants, however, here I am setting it to the access token which is generated by the server so that I can access it throughout the application.

*   AJAX Call to the [server](http://api.susi.ai)

*   Login endpoint for SUSI AI [http://api.susi.ai/aaa/login.json?type=access-token&login=EMAIL&password=PASSWORD](http://api.susi.ai/aaa/login.json?type=access-token&login=EMAIL&password=PASSWORD)
```
$.ajax({ options: options,
        success: function (response) {
//Get the response token generated from the server
                let accessToken = response.access_token;                       // store the current state
                 let state = this.state;
// set the time for which the session needs to be valid
            let time = response.valid_seconds;
//set the access token in the state
             state.accessToken = accessToken;
// set the time in the state
             state.time = time;           
// Pass the accessToken and the time through the binded function
             this.handleOnSubmit(accessToken, time);
            }.bind(this),
        error: function ( jqXHR, textStatus, errorThrown) {
                   // Handle errors
                   }
        });
```
Function - **handleOnSubmit()**
```
// Receive the accessToken and the time for which it needs to be valid
handleOnSubmit = (loggedIn, time) => {
        let state = this.state;
        if (state.success) {
              // set the cookie of with the value of the access token at path ‘/’ and set the time using the parameter ‘maxAge’
            cookies.set('loggedIn', loggedIn, { path: '/', maxAge: time });
// Redirect the user to logged in state and reload
            this.props.history.push('/', { showLogin: false });
            window.location.reload();
        }
        else {
        // Handle errors
    }
}
```

4\.  To access the value set to the cookie, we make use of the _get_ function. To check the logged in state of the User we check if _get_ method is returning a null value or an undefined value, this helps in maintaining the User behaviour at every point in the application.
```
if(cookies.get('loggedIn')===null||
    cookies.get('loggedIn')===undefined) {
    // Handle User behaviours do not send chat queries with access token if the cookie is null
    url = BASE_URL+'/susi/chat.json?q='+
          createdMessage.text+
          '&language='+locale;
  }
  else{
   //  Send the messages with User’s access token
    url = BASE_URL+'/susi/chat.json?q='
          +createdMessage.text+'&language='
          +locale+'&access_token='
          +cookies.get('loggedIn');
  }
```

5\. To delete the cookies, we make use of the remove function, which deletes that cookie. This function is called while logging the user out of the application.
```
cookies.remove('loggedIn');
this.props.history.push('/');
window.location.reload();
```
Here’s the full code in the repository. Feel free to contribute:[https://github.com/fossasia/chat.susi.ai](https://github.com/fossasia/chat.susi.ai)

### Resources

*   Package - [https://www.npmjs.com/package/universal-cookie](https://www.npmjs.com/package/universal-cookie)
*   Important README - [https://github.com/reactivestack/cookies/blob/master/packages/universal-cookie/README.md](https://github.com/reactivestack/cookies/blob/master/packages/universal-cookie/README.md)
*   Github repository - [https://github.com/reactivestack/cookies/tree/master/packages/universal-cookie](https://github.com/reactivestack/cookies/tree/master/packages/universal-cookie)