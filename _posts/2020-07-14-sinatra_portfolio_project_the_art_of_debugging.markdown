---
layout: post
title:      "Sinatra Portfolio Project – The Art of Debugging"
date:       2020-07-14 12:15:54 -0400
permalink:  sinatra_portfolio_project_the_art_of_debugging
---


We recently finished the Sinatra module at the Flatiron Software Engineering program, including SQL and Active Record. For my portfolio project, I built a CMS (Content Management System) for creating Github portfolios. My Sinatra app pulls a user’s Github repos from the Github API and allows them to select specific repos to include in their portfolio. They can then dynamically generate html and css for that portfolio that could be included, for example, in their personal website. If you want to check out my portfolio project you can find the repo [here](https://github.com/charlie763/github-portfolio-cms) and a heroku hosted version [here](https://infinite-cove-25560.herokuapp.com)

During this project, I got stuck on some particularly nasty bugs. Despite my best googling efforts, there was no clear answer, and I struggled a lot before figuring them out. I like to think that such debugging sessions are valuable as they build my confidence and make me a better debugger for the future. 

**Bug #1: Sinatra and Active Record Version Mismatch Causes Rake DB:Migrate to Fail Silently**

Throughout the coding labs provided by Flatiron, the Active Record gem was always specified to be version 5.2. When I was setting up my initial project I wanted to use the newest versions of all my gems to see what happened. 

Everything was going swimmingly as I set up my project and got it to “Hello World” status. I decided to build my file structure mostly from scratch, instead of using the Corneal gem because I wanted practice and better understanding of initial configuration steps. In the environment file, I configured the database like I had seen it done in previous labs:

```
ActiveRecord::Base.establish_connection(
	adapter: ‘sqlite3’,
	database: ‘db/development.sqlite3’
)
```

I created my first migration file, but when I typed in the command rake db:migrate at the terminal, NOTHING HAPPENED. No errors. Nothing but silent failure….

I looked at previous labs. They all had the same code. I googled my heart out, but the best I could find was a stackoverflow question by someone who seemed to have the same problem. I saw that this person found that if they switched the ActiveRecord gem to an earlier version things magically fixed themselves. This solution worked for me, but I wasn’t satisfied. Did Sinatra just not work with Active Record version 6.0? I finally found my answer in the Sinatra documentation. I saw Sinatra was recommending the following code for configuring the database:

```
set :database, {
	adapter: ‘sqlite3’,
	database: ‘db/development.sqlite3’
}
```

They had changed the syntax for configuring the database. There was no mention of the old syntax, and no error was thrown if you used it.

**Bug #2: Loading CSS and Static Files through href in Sinatra**

When making my css files I copied a pattern I had seen of putting them in a public/stylesheets folder at the root directory. In my layout.erb file I put:

```
<link rel=“stylesheet” href=“/public/stylesheets/default.css”>
```

From my understanding the href=“/...” would tell the browser to look for the default.css file in the public/stylesheets folder relative to the root directory. Yet, my css was nowhere to be found when the pages got rendered. Again, I googled. But most answers were of the sort, use href=“/...” to reference relative to the root and href without the “/” to reference files relative to the current path.

Again, the answer was in the Sinatra documentation. Apparently, Sinatra, by default looks for static files in the ‘public’ folder. When I wrote: 

```
<link rel=“stylesheet” href=“/public/stylesheets/default.css”>
```

Sinatra was looking for default.css in the /public/public/stylesheets directory, which didn’t exist. Changing the code to:

```
<link rel=“stylesheet” href=“/stylesheets/default.css”>
```

solved my problem. I later learned that you also need to explicitly ```set :public_folder, ‘public’``` in the configuration of your app controller or else heroku will not be able to find your static files when the app is deployed. 

**Lessons**

The biggest lesson I learned is, when there aren’t informative errors or answers on the internet, read documentation very closely. Develop hypotheses for what might be wrong based on the documentation and methodically test out those hypotheses. If all else fails, look at source code and try to understand what’s actually going on behind the scenes. 


