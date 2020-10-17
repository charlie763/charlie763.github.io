---
layout: post
title:      "Sessions with a React-Redux Frontend and Rails API Backend"
date:       2020-10-17 16:13:21 +0000
permalink:  sessions_with_a_react-redux_frontend_and_rails_api_backend
---


For my last portfolio project at the Flatiron Coding bootcamp I built an app that allows parents to crowdsource online educational resources and organize them (check it out on github[https://github.com/charlie763/edu-source]). Think Pinterest for online educational resources. 

One of the bigger challenges I took on was managing sessions with a decoupled frontend/backend. Rails makes sessions pretty easy to handle, IF EVERYTHING IS IN RAILS, but it gets a little more complicated when you use a different system for managing frontend behavior, particularly when interaction between the frontend and backend is asynchronous.

Sadly, I was unable to find a definitive guide that covers all the things you need to know to make this work. Instead, I pulled together a solution based on three different sources:
    1. [Mike Clark’s overview](https://pragmaticstudio.com/tutorials/rails-session-cookies-for-api-authentication) of the general flow you want to have
    2. [Kailana Kahawaii’s overview](https://dev.to/kahawaiikailana/rails-api-quickstart-guide-with-postgressql-and-jwt-tokens-3pnk) on setting up JSON Web Tokens (JWT) with Rails 
    3. [JMFurlott’s guide](https://www.jmfurlott.com/handling-user-session-react-context/) on using JavaScript Cookies to store and utilize session data in React 

!!CAVEAT!! - This is a beginners solution. I’d guess that this is not the most secure solution. Rather it’s a moderately secure solution that I hope will be easier to understand for beginners just learning about how to implement secure sessions.

*Step 1: Set up JWT in the Rails API Backend*
First things first, you need to install the JWT gem. So, in your Gemfile put `gem "jwt", "~> 2.2"` and then `bundle install`

Next, you need to generate and securely store a SESSION_SECRET, a cryptographically secure key you’ll use to generate tokens. There are multiple ways to both generate and securely store this key. For example, the article reference above uses a JWT generator website and `rails credentials`. I used Ruby’s `SecureRandom` gem to generate the key and the `dotenv` gem to store it my Rails environment. It’s helpful to set up your storage environment first. Put `dotenv-rails` in your gemfile, `bundle install`, then create a .env file in your root directory and put `.env` in your `.gitignore` file (so you don’t upload your secret key to github). Now, after installing `securerandom` you can run from the terminal:

`ruby -e "require 'sysrandom/securerandom'; puts SecureRandom.hex(64)"`

Copy and paste the key this command outpus and save it in your `.env` file like so:
`SESSION_SECRET = [insert the key you just generated]`

The rest, I blatantly stole from the article above, minus swapping out the return value for the `jwt_key` function to return the key from where I had stored it. You can put the code below in your `ApplicationController` that other controllers inherit from, or you could create a helper module. I did the former.

```
def jwt_key
	ENV['SESSION_SECRET']
end

def issue_token(user)
	JWT.encode({user_id: user.id}, jwt_key, 'HS256')
end

def decoded_token
	begin
		JWT.decode(token, jwt_key, true, { :algorithm => 'HS256' })
	rescue JWT::DecodeError
		[{error: "Invalid Token"}]
	end
end

def token
	request.headers['Authorization']
end

def user_id
	decoded_token.first['user_id']
end

def current_user
	user ||= User.find_by(id: user_id)
end

def logged_in?
	!!current_user
end
```

Now, when creating a user you can simply call the `issue_token` function, and ,when authorizing a user, calling the `current_user` function will only work if there is a valid token.

*Step 2: Setting up a Session/User Store*
There are probably a variety of ways to do this. I used Redux and created a `usersReducer` function that set up and handled the store for the user data model. My code assumes some working knowledge of how Redux works, but you could also save this data in state if you’re not using Redux. I briefly used the `react-redux-session` package to implement this but found that it forced me into abnormal patterns more than it helped. The import piece of information is that the store I set up had the following data structure:
```
state = {
	...[your other data models],
	user: { 
		current: {}, 
		valid: true, 
		authCompleted: false, 
		errors: {}
		} 
```

Here, `user.current` will store a valid user object, `user.valid` signifies the result of authenticating and authorizing a user, and `user.authCompleted` tells you whether or not an auth process has been conducted yet to see if there is a valid user. I defaulted `user.valid` to true to avoid some unintended redirects to the login screen. However, there is probably a more elegant solution to that, and I suspect it’s safer to default user.valid to false rather than setting it as such whenever an auth process starts. 

*Step 3: Login/Signup your User*
I won’t go through everything here. I’ll just give an outline. The important pieces are to install the `js-cookie` package, import that functionality, set the cookie with the value of the JWT that is returned from your Rails API backend, and then update your store accordingly. Some of the code that’s needed to do this:

```
`import * as Cookies from ‘js-cookie’`
```

After sending a fetch request to your API backend, if the user data is valid, set the cookie. The JSON my sessions controller sent back looked something like `{ valid: true, user: [user data here], token: [generated api token here]}`. With that, I was able to set the cookie with the following code:

```
Cookies.remove("eduResourceSession")
Cookies.set("eduResourceSession", jsonData.token, { expires: 14 })
```

`eduResourceSession` refers to the name of the cookie that gets stored, but this could be anything, and the second argument in `Cookies.set` sets the value of the cookie to the token that’s returned.

The last step is then to update your store/state based on what’s returned. If the user is valid, set `user.valid` to true, and `user.current` to the user data sent back. Regardless, set `user.authCompleted` to true.

*Step 4: Redirect user to Login if not authorized*
For components in my react app that dealt with user sensitve data I inserted the following code:

```
componentDidMount(){
	this.props.authUser()
}
```

`authUser()` is a function that makes a fetch request to the API sessions controller including the token in the `Authorization` header and sets the state to `user.valid = true` if the controller returns a valid user. This includes the code:

```
let token = Cookies.get("eduResourceSession")
…
headers: {
'Content-Type': 'application/json',
Accept: 'application/json',
Authorization: token
}
```

I would then put some logic at the top of my `render()` function along the lines of  `if (this.props.user.valid){ return <Component/>} else {return  <Redirect to=’\login’/>`.  Components re-render when state changes, so when the `user.authCompleted` changes during the auth process, the component re-renders and checks again if the user is valid.

*Step 5: Authorizing your user when you make requests to the Backend API*
The last step is to authorize your user whenever you are making a request to the backend that requires them to be authorized. For my application, this was restricted to POST, PATCH, and GET requests for data that was specifically tied to the user data model. The basic outline for this step is to get the token from cookies whenever making one of these requests, send it in the authorization header, and only return data or post data from the backend if the token is valid. The code for this is very similar to the code in step 4.

*Conclusion*
That is a LOT! I hope this helps you understand the session flow using a React-Redux frontend and Rails API backend. Please reach out if you have any comments, questions or improvements!




