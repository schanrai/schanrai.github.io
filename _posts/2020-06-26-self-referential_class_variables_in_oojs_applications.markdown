---
layout: post
title:      "Self-referential class variables in OOJS applications"
date:       2020-06-26 00:33:12 -0400
permalink:  self-referential_class_variables_in_oojs_applications
---


You’ve all heard it before - global variables are evil! A surefire route spaghetti-code hell and bugville central….not to mention a dead giveaway of a beginner's mindset.

When we talk of a global variable in Javascript, we are describing an entity which [”declared outside a function, becomes GLOBAL….it has global scope: All scripts and functions on a web page can access it”](https://www.w3schools.com/js/js_scope.asp). Essentially, the quality and appeal of a global variable is universal accessibility. 

## Freedom has a cost

Unfettered freedom of access usually comes at a cost, and this is no different when dealing with global variables. The most problematic issues that occur with global variable usage are:

**Naming collisions **

The chances of accidentally using a global variable that's already been declared, increases with the number of contributers (as in a team environment) and the complexity of the application. The larger your codebase gets, the more difficult it becomes to select and remember unique names that will never overlap. The possibility of accidentally overwriting variables is especially problematic in the case where you accidentally pick a name that might be provided natively by the browser later on. 

The name ‘color’, for example, is an example of this, it is a generic plain noun without any qualifiers or descriptors, so the chance of collision with an upcoming native API, or another developer’s work, is quite high.

**Encapsulation and Mutable State **

The Object-Oriented concept of “encapsulation” - where we organize data and functions within a common enclosed entity that has boundaries and controlled access - is absolutely threatened by the usage of global variables. 

Global variables break the protections afforded by ‘encapsulation’ with their ease of redefinition. In the process of running an application that has a globablly mutable state, it is all too easy to affect that state and permanently alter it without transmitting that change to other dependencies in the codebase.

This is demonstrated below, where calling the `toString` function with the argument of the global variable `num`, will change the value and type of `num` to a string. Consequently, calling `addOne` would then throw an error because `num` is no longer a number, and therefore cannot be added to 1.

```
let num = 4

function addOne(num){
  return num + 1
}

function toString(item){
  item = "string" 
}

toString(num)
```



## Best practices for avoiding globals

There are many  approaches and patterns that have been well-documented and disseminated as alternatives to global variable usage. It is beyond the scope of this post to go into detail and explain the inner workings of each of them, but a cursory mention is useful. Some of the more popular ones include:

* Working locally: the easiest code to maintain is code in which all variables are defined locally. The reason being that the scope of local variables are limited to the scope of the function or block in which they are defined. More info on this can be found [here](https://www.bitdegree.org/learn/javascript-global-variable).

* Use `const` to declare your global variables: variables declared or initialized with `var`, `let` and `const` at the top level of programs and functions, are available globally to all elements in the application. However, `const` has more stringent rules governing its usage, particularly around reassignment. Therefore, it less susceptible to issues such as name collisions or overwriting.

* Defining variables in namespaces and modules:  the [ 'one global' approach](https://stackoverflow.com/questions/45652069/entire-js-inside-one-global-variable) 
 or the[ 'revealing module' pattern](https://www.w3.org/wiki/JavaScript_best_practices#Avoid_globals) are popular and well documented as alternatives. 
 
*  Wrap it all up in closures:  a closure variable is a variable declared within the execution of a function. The approach documented [here](https://stackoverflow.com/questions/1841916/how-to-avoid-global-variables-in-javascript)  decribes how you would wrap your code in a closure and manually expose only those variables that you need to access globally.


However, these approaches fall short  in Object-Oriented applications, and specifically in the context of needing access to common variables and functions across classes. This was certainly my experience and need when I built [Whatfishy](https://github.com/schanrai/fish-id-quiz-frontend) - a Javascript frontend/Rails API quizlet application which helps you learn and identify saltwater fish native to Florida.  Game applications generally require constant access to player or game object variables. Having the ability to store and pass these objects around in order to progress the game is critical and needs to happen in a highly synchronous manner.

The temptation to attach objects to the global scope is especially high in a Single Page Application when you can have a multitude of scopes in different modules for different modal views all trying to play together nicely with the end goal of seamless state management. Giving into temptation within a SPA game application, looks something like this:

```
//index.js

const BASE_URL = "http://localhost:3000/api/v1";
const startBtn = document.querySelector("#start");

let newGame;

function startGame() {
  startBtn.addEventListener("click", () => {
    newGame = new Game();
  });
}

contBtn.addEventListener('click', () => {
  newGame.newTurn() 
}



//game.js

class Game {
  constructor() {
    this.score = 0;
    this.questionCounter = 0;
    this.questions = {};
  }

newTurn(){
    contBtn.classList.add('hide')
    const counter = document.querySelector('#question-count')
    const image = document.querySelector('img')
    ++this.questionCounter //state change
    counter.firstElementChild.innerText = this.questionCounter
    mainPrompt.innerText = "What fish is this?"
    image.src = `${this.questions.correctChoice.image_url}`
  }
}
```

In the code snippet above, you see that ‘let’ is selected to declare the global variable of `newGame` because ‘let’ is reassignable. This allows a new game object to be instantiated every time a logged-in player wishes to start a new game or play again, and is accessible from anywhere in the codebase . The `newGame` variable contains a game object, which gives us access to all the functions that are encapsulated in the Game class such as `newTurn()`. Whilst we are somewhat protected from name collisions through the use of `let`, we are certainly not protected from mutability. 

## Self-referential class variables

A much cleaner and safer approach is to setup a **self-referencing class property** or variable, which effectively ’stores’ the game object within the instance. This might sound like a convoluted process but it actually requires a very minimal amount of additional code and it allows us to maintain separation of our concerns. It also prevents any further pollution of the global namespace. 

What do we mean by a **self-referencing class** though? Arguably the most succinct definition I have come across states that a [“self-referencing class contains a reference member that refers to an object of the same class type”.](https://flylib.com/books/en/2.255.1/self_referential_classes.html) 

Rewriting the code above with this ‘meta-like’ approach looks like this:


```
//index.js

const BASE_URL = "http://localhost:3000/api/v1";
const startBtn = document.querySelector("#start");


function startGame() {
  startBtn.addEventListener("click", () => {
    Game.newGame = new Game();
  });
}

contBtn.addEventListener('click', () => {
  Game.newGame.newTurn() 
}

//game.js

class Game {
  constructor() {
    this.score = 0;
    this.questionCounter = 0;
    this.questions = {};
    Game.newGame = undefined;
  }

newTurn(){
    contBtn.classList.add('hide')
    const counter = document.querySelector('#question-count')
    const image = document.querySelector('img')
    ++this.questionCounter //state change
    counter.firstElementChild.innerText = this.questionCounter
    mainPrompt.innerText = "What fish is this?"
    image.src = `${this.questions.correctChoice.image_url}`
  }
}
```

The code above works because when we instanciate a new Game object, we have only declared the `Game.newGame` property/variable in the constructor function, we have not assigned a value as yet. After instanciation, we immediately assign the object instance to the `Game.newGame` property.

`Game.newGame` is a class property that gets assigned in the constructor of the game object. In so doing, we still have access to our game object within a global scope but it is encapsulated in the Game Class.  This allows us to maintain a separation of concerns. By instantiating `Game.newGame` in the constructor, we can be assured that it always points to the current game object.

In a single player game application, we want to be sure that only one game is running at any time. The semantics of a singleton class support this, and therefore the self-referential approach described above supports this functionality. The approach would also work in multi-layer context, provided that each player is playing within their own application environment.






