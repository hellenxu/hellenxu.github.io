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
The Firebase Command Line Interface(CLI) will allow you to deploy your Actions project to Cloud Functions.

#### Step One: Firebase tools and configuration

+ Install Firebase cli

```shell
npm -g install firebase-tools
```

+ Login Firebase

```shell
firebase login
```

+ Choose your project and install dependencies

```shell
firebase use <PROJECT_ID>
```

We can use firebase list to get all your project ids.

![Firebase List](/imgs/20181231_firebase_list.png)

Then install all dependencies by using:

```shell
npm install
```

+ Deploy

```shell
firebase deploy
```

+ Retrieve deployment url and set it in Dialogflow
Open Firebase Console, Functions -> Dashboard:

![Retrieve deployment url](/imgs/20181231_firebase_deploy_url.png)

After that, we go to Dialogflow console and set it for webhook.

![Dialogflow webhook](/imgs/20181231_dialogflow_webhook.png)

Then the connection between firebase and dialogflow is established, and we can just update the local js file to handle all those intents.

#### Step Two: A simple conversation
Before starting a simple conversation, it would be better to take a look at the structure of a local firebase project first.
+ Structures of a Firebase Project

A basic firebase project mainly contains three parts:
1) index.js -- is to handle all intents;
2) package.json -- include all dependencies of this project;
3) node_modules -- a directory, including downloaded dependencies.


+ Initialization of a firebase project

Here we use Visual Studio Code to complete initialization.
1) Open the workplace in terminal.

![Open terminal](/imgs/20181231_firebase_project_terminal.png)

2) firebase init

![Firebase Init](/imgs/20181231_firebase_init.png)

Then follow all steps and choose options based on your requirements.

+ A simple Conversation
— Greetings + deep link：Google assistant greetings + what’s your favourite colour?
— handle intent：based on the user input, we reply the length of the colour as the lucky number
— close conversation: Goodbye

+ Implement a simple conversation
First of all, import classes.

```js
const {
    dialogflow,
    Permission,
    Suggestions,
    BasicCard,
} = require('actions-on-google');

```

[index.js — ask for name, so we can show the user name next time they talk to us]

```js
app.intent('Default Welcome Intent', (conv) => {
    const name = conv.user.storage.userName;

    if(!name) {
        conv.ask(new Permission({
            context: 'Hi there, to get to know you better',
            permissions: 'NAME',
        }));
    } else {
        conv.ask(`Hi again, ${name}. What\'s your favorite color?`);
    }

});
```


[index.js — based on whether we get the permission of getting user names, we ask users in different ways.]

```js
app.intent('actions_intent_PERMISSION', (conv, params, permissionGranted) => {
    if(!permissionGranted) {
        conv.ask('Ok, no worries. What\'s your favorite color?');
        conv.ask(new Suggestions('Blue', 'Red', 'Green'));
    } else {
        conv.user.storage.userName = conv.user.name.display;
        conv.ask(`Thanks, ${conv.user.storage.userName}. What\'s your favorite color?`);
        conv.ask(new Suggestions('Blue', 'Red', 'Green'));
    }
});
```


[index.js — tell users their luck numbers]

```js
app.intent('favorite color', (conv, {color}) => {
    const luckyNumber = color.length;
    const audioSound = 'https://actions.google.com/sounds/v1/cartoon/crazy_dinner_bell.ogg';
    // Respond with the user's lucky number and end the conversation.
    if (conv.user.storage.userName) {
        const name = conv.user.storage.userName;
        conv.ask(`<speak>${name}, your lucky number is ${luckyNumber} .<audio src="${audioSound}></audio></speak>`);
    } else {
        conv.ask(`<speak>Your lucky number is ${luckyNumber} .<audio src="${audioSound}></audio></speak>`);
    }

    conv.close('Good bye, see you next time.');
});
```



### Part Four: References
[Codelabs](https://github.com/actions-on-google/codelabs-nodejs)

[Guides](https://codelabs.developers.google.com/codelabs/actions-1/#0)

[SSML](https://developers.google.com/actions/reference/ssml)

[Conversation Components](https://designguidelines.withgoogle.com/conversation/conversational-components/chips.html)

[Simulator](https://developers.google.com/actions/tools/simulator)

[Helpers](https://developers.google.com/actions/assistant/helpers)
