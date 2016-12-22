# Cards

A card is a resource. Let's pontificte on cards a bit and explain the context of them. Talk about how you can make a general GET to list all cards.

## Get A Single Card

```shell
curl "http://api.trello.com/1/cards/cardId?key=YOUR_API_KEY"
```

```javascript
\\ Make sure you've included client.js with your key.

Trello.cards.get("cardId")
```

> The above command returns JSON structured like this:

```json
{
    "id": "585833c0b83d090631bde62c",
    "badges": {
        "votes": 0,
        "viewingMemberVoted": false,
        "subscribed": false,
        "fogbugz": "",
        "checkItems": 3,
        "checkItemsChecked": 0,
        "comments": 0,
        "attachments": 0,
        "description": false,
        "due": "2016-12-29T17:00:00.000Z",
        "dueComplete": false
    },
    "checkItemStates": [],
    "closed": false,
    "dueComplete": false,
    "dateLastActivity": "2016-12-19T19:24:04.456Z",
    "desc": "",
    "descData": null,
    "due": "2016-12-29T17:00:00.000Z",
    "email": null,
    "idBoard": "5854175bc371136d79abbe50",
    "idChecklists": [
        "585833cdcca0206bb2e63d53"
    ],
    "idList": "58541760dada4cf7e9229a25",
    "idMembers": [],
    "idMembersVoted": [],
    "idShort": 7,
    "idAttachmentCover": null,
    "manualCoverAttachment": false,
    "labels": [
        {
            "id": "5854175b84e677fd36c87641",
            "idBoard": "5854175bc371136d79abbe50",
            "name": "",
            "color": "green",
            "uses": 1
        }
    ],
    "idLabels": [
        "5854175b84e677fd36c87641"
    ],
    "name": "This Is A Test Card",
    "pos": 65535,
    "shortLink": "31Akmw6l",
    "shortUrl": "https://trello.com/c/31Akmw6l",
    "subscribed": false,
    "url": "https://trello.com/c/31Akmw6l/7-this-is-a-test-card"
}
```

This endpoint retrieves all kittens.

<aside class="warning">This is a warning about something.</aside>

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Options | Description
--------- | ------- | ------- | -----------
actions|????| all, or a comma seperated list of (ActionTypes)[#] | Actions are things.
actions_entities|false|true,false|An action entitiy is..
actions_display|false|true, false|Actions display things like
actions_limit|50|a number from 0 to 1000|Acion limits is a number that limits
action_fields|all|all,data, date, idMemberCreator, type|The fields you'd want back from an action

<aside class="success">
Query parameters can be a list seperated by commas `api.trello.com/1/cards/?actions=disablePowerUp,updateCard`
</aside>
    
## Card Actions

A card has a number of actions attached to it. Actions are generated for a card when thing x, y, and z occurs.

### Query Parameters

Parameter | Default | Options | Description
--------- | ------- | ------- | -----------
entities (optional)|false|true,false| Lorem ipsum
display (optional)|false|true,false| Thidn soe he
filter (optional)|commentCard, updateCard:idList|all or a comma-separated list of (filter types)[Filter types]|Things
fields (optional)|all|all or a comma-seperated list of data, date, idMemberCreator, type|Fields are the actual values

```shell
curl "http://api.trello.com/1/cards/cardId/actions/?key=YOUR_API_KEY"
```

```javascript

Trello.cards.get('CardID',['actions'])
```

> The above command returns JSON structured like this:

```json
[
    {
        "id": "5859a5a16d46a575cd611cb7",
        "idMemberCreator": "58530f69bd582fbf01f4a8fc",
        "data": {
            "list": {
                "name": "Test Board",
                "id": "58583a46fdada4bcf993db96"
            },
            "board": {
                "shortLink": "TJBdSPf6",
                "name": "Test Board!!!",
                "id": "58583a421335eb9ad2d3403a"
            },
            "card": {
                "shortLink": "38UCRrH5",
                "idShort": 1,
                "name": "Thing Test Again Again",
                "id": "58583a4a8293d411ec44abf5"
            },
            "text": "Thing\n"
        },
        "type": "commentCard",
        "date": "2016-12-20T21:41:53.613Z",
        "memberCreator": {
            "id": "58530f69bd582fbf01f4a8fc",
            "avatarHash": null,
            "fullName": "Bentley Test",
            "initials": "BT",
            "username": "bentleytest1"
        }
    },
    {
        "id": "5859989ff4106e02dc3aa479",
        "idMemberCreator": "556c8537a1928ba745504dd8",
        "data": {
            "list": {
                "name": "Test Board",
                "id": "58583a46fdada4bcf993db96"
            },
            "board": {
                "shortLink": "TJBdSPf6",
                "name": "Test Board!!!",
                "id": "58583a421335eb9ad2d3403a"
            },
            "card": {
                "shortLink": "38UCRrH5",
                "idShort": 1,
                "name": "Thing Test Again Again",
                "id": "58583a4a8293d411ec44abf5"
            },
            "text": "test\n"
        },
        "type": "commentCard",
        "date": "2016-12-20T20:46:23.604Z",
        "memberCreator": {
            "id": "556c8537a1928ba745504dd8",
            "avatarHash": "0b48c7057767cca8f339109b27a064d7",
            "fullName": "Matt Cowan",
            "initials": "MC",
            "username": "mattcowan"
        }
    },
]
```

This endpoint retrieves all kittens.

<aside class="warning">This is a warning about something.</aside>


## Card Attachments
## Card Board
## Card CheckItemStates
## Card Checklists
## Card List
## Card Members
## Card MembersVoted
## Card PluginData
## Card Stickers