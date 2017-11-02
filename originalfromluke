'use strict';
var Alexa = require("alexa-sdk");
var AWS = require("aws-sdk");
var APP_ID = "amzn1.ask.skill.eb435537-3068-417b-9695-5e5e9ee909af";
var SKILL_NAME = "Game Shelf";
var WELCOME_MESSAGE = "Welcome to game shelf. What would you like to do?";
var GAME_SUGGESTION = "I suggest you play the game ";
var HELP_MESSAGE = "You can add or delete a game and ask for a suggested game to play. I can also list all your games. To quit, say cancel. What can I help you with?";
var HELP_REPROMPT = "Are you still there?";
var REPROMPT_SPEECH = "Try again.";
var EXIT_SKILL_MESSAGE = "Goodbye.";
var myData = [];
var game = "";

exports.handler = function (event, context, callback) {
    var alexa = Alexa.handler(event, context);
    alexa.APP_ID = APP_ID;
    alexa.registerHandlers(handlers);
    alexa.execute();
};

AWS.config.update({
    region: "us-east-1",
    endpoint: "https://dynamodb.us-east-1.amazonaws.com"
});

var handlers = {
    'LaunchRequest': function () {
        console.log("BEGIN start");
		storageGetGameList(this.event.session, (data) => {
			if (typeof(data) !== 'undefined' && typeof(data.Item) !== 'undefined' && typeof(data.Item.gameList) !== 'undefined') {
			    myData = data.Item.gameList;    
			}
            this.emit(":ask", WELCOME_MESSAGE, HELP_MESSAGE);
        });			
    },
    'AddAGameIntent': function () {
        console.log("BEGIN AddAGameIntent");
        var game = this.event.request.intent.slots.game.value;
        var response = "";        
        
        if (typeof(game) == 'undefined' || game == null)
        {
            response = "Please specify the name of the game when making your request.";
            this.emit(':ask', response, HELP_MESSAGE);
        }
        else {
            storageGetGameList(this.event.session, (data) => {
    			if (typeof(data) !== 'undefined' && typeof(data.Item) !== 'undefined' && typeof(data.Item.gameList) !== 'undefined') {
    			    myData = data.Item.gameList;
    			}
            });
            var index = myData.indexOf(game);

            if (index > -1) {
                response = "The game " + game + " is already in your list.";
                this.emit(':ask', response, HELP_MESSAGE);
            } else {
                myData.push(game);
                storageSave(myData, this.event.session, game, (game) => {
                    response = 'Ok ' + game + ' was added to your list of games.';
                    this.emit(':ask', response, HELP_MESSAGE);
                });
            }
        }
    },
    'DeleteAGameIntent': function () {
        console.log("BEGIN DeleteAGameIntent");
        var game = this.event.request.intent.slots.game.value;
        var response = '';
        
        if (typeof(game) == 'undefined' || game == null)
        {
            response = "Please specify the name of the game when making your request.";
            this.emit(':ask', response, HELP_MESSAGE);
        }
        else {
            storageGetGameList(this.event.session, (data) => {
    			if (typeof(data) !== 'undefined' && typeof(data.Item) !== 'undefined' && typeof(data.Item.gameList) !== 'undefined') {
    			    myData = data.Item.gameList;
    			}
            });
            var index = myData.indexOf(game);
    
            if (index > -1) {
                myData.splice(index, 1);
                storageSave(myData, this.event.session, game, (game) => {
                    response = 'Ok ' + game + ' was deleted from your list of games.';
                    this.emit(':ask', response, HELP_MESSAGE);
                });
            } else {
                response = "The game " + game + " was not in your list.";
                this.emit(':ask', response, HELP_MESSAGE);
            }
        }
    },
    'ListGamesIntent': function() {
        console.log("BEGIN ListGamesIntent");
        storageGetGameList(this.event.session, (data) => {
            var response = "";
			if (typeof(data) !== 'undefined' && typeof(data.Item) !== 'undefined' && typeof(data.Item.gameList) !== 'undefined') {
			    myData = data.Item.gameList;
				myData.sort;
                response = "Here are the games in your list: " + myData.toString();
			} else {
			    response = "You don't have any games in your list."
			}
            this.emit(":ask", response, HELP_MESSAGE);
        });
    },
    'SuggestAGameIntent': function () {
        console.log("BEGIN SuggestAGameIntent");
        storageGetGameList(this.event.session, (data) => {
            var response = "";
			if (typeof(data) !== 'undefined' && typeof(data.Item) !== 'undefined' && typeof(data.Item.gameList) !== 'undefined') {
			    myData = data.Item.gameList;
			    var dataIndex = Math.floor(Math.random() * myData.length);
                var randomGame = myData[dataIndex];
                response = GAME_SUGGESTION + randomGame + ". Have fun!";
			} else {
			    response = "You must add a game to your list before I can make a suggestion.";
			}
			this.emit(":tell", response, HELP_MESSAGE);
        });
    },
    "AMAZON.StopIntent": function () {
        this.emit(":tell", EXIT_SKILL_MESSAGE);
    },
    "AMAZON.CancelIntent": function () {
        this.emit(":tell", EXIT_SKILL_MESSAGE);
    },
    'AMAZON.HelpIntent': function () {
        this.emit(':ask', HELP_MESSAGE, HELP_REPROMPT);
    },
    'Unhandled': function () {
        this.emit("AMAZON.HelpIntent");
    }
};

function storageSave(myData, session, game, callback) {
    console.log("BEGIN storageSave");
    var dynamodb = new AWS.DynamoDB.DocumentClient();
    var params = {
        TableName: 'userGameList',
        Item: {
            'userId': session.user.userId,
            'gameList': myData
        }
    };

    dynamodb.put(params, function (err, data) {
        if (err) { 
            console.log(err, err.stack);
        } else { 
			callback(game);
        }
    });
}

function storageGetGameList(session, callback) {
    console.log("BEGIN storageGetGameList");
    var dynamodb = new AWS.DynamoDB.DocumentClient();
    var params = {
        TableName: 'userGameList',
        Key: {
            userId: session.user.userId
        },
        ProjectionExpression: 'gameList'
    };
    console.log(session.user.userId);

    dynamodb.get(params, function (err, data) {
        if (err) { 
            console.log(err, err.stack);
        } else {
            callback(data);
        }
    });
}
