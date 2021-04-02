# avalanche-tutorial


## Overview
There are 3 main parts to avalanche code you need to install: 

1) Security
2) UI components (via iframe for web or webview for mobile)
3) Tracking



## (1) Security
With any interaction you have with Avalanche API, you need to also give us the TOKEN. 
This token must be generated by using your client_id and client_secret and the following code: 

Javasccript
```sh
const getApiToken = () => {
	const client_id = 'RFNkSvxTlivttwan4YKDecFsOhAkdWnZ';
	const client_secret = 'huQLthIkJepOt1LKW1ye1ht__HuLwB_RlWPP6Q97tLGOjYiwBKHgMH5-Ln6rQdTp';
      return new Promise(function (resolve, reject) {
        var options = {
          method: "GET",
          url: "http://salty-reef-38656.herokuapp.com/events/updateTokenFromClientCreditentials?client_id="+client_id+"&client_secret="+client_secret,
          headers: { "content-type": "application/json" }
        };
        request(options, function (error, _, body) {
          if (error) {
			console.log('error update token', error);
            return "";
          }
          var jsonBody = JSON.parse(body);
		  
		  if(jsonBody.token){
			resolve(jsonBody.token);
		  } else {
			  resolve({});
		  }
        });
      });
};
```
Python
```sh
import requests
import json

url = 'https://useavalanche.us.auth0.com/oauth/token'
body = {
    'client_id': 'ertuRwxTMB6k8cV5LIZVg5nWTosIyVUH',
    'client_secret': 'Ls2aqX4pEp8npwx3yk6OeZq8ETbtdWPYqOcnepaKIHIr1u0U5b4XoDXsJaKyufeP',
    'audience': 'https://useavalanche.us.auth0.com/api/v2/',
    'grant_type': 'client_credentials'}
# headers for json response
headers = {'content-type': 'application/json'}

r = requests.post(url, data=json.dumps(body), headers=headers)
data = r.json()

token = data['token_type'] + " " + data['access_token']
```

This can be placed on backend (safest) or frontend (easiest).
### If you place it in the frontend:
with all of the calls you make (2-UI components and 3-Tracking), also give the token, which you can get by directly calling getApi token

### If you place it in the backend:
make the method avaialble to your froend by creating an endpoint, like app.get('/get-auth-token',...). Then from your frontend, when you need the token to make a call to avalanche-api, make a get request to your backend to /get-auth-token



# (2) UI compponents
For Web - use iframe
```sh


<iframe 
            sandbox="allow-top-navigation allow-scripts allow-same-origin allow-forms allow-popups" 
            height="500px" 
            width="800px"
            scrolling="no"
            src={`uri`} />


uri = ${REFERAPP_URL}?email=${email}&name=${name}&base_url=${APP_BASE_URL}&redirect_uri="http://localhost:3000/explore"&token=${authToken}
REFERAPP_URL = 'https://refer-ui-two.vercel.app/'
email = currentuser@gmail.com
name = Name CurrentUser

APP_BASE_URL = your_url
redirect_uri - your_url

REFERAPP_API_URL  = 'https://salty-reef-38656.herokuapp.com'

authToken = data you get from getApitoken

```


***note - authToken is an async function, so you need to wait for it to return before showing the iframe.
if you're getting ....token= ..&.. (empty token) then this is your issue - you are trying to show the iframe/webview before you get token from getApiToken
for example in react you can use a hook for this
const [showIframe, setShowIframe] = useState(false);
and set ShowIframe(true) once your token is returned


```sh
{showIframe && authToken &&
          <iframe 
            sandbox="allow-top-navigation allow-scripts allow-same-origin allow-forms" 
            height="500px" 
            width="800px" 
            src={`${REFERAPP_URL}?email=${email}&name=${name}&base_url=${APP_BASE_URL}&redirect_uri="http://localhost:3000/explore"&token=${authToken}`} />}

```
For Mobile - use webview
```sh
<WebView
          source={{ uri }}
          startInLoadingState={true}
          renderLoading={() => (
            <ActivityIndicator
              color='black'
              size='large'
              style={styles.flexContainer}
            />
          )}
          ref={webviewRef}
          onNavigationStateChange={navState => {
            setCanGoBack(navState.canGoBack)
            setCanGoForward(navState.canGoForward)
            setCurrentUrl(navState.url)
          }}
        />

uri = `https://refer-ui-two.vercel.app/?email=${email}&name=${name}&base_url=${link-to-app-in-appstore}&token=${authToken}`;

```

### HINT: if you can't figure out the tokens at all, but you want to test the functionality without doing the fancy tokens thing, you can also
### just specify &client_id=${your-client-id}&client_secre${your-client-secret}  instead of token=${authToken}  - we don't recommend this long term, but for the start this is OK



# (3) tracking functionality
Our tracking functionality consists of 2 functions + 1 helper function

## Helper function - retrieving the referral code from URL search parameters - 
when someone clicks on share via or email...we will send their friend a a link, which will have at the end amazingstartup.com?refAPI_ref_code=xyz... and this ref_api_code is their referral code which is used to track who invited them

For web, install this on your landing page

```
const searchParams = new URLSearchParams(window.location.search);
if(searchParams.get('refAPI_ref_code')){
		localStorage.setItem('refAPI_ref_code', searchParams.get('refAPI_ref_code')); //important to keep key as refAPI_ref_code
		document.cookie = `refAPI_ref_code=${searchParams.get('refAPI_ref_code')}`;
}
```

## Sign Up 
Do all of your suers need to sign in before making a purchase or getting a loan etc etc?

YES => call this Sign Up method right after sign up
```
avalancheBrowser.signUpMyAppSdk({ email: email, authReferApiToken: token  });
```

--this is assuming that you already stored the referral code as a cookie called refAPI_ref_code


NO, not all, some can buy etc without signing in => 
then call this Sign Up method together with your premiummEvent (described below) - specifically, right before your premiumEvent call, in a series like this:  signUpEvent THEN premiumEvent. Code is the same as above

BUt if you're ecommerce etc (anything where not ALL your users are signed in neccesarily) then you can combine 

## Premium event - this is the event that is your 'conversion' event - "if the referred user reaches this point, I want this to be considered a sucesful referral and I want this event to mean that this user or whoever referred them deserves a reward!"

this can be called both frontend (easy) and backend (secure)
### frontend 
```
avalancheBrowser.premiumEventMyAppSdkV2({ email: 'amote1234@mail.ru', token: token, authReferApiToken: token });
					resultier.then(response => {
						console.log('response', response)
					}).catch(error => {
						console.log('error', error)
					}).finally(finals => {
						console.log('finals', finals)
					})
```

where token is token we get from (1) security section. note, it's async, so make sure to await for getToken, then call this	
					
### backend JS
```
avalancheApi.premiumEventMyAppSdkV2({ email: 'amote1234@mail.ru', token: token, authReferApiToken: token });
					resultier.then(response => {
						console.log('response', response)
					}).catch(error => {
						console.log('error', error)
					}).finally(finals => {
						console.log('finals', finals)
					})
```
where token is token we get from (1) security section. note, it's async, so make sure to await for getToken, then call this


### backend Python
```
url = 'http://salty-reef-38656.herokuapp.com//events/premium_event'
body = {
    'email': 'your@user.email'
}
# headers for json response
headers = {'Content-type': 'application/json', 'authorization': token}

r = requests.post(url, data=json.dumps(body), headers=headers)
```
where token is token from the Security section




..and you;re all set!

Feel free to send us an email: elezhan@use-avalanche.com
OR you can also whatsapp our customer support team +7 776 125 06 28   link: wa.me/77761250628




