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

#### Permission Settings
Firstly, we need to check Google permission settings since we need those permission to test actions.
Open [Activity Controls](https://myaccount.google.com/activitycontrols), and enable three permissions as follows:
> + Web & App Activity
> + Device Information
> + Voice & Audio Activity

#### Create an Action Project
Navigate to [action console](https://console.actions.google.com/), you will see the Add/import project tab option.

![Add/Import project](/imgs/20181231_add_project.png)

After clicking 'Create Project' button, we will see the category choosing page, and at this time we can just skip this step.

![Skip Choosing Category](/imgs/20181231_skip_category.png)

Next step will be creating an action. You can find that button from the left navigation bar(BUILD -> Actions), and then select custom intent. Lastly, clicking BUILD is good enough.

![Build actions](/imgs/20181231_build_actions.png)

#### Create a Dialogflow agent
Why do we need to create a Dialogflow agent? The picture below tells you why.

![Dialogflow agent](/imgs/20181231_dialogflow_agent.png)

> + Dialogflow: A web-based service provided by Google that uses an agent to process user input. This service allows you to integrate conversational apps with the Assistant, as well as with other conversation platforms.
> + NLU: Acronym for "Natural Language Understanding". This refers to the capability of software to understand and parse user input. Developers can choose to use Dialogflow or their own NLU solutions when creating Actions.


### Part Three: Design and implement a simple conversation


### Part Four: References
[Codelabs](https://github.com/actions-on-google/codelabs-nodejs)

[Guides](https://codelabs.developers.google.com/codelabs/actions-1/#0)

[SSML](https://developers.google.com/actions/reference/ssml)

[Conversation Components](https://designguidelines.withgoogle.com/conversation/conversational-components/chips.html)

[Simulator](https://developers.google.com/actions/tools/simulator)

[Helpers](https://developers.google.com/actions/assistant/helpers)