# Sukafu Analytics

##### **(This is a small project to learn how to build an API in python using Flask and MSQAlchemy, it is still a work in progress and should be treated as such)**

This is a analytics system made for game developers who want to worry about the game design, not the data analysis!

We are focused on creating a web application that can be easily implemented into your games, and we also provide a Unity Package that can be used to register events in the system.

The Web Application will be a `REST API` and the Unity Package can be used in any Unity version after `Unity 2017.3.x`

# System Structure

When using the Sukafu Analytics, you will only have to worry about what you want to register and analyze later, we will provide you with the following structure:

#### 1- Apps
The game or application that you are working on, this will hold all of the events that this game can register

#### 2- Events
Each App can have multiple events, which will be registered in the system for later analysis, an event can be triggered by specific actions in the game (e.g.: player died)

#### 3- Attributes
Each event can have multiple attributes, which are types of data that will be saved for future analysis and will be sent to the system by games when the event is triggered, attributes can have the following types:

- **Int**
- **Float**
- **Bool**
- **Timedate**
- **Vector3**: Composed of three floats (x, y, z)
- **Transform**: Composed fo three Vector3 (position, rotation, scale)

```
Tip: Events can have multiple attributes, which can makes then quite complex.
For example you could create an event of "Finish Stage" which would have the following attributes:
- Time (Float)
- PlayerTransform (Transform)
- Fuel (Int)
- FinishDate (Timedate)
```

# API Usage

## Creating a User
To use the API you will need to create an account in the analytics system using the `/users/` endpoint, making a POST request with the following params:
```javascript
{
	"username": "string",
	"password": "string"
}
```
---
After that you will need to login using the `/users/login` endpoint making a POST request with the following params:
```javascript
{
	"username": "string",
	"password": "string"
}
```
---
This will return you the access token which will be used to make all the following requests, including the setup and registering of game events.

## Setting up the Game

To setup a game you will need to create its `App`, at least one `Event` and at least one `Attribute`, this can be made using the following endpoints:

### - `/apps`
Making a GET request is this endpoint will return all of your apps, and making a POST request with the following params will create a new one:
```javascript
{
	"name": "string", // The app's name
	"genre": "string", // The app's genre (optional)
	"platform": "string" // The app's plataform (optional)
}
```
This request will return the newly created app and its `UUID`, which will be used for later requests
```javascript
{
	"name": "string", // The name of the created App
	"uuid": "string" // The UUID of the created App
}
```

### - `/apps/[app:uuid]/`
Making a GET request on this endpoint will return the given App's info and its events
```javascript
{
	"name": "string",
	"genre": "string",
	"platform": "string",
	"uuid": "string",
	"events": [
		{
			"name": "string",
			"description": "string",
			"uuid": "string",
			"attributes": [
				{
					"name": "string",
					"type": "string",
					"uuid": "string"
				}
			]
		}
	]
}
```

### - `/apps/[app:uuid]/events`
Making a GET request is this endpoint will return all the events for the given App, and making a POST request with the following params will create a new one:
```javascript
{
	"name": "string", // The name of this event
	"description": "string" // More info about this event (optional)
}
```
This request will return the newly created event and its `UUID`, which will be used for later requests
```javascript
{
	"name": "string",
	"uuid": "string"
}
```

### - `apps/[app:uuid]/events/[event:uuid]`
Making a GET request on this endpoint will return the given Event's info and its attributes:
```javascript
{
	"name": "string",
	"description": "string",
	"uuid": "string",
	"attributes": [
		{
			"name": "string",
			"type": "string",
			"uuid": "string"
		}
	]
}
```

### - `apps/[app:uuid]/events/[event:uuid]/attributes`
Making a GET request is this endpoint will return all the attributes for the given Event, and making a POST request with the following params will create a new one:
```javascript
{
	"name": "string", // The name of this attribute
	"type": "string" // The type of this attribute (Int, Float, Vector3, etc)
}
```
This request will return the newly created attribute and its `UUID`, which will be used for later requests
```javascript
{
	"name": "string",
	"uuid": "string"
}
```
### -`apps/[app:uuid]/events/[event:uuid]/attributes/[attribute:uuid]`
Making a GET request on this endpoint will return the given Attribute's info
```javascript
{
	"name": "string",
	"type": "string",
	"uuid": "string"
}
```

## Querying the data
You can query both `Events` and `Attributes` of an App, both will accept the following fields as query params:

- **start_date**: The minimum datetime for when the occurrence happened, the default is the minimum value possible
- **end_date**: The maximum datetime for when the occurrence happened, the default is the current timedate
- **order_by**: The field to order by, the name of the attribute in case of Events (e.g.: "time") or the field in case of a complex attribute (e.g.: "x") and which order to make: "ASC" or "DESC". (e.g.: `"time ASC"`), the default will be sorting by date of creation, for both Events and Attributes
- **page**: The page to get the occurrences, since occurrences list can get really big this endpoint will be paginated, the default is `1`
- **per_page**: How many occurrences you will have per page, the default is `20` and this number is hard capped in the API to `100`

### Querying Events

To query all the occurrences of an Event you can call the endpoint:
`/app/[app:uuid]/events/[event:uuid]/occurrences`
using the params listed above, this will return a list of occurrences for the given Event and a pagination object, the response will look like this:
```javascript
{
	"name": "string", // The name of this event
	"description": "string", // Description of this event
	"ocurrences": [ // An array of the ocurrences of this event in this page
		{
			"attribute1": {
				"name": "string",
				"type": "string",
				"value": 10
			},
			"attribute2": {
				"name": "string",
				"type": "string",
				"value":  23.3
			},
			"attribute3": {
				"name": "string",
				"type": "string",
				"value": {
					"x": 10,
					"y": 2,
					"z": 0
				}
			}
		}
	],
	"pagination": {
		"page": 1,
		"per_page": 20,
		"total_pages": 3,
		"has_next": true,
		"has_previous": false
	}
```

### Querying Attributes
To query all the occurrences of an Attribute you can call the endpoint:
`/app/[app:uuid]/events/[event:uuid]/attributes/[attribute:uuid]/occurrences`
using the params listed above, this will return a list of occurrences for the given Attribute and a pagination object, the response will look like this:
```javascript
{
	"name": "string", // The name of this attribute
	"type": "string", // The type of this attribute
	"ocurrences": [ // An array of the ocurrences of this attribute in this page
		{"value":10},
		{"value":12},
		{"value":14},
		{...}
	],
	"pagination": {
		"page": 1,
		"per_page": 30,
		"total_pages": 1,
		"has_next": false,
		"has_previous": false
	}
```
