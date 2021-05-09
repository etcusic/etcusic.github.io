---
layout: post
title:      "Game Logic with Timer & Recursion"
date:       2021-05-09 18:12:52 +0000
permalink:  game_logic_with_timer_and_recursion
---


Today I'm creating a game with my flash cards, and so I figured I'd go through the game logic for this post. The game is very straight forward: it's essentially multiple choice with timed rounds, and the scoring takes into account the timer - for a correct answer, `score += timer`, incorrect answer `score -= timer`, and no answer `score -= 10` (each round is 10 seconds). This way the player can get a higher score depending on how fast they are but can also be penalized more heavily if they rush their answer.

Let's get right into it. This is a React application, so I'll be using the game component's state to track the round, timer, etc. I shuffle the set of cards before passing it to the component, so the set will just be accessed within props, and then there will be a `currentSet` in state which is an array representing the current 4 cards being mapped over and displayed. Here is what my constructor function looks like:
```
constructor(){
        super()
        this.state = {
            round: 0,
            score: 0,
            timer: 11,
            currentSet: [{side_a: "Side A", side_b: "Side B"}],
            inPlay: true
        }
    }
```
I'll go over the `timer` and `inPlay` attributes a bit later as we dig into the game logic more, but first let's take a look at the `setRound` function inside this container component which will allow us to set each round of the game. This function will take in 2 parameters - score and round. We begin the game at round 0, which allows us to use the round as the index for `this.props.cards` and also to check against its length to see if the we've gone through the entire set thereby ending the game. So, the first thing we do in our `setRound` function is check to see if the game should end. If not, then our function will go to work resetting the timer and setting an array of 4 cards to display. And of course, it's important to make sure that at least one of those 4 cards is in the correct order of its props and also not duplicated in the `currentSet`. Here is what the end result looks like:
```
setRound = (score, round) => {
        if (round >= this.props.cards.length){
            this.gameOver()
        } else {
            let filteredCards = [...this.props.cards.filter((card, i) => i !== round)]
            let currentSet = shuffleCards([this.props.cards[round], ...shuffleCards(filteredCards).slice(0, 3)])
            this.setState({
                round: round,
                score: score,
                timer: 11,
                currentSet: currentSet
            })
        }
    }
```
And now if we put `this.setRound(0, 0)` in our `componentDidMount` lifecycle function, then we're all set up for our display, and we can address the countdown. This will be our `playGame` function which will operate using recursion by checking to see if the timer has reached zero, calling a `setTimeout` for one second, setting the state's timer to - 1, and then calling itself again. This function also requires a bit of additional logic to check to see if the game has ended in order to break out of its recursive calls, and also if the round has ended without the player selecting an answer then it will need to adjust the score and set the next round as well. Here is what our function ends up looking like:
```
export const sleepOneSecond = () => {
    return new Promise(resolve => setTimeout(resolve, 1000));
} 

playGame = () => {
        let time = this.state.timer - 1
        if (time > 0 && this.state.inPlay){
            this.setState({ timer: time })
            sleepOneSecond()
            .then(this.timerMinusOne)
        } else if ((this.state.round + 1) < this.props.cards.length) {
            let score = this.state.score - 10
            let round = this.state.round + 1
            this.setRound(score, round)
            sleepOneSecond()
            .then(this.timerMinusOne)
        } else {
            return this.gameOver()
        }
}
```
That pretty much takes care of it. We put this function underneath the initial `setRound(0, 0)` within the `componentDidMount` and our application is off to the races! All that we're missing is the `selectCard` function which is passed down as props to each card component and the `gameOver` function. So once we have those in place, then we're good to go!
```
selectCard = (answer) => {
        let score = this.state.score
        answer === this.props.cards[this.state.round]["side_b"] ? score += this.state.timer : score -= this.state.timer
        let nextRound = this.state.round + 1
        this.setRound(score, nextRound)
}

gameOver = () => {
        this.setState({ inPlay: false })
        this.props.exitExercise()
    }
```
