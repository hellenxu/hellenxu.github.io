Google Action Basics
=====================

### Part One: How Actions work

Before everything, we need to get to know some key terms:

> + Action: An Action is an entry point into an interaction that you build for the Assistant. Users can request your Action by typing or speaking to the Assistant.
> + Intent: An underlying goal or task the user wants to do; for example, ordering coffee or finding a piece of music. In Actions on Google, this is represented as a unique identifier and the corresponding user utterances that can trigger the intent.
> + Fulfillment: A service, app, feed, conversation, or other logic that handles an intent and carries out the corresponding Action.

![How an action works](/imgs/20181231_how_an_action_work.png)

Under the hood, things work like the following ways:
> + The user's device sends the user's utterance to the Google Assistant, which routes it to your fulfillment service via HTTP POST requests.
> + Your fulfillment figures out a relevant response and sends that back to the Assistant, which ultimately returns it to the user.


### Part Two: Setup

### Part Three: Design and implement a simple conversation

### References