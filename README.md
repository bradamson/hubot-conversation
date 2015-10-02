# Hubot-Conversation

Have a multi-message chat with your bot.

##How it works:

1) Make a new `Conversation` instance aware of your robot:
    
   ```javascript
   var switchBoard = new Conversation(robot);
   ```
    
This will register a `hear` listener allowing the instance to listen in on all incoming messages

2) Given an starting message, create a new Dialog instance and give the dialog choices.
  
  ```javascript
  var dialog = switchBoard.createDialog(msg);
  
  dialog.addChoice(/foo/, function(msg2){/*Do stuff*/}
  ```

The switchBoard will listen to the next message **FROM THE SAME USER** and try to match it to any of the available choices.
After a match has been found. It will clear the choices, and end the dialog.

The bot will forget about your dialog after a default timeout of 30 seconds.

##Use

On your hubot script:

```javascript

var Conversation = require('hubot-conversation');
module.exports = function (robot) {

    var switchBoard = new Conversation(robot);

    robot.respond(/clean the house/, function (msg) {
        var dialog = switchBoard.startDialog(msg);

        msg.reply('Sure, where should I start? Kitchen or Bathroom');
        dialog.addChoice(/kitchen/i, function (msg2) {
            msg2.reply('On it boss!');
        });
        dialog.addChoice(/bathroom/i, function (msg2) {
            msg.reply('Do I really have to?');
            dialog.addChoice(/yes/, function (msg3) {
                msg3.reply('Fine, Mom!');
            })
        });
    });

    robot.respond(/jump/, function (msg) {
        var dialog = switchBoard.startDialog(msg);
        msg.reply('Sure, How many times?');
        
        dialog.addChoice(/([0-9]+)/i, function (msg2) {
            var times = parseInt(msg2.match[1], 10);
            for (var i = 0; i < times; i++) {
                msg.emote("Jumps"); //We can use the original message too.
            }
        });
    });
    
    
  robot.respond(/.*the mission/, function (msg) {
        msg.reply('Your have 5 seconds to accept your mission, or this message will self-destruct');
        var dialog = switchBoard.startDialog(msg, 5000); //5 Second timeout
        dialog.timeout = function (msg2) {
            msg2.emote('Boom');
        }
        dialog.addChoice(/yes/i, function (msg2) {
            msg2.reply('Great! Here are the details...');
        });
    });
};
```

This will give you the following interactions:

```bash
hubot> hubot clean the house
hubot> Shell: Sure, where should I start? Kitchen or Bathroom
hubot> Kitchen
hubot> Shell: On it boss!
hubot> hubot clean the house
hubot> Shell: Sure, where should I start? Kitchen or Bathroom
hubot> Bathroom
hubot> Shell: Do I really have to?
hubot> yes
hubot> Shell: Fine, Mom!

```

```bash
hubot> hubot jump
hubot> Shell: Sure, How many times?
hubot> 3
hubot> ready to jump 3 3
* Jumps
* Jumps
* Jumps
```

```bash
hubot> hubot what's the mission?
hubot> Shell: Your have 5 seconds to accept your mission, or this message will self-destruct
//5 seconds later
* Boom  
```


##API

###Conversation

##### startDialog(msg, [timeout])

Returns a new Dialog object, with a default timeout of 30000ms.

**msg**: An incoming message heard / responded to by the robot
 eg: 
  
 ```javascript
 robot.respond(/foo/,function(msg){ 
    var dialog = conversation.startDialog(msg);
 })
 ```
 
 **timeout**: optional time in ms before the conversaion expires.
 The Dialog object has a default `timeout` function property which can be overriden to specify the timeout behavior
 
##### talkingTo(id)
 
 Returns the Dialog being held with a given user
 
 **id**: The Id of the user with whom a dialog is taking place.
 
 
### Dialog
 A multiple-choice message handler
 
##### receive(msg)
 
 Process an incoming message to see if any choice matches it.
 
 **msg** A hubot message
 
##### addChoice(regex, handler)
 
 Adds a listener choice to this Dialog. This works just like hubots `hear` listener. 
 If a message is received that matches the choice `regex`, the handler will be executed.
  
 **regex** a regular expresion that will be aplied to the incoming message from the receive function
 **handler**: function(message),  A function that is executed against a successfully matched message. The `match` property of the original 
 message is replaced by the match against this regex, so you can retrieve matched groups.

##### getChoices()
 
 Returns the array of choices
 
##### reset()

 Clears the choices and the timeout
 
##### timeout -> function(message){ }
Default timeout handler.  You can override this property with your own function.
