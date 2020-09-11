---
layout: post
title:      "Building Tetris in JS – The Rotation Problem and Asynchronous Functions"
date:       2020-09-11 13:48:34 -0400
permalink:  building_tetris_in_js_the_rotation_problem_and_asynchronous_functions
---


I’ve been learning Javascript over the last month as part of the Flatiron Software Engineering program. My love-hate relationship with Javascript has gone more towards the love side as I’ve gotten a better understanding of its quirks. For example, understanding hoisting and that Javascript separates compilation and execution at runtime goes a long way towards being able to understand some of the nasty bugs (or lack of bugs) that Javascript will throw if I don’t structure my code well.

For my Javascript/Rails API portfolio project, I decided to build the classic game, Tetris. My other portfolio projects have been CMSs of one form or another. I wanted to build something more fun. Tetris has also been a good way for me to delve into some of the aspects that make Javascript unique, like DOM manipulation and event handling in the browser. 

Most of the project went swimmingly. I set up a representation of the Tetris board with a 12×24 nested array, and displayed the board on the DOM with a table element. I built out separate classes to represent cells and pieces, and built child classes of the Piece class to represent the different types of pieces. Along the way, I came across two main challenges:


*Solving the Rotation Problem*
Perhaps more of a math problem than a software engineering problem, getting a piece to rotate around a pivot point on a grid is surprisingly difficult. I started by trying to hard code the permutations of positions a piece would inhabit as it rotated. Although not a long term solution, I thought this would at least give me a sense of the patterns involved with rotation. I quickly realized that the permutations were too many to get very far. 

![tetris LPiece](https://drive.google.com/uc?export=view&id=1D0dwGQQgmN9Sy8rYXdrDfESmUunjOGf6)
![tetris rotated LPiece](https://drive.google.com/uc?export=view&id=13-UC1PzwplMTX5ixzbHHLmHJBT-bYBc3)


For example, the piece on the left might be represented by the array `[[1, 3], [2, 3], [2, 2], [2, 1]`. Assuming the only difference with the piece on the right is that it’s been rotated clockwise, it would be represented by the array `[[1, 2], [1, 3], [2, 3], [3, 3]]`. Here, the 4 nested arrays each represent a ‘cell’ with x and y coordinates in the ‘piece’. The challenge of the rotation problem is figuring out how to abstractly derive the coordinates in the second array given the first array. It becomes more difficult when you take into account the other types of pieces in Tetris.


To solve this problem, I began by getting out a piece of paper, picking one piece type, drawing different rotation positions and listing the associated coordinates of each position. I tried to identify which variables the end coordinates were dependent on. It became clear that new coordinates were dependent on a combination of the old coordinates and the coordinates of the pivot piece. In hindsight, I’d like to say that I set up a system of equations and solved it mathematically, but it was a little more trial and error. After tinkering for a bit, I knew that the equation for deriving new x and y coordinates for a given cell should look something like:
```
	xnew = xold  +|-  (yold  +|-  ypivot) +|- (xold +|- xpivot)
	ynew = yold  +|-  (xold  +|-  xpivot) +|- (yold +|- ypivot)
```

The nice thing about software engineering is that it’s pretty easy to quickly test out different solutions. I created a function that generated new arrays based on old arrays given these equations. I made some intelligent guesses about where to put the pluses and minuses and then tinkered until the output of the function matched the expected outcome.

In the end, I had a function that returned the end positions of an arbitrary piece after a clockwise rotation:

```
prepRotation(){
    const pivot = this.cells[1];
    function xPivot(cell){
        return pivot.y - cell.y + pivot.x 
    }
    function yPivot(cell){
        return pivot.y - pivot.x + cell.x
    }
    return this.cells.map(cell => {
        return {x: xPivot(cell), y: yPivot(cell)}
    });
} 
```
 
Here, `pivot` represents the cell that the rest of the cells are pivoting around, and xPivot and yPivot return the x and y coordinates of a cell after rotating.


*Login, Saving and Loading with Asynchronous Functions*
The second main challenge I came across was making sure someone was logged in before saving or loading a game. In particular, I wanted to write code that both honors the Single Responsibility principal of SOLID design and takes advantage of asynchronous functions. In other words, the “save” function should only save a game, and the “login” function should only login a user, BUT I needed to make sure these events occurred in a particular sequence: (1) the user clicks the save button, (2) then a login form is displayed, (3) then the user fills in and submits the form, (4) then a login request is sent to a backend, (5) then a response from the backend is received, and (6) if the login was successful configure and send another request to the backend to save the game. 

This isn’t too difficult to do if you chain some `fetch` and `then` statements together, something like: 

```
saveButton.addEventListener(‘click’, ()=>{
	  displayLogin();
	  fetch(‘user_post_url’, configObj)
		    .then (resp => resp.json())
		    .then (json => {
		    fetch(‘game_post_url’, formatGame(json);)
	  })
})
```

There are two big problems with this code: (1) all the login and save logic is in the same function, and it’s hard to separate it out because the save logic gets triggered by the `then` statement which is dependent on the login logic. (2) There’s no logic making sure that the user actually fills out and submits the form before sending off the first fetch request. Not only is saving a game dependent on a login fetch request, it’s dependent on a login DOM event.

My solution to this problem was to create a couple of intermediary functions that handle the interaction between logging in, saving and loading. 

```
  function handleLoad(){
    if (loggedIn){
      loadGame();
    } else {
      pauseGame();
      displayLogin();
      afterLogin(loadGame);
    }
  }

function afterLogin(callback){
    const interval = window.setInterval(()=>{
      if (loginRequest){
        loginRequest.then(()=>{
          callback();
          window.clearInterval(interval);
        })
      }
    }, 1000);
  }
```

In particular, after displaying the login, `afterLogin()` sets an interval that regularly checks to see if the user has submitted the login form yet. It knows this because the variable `loginRequest` represents the fetch object. If there is a login request then a callback, either `saveGame()` or `loadGame()` gets called.  By passing around the fetch request as a variable and using setInterval() to regularly check for the login event, I was able to both honor the Single Responsibility principal and take advantage of Javascript’s asynchronous functions. 
