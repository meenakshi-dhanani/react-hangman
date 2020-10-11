## Tutorial: Intro to React Hooks

This tutorial assumes knowledge of React state and lifecycle concepts.

### Before We Start the Tutorial
We will be building a small game in this tutorial. This is a practical way of getting accustomed to building react functional components using hooks. We will be walking through each section of this tutorial along with the code snippets, so that you can follow along as you build your game. 

This tutorial is divided into the following sections:

- **Setup for the Tutorial** will equip you with the starter code
- **Overview** will delve into basics of hooks with some history
- **Building the Game** will use the most common hook in React development
- **Adding a Time Limit** will extend the game to add a time limit
- **Wrapping Up** will discuss extensions and conclude

You can follow along till you build a basic version of the game to get an understanding of the hooks along with some hands-on.

#### What are We Building?
In this tutorial, we'll build the an interactive hangman game using React hooks.

Hangman is a classic game in which the player has to guess a word one letter at a time. You can play with the following to get comfortable with the game.

<Hangman />

There are several rules that can be applied to the game to add more complexities, but we will be focussing on completing just the first iteration of the game. You are encouraged to experiment and extend this solution for more complex use cases suggested in the extensions section.

#### Prerequisites
We'll assume you have used React before and are familiar with creating components, state management and lifecycle methods. 
We are also using features from ES6 - arrow functions, const, let statements. You can check (Babel REPL)[https://babeljs.io/repl/#?presets=react&code_lz=MYewdgzgLgBApgGzgWzmWBeGAeAFgRgD4AJRBEAGhgHcQAnBAEwEJsB6AwgbgChRJY_KAEMAlmDh0YWRiGABXVOgB0AczhQAokiVQAQgE8AkowAUAcjogQUcwEpeAJTjDgUACIB5ALLK6aRklTRBQ0KCohMQk6Bx4gA] to understand what ES6 compiles to. 
Note, we are using hooks in this tutorial, since hooks have been introduced in React version 16.8, you would need to have 16.8 as the min. React version for this tutorial. 

### Setup for the Tutorial
Let's get started. 
We want to first create a react app. We can either create it from scratch or use create-react-app to reduce boilerplate code. In this tutorial we'll be using (create-react-app)[https://reactjs.org/docs/create-a-new-react-app.html]. 

```
npx create-react-app react-hangman
cd react-hangman
npm start
```

The above snippet will create a React app with a simple App component. For this tutorial, we won't be focussing on styling and testing the component, so let's go ahead and delete the `App.css` and `App.test.js` files. Now we can simply edit the `App.js` to include `Hangman` component. The `Hangman.jsx` is what we're going to focus on building in this tutorial. 

`App.js`
```
import React from 'react';
import Hangman from './Hangman';

const App = () => <Hangman />

export default App;
```

View the full code as this point - https://github.com/meenakshi-dhanani/react-hangman/commit/3df118d7a3cbf0bf63466d9ad17a2abb39ac9a23

### Overview

Now that you're all set, let's first get an overview of React hooks.

#### What are React Hooks?
Prior to 16.8, class components in React were used to manage state and had logic distributed across lifecycle methods. Functional components were used to extract out some common UI. With React hooks, you can now hook into your functional components state and logic that would earlier be spread across lifecycle methods. Related logic can now be in one place, instead of being split. Logic can also be shared across components by building custom hooks. 

<Diagram />

### Building the Game

As part of the first iteration, we want a secret word to be displayed, let's say we mask all the letters with __ and we need all alphabets A-Z to be listed, so that the player can select a letter and if that letter is part of the secret word it will reveal itself. 

Let's say the secret word is "HANGMAN". Then the following expression should mask the secret word as `_ _ _ _ _ _ _`

```
"HANGMAN".split("").fill("_").join(" ")
```


Let's start with a basic layout:

`Hangman.jsx`
```
import React from 'react';

export default function Hangman() {
	const word = "HANGMAN";
    const alphabets = ["A", "B", "C", "D", "E", "F", "G",
        "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R",
        "S", "T", "U", "V", "W", "X", "Y", "Z"];

	return 	<div>
			<p>{word.split("").fill("_").join(" ")}</p>
			{alphabets
			.map((alphabet, index) => 
			<button key={index}>{alphabet}</button>)}
			</div>
}
```


<Display what the above code generates />

Our next step, would be to click on an alphabet and guess if the letter is a part of the word. If the letter is indeed a part of the word it would show up and if it isn't it would not reveal itself. For this we need to persist all the letters that are correctly guessed so that they are displayed as part of the secret word. Now we have a use case for persisting data across a component re-render. This calls for the need of state. Let's have a look at how we can infuse state using the State hook in React.

#### The State Hook
We can use the state hook to inject state into functional components in React. This state will be preserved across re-renders of the component. The `useState` is a hook that we can use. The `useState` returns a pair having the current value for the state and a function that let's you set the state. In class components, we used to do something similar with `this.setState`. You can use multiple `useState` in a component for different values that need to be preserved. 

We need to persist correctGuesses for the Hangman component. Let's use the useState hook. We modified the word to display __ for all letters not guessed yet.

```
import React, {useState} from 'react';

export default function Hangman() {
	const word = "HANGMAN";
    const alphabets = ["A", "B", "C", "D", "E", "F", "G",
        "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R",
        "S", "T", "U", "V", "W", "X", "Y", "Z"];
	const [correctGuesses, setCorrectGuesses] = useState([])	

	const maskedWord = word.split('').map(letter => 
	correctGuesses.includes(letter) ? letter : "_").join(" ");
	
	return 	<div>
			<p>{maskedWord}</p>
			{alphabets
			.map((alphabet, index) => 
			<button key={index} onClick={() => {
                if (word.includes(alphabet)) {
                    setCorrectGuesses([...correctGuesses, alphabet])
                }
            }}>{alphabet}</button>)}
			{!maskedWord.includes("_") && <p>You won!</p>}
			</div>
}
```

The above code generates the following:

<Component />

### Adding a Time Limit
Now that we have a fair working solution, let's add some rules to this game. We'll have a max time limit as 2 minutes for the word to be guessed, if the word is not guessed within 2 minutes, we'll display "Game Over". 

We will need to inject a timeout in this case. The timeout will affect the results of this game. Let's look at the effect hook to understand how we can add the logic of this timeout inside our component. 

#### The Effect Hook
The effect hook is another one of the most commonly used hooks in React. It takes in a function(effect) that runs when any one of it's dependant variables are changed. The effect (short for side effect) hook, is used to manage any side effects on the component - manipulating DOM elements, fetching data, subscriptions,etc. In our case, we will use the `useEffect` to set a timeout. The `useEffect` runs by default for every component render, unless we mention `[]` as it's parameter, in which case it runs only during the first render of the component. 

```
import React, { useEffect, useState } from 'react';

export default function Hangman({duration = 120000}) {
    const word = "Hangman".toUpperCase();
    const alphabets = ["A", "B", "C", "D", "E", "F", "G",
        "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R",
        "S", "T", "U", "V", "W", "X", "Y", "Z"];
    const [correctGuesses, setCorrectGuesses] = useState([])
    const [timeUp, setTimeUp] = useState(false);

    useEffect(() => {
        const timeout = setTimeout(() => {
            setTimeUp(true);
        }, duration);

        return () => clearTimeout(timeout);
    }, [])


    const maskedWord = word.split('').map(letter => correctGuesses.includes(letter) ? letter : "_").join(" ");
    return (
        <div>
            <p>{maskedWord}</p>
            {alphabets.map((alphabet, index) => <button key={index} onClick={() => {
                if (word.includes(alphabet)) {
                    setCorrectGuesses([...correctGuesses, alphabet])
                }
            }}>{alphabet}</button>)}
			{timeUp ? 
			<p>You lost!</p> : 
			!maskedWord.includes("_") &&  <p>You won!</p>}
        </div>
    );
}

```

Notice how we are preserving the state of timeUp using `useState`.  In the second parameter of `useEffect` we mention `[]`, so the timeout is set only during the first render of Hangman. At the end, when the component unmounts, since the game gets over, we clear up the effect, in `return () => clearTimeout(timeout)`. This can be used to unsubscribe, clear up resources used in the effect. 


### Wrapping Up
Congratulations! You have a hangman game that :

 - Let's you play hangman
 - Has a time cap for you to guess
 
 We hope you've got a hang(pun-intended) of the basic hooks. 
 
The tutorial was an attempt to get you started on react hooks. We would further encourage you to explore more hooks, eg. useContext, useHistory, create your own custom hooks. etc. Check out detailed explanation on hooks here - https://reactjs.org/docs/hooks-overview.html

 There are a lot of rules that can be applied and the game can be further extended. It will be a good exercise for you to try your hand at these functional components using hooks. 

 - Max number of guesses allowed can be 6
 - Display time left on timer
 - Limiting guesses on vowels
 - Fetch a list of words based on themes


You can find the code sample used here in this repo. Feel free to write to us at meena24dhanani@gmail.com for any feedback. You can also submit PRs that extend the game. 

