# Lux9 in Social Media

A friend of mine wanted to make a single post deploy to many social media sites. I thought that was a great
idea and more important. I don't want to go through the turmoil of dealing with every social API. Luckily someone else
did that work.


## Buffer.com and lux9.dev

* [https://lux9.dev](https://lux9.dev)
* https://buffer.com

Lux9 is my long journey into coding on strange operating systems and the result was a light weight language with a lot of 
power and a ton of features. 

## Buffer use GraphQL

I'm not a fan of GraphQL but I had to make sure my language supported because its not likely to go away any time soon. 

To get up and running you only need to do a few things.

1. Create an account on buffer
2. Create an API key: Seetings -> API -> Generate API key
3. Conect a new channel: linkedin, mastadon, and instagram
4. API documentation: https://developers.buffer.com/examples/create-draft-post.html

Currently supports the three: linkedin, mastadon, and instagram. Its all I need for free tier

You need some code: 

```
import "creds.lux";

var channel = ["6abxxxxxxxxx"];


class Headers { init() {}}
class GraphQLRequest { init(query) {this.query = query;} }

fun BufferAPI(query) {
	var access_key = appSecret();
	var headers = Headers();
	headers.authorization = "Bearer " + access_key;
	headers.Content_Type = "application/json";
	var url = "https://api.buffer.com";
	var body = toJSON(GraphQLRequest(query));
	var res = httpRequest("POST", url, body, headers);
	return res;
}
```

Running this is simple: `lux buffer.lux`

To execute the function above you need to write a couple more lines:

```
var buffer = BufferAPI(org_query);
print buffer;
```

The orq_query is a GraphQL line that you can copy and paste: 

Get Organization ID: 

```
var org_query = "
query GetOrganizations {
  account {
    organizations {
      id
      name
    }
  }
}
";
```

Response: `{"data":{"account":{"organizations":[{"id":"6abxxxxxxxxxxxx","name":"My organization"}]}}}`

You also need to get a list of channel id's. After you connect linkedin and instagram you can run this. 

```
var chan_query = "
query GetChannels {
  channels(input: {
    organizationId: \"6abmyOrgxxxxxxxxx\"
  }) {
    id
    name
    displayName
    service
    avatar
    isQueuePaused
  }
}
";
```

To write a single post is a more complicated GraphQL. I created a function to make it easier to draft messages to post. 

```
fun draftPost(channel, text, image_url) {
	var cleanId = strTrim(channel);
	var media_metadata = ""; // instagram
	if (cleanId == "6ab87246ea19ca0bdefd7b57") {
		media_metadata = "    metadata: { instagram: { type: post, shouldShareToFeed: true } }\n";
	}

	var query = "
mutation CreateDraftPost {
  createPost(input: {
    text: \"\"\"" + text +  "\"\"\"
    channelId: \"" + channel + "\"
    schedulingType: automatic
    mode: addToQueue
    saveToDraft: true
" + media_metadata + "
    assets: [
      {
        image: {
          url: \"" + image_url + "\"
        }
      }
    ]
  }) {
    ... on PostActionSuccess {
      post {
        id
        text
      }
    }
    ... on MutationError {
      message
    }
  }
}
	";
	return query;
}
```
I pass three things, the "channel" ID that I would keep in an array, the "text" message I want to send and an image URL. You kind of need that every time if you are dealing with instagram and other social media tools. It must be a public available image. Instagram is also weird. I had to create a filter for it

```
var cleanId = strTrim(channel);
	var media_metadata = ""; // instagram
	if (cleanId == "6ab87246ea19ca0bdefd7b57") {
		print "hit channel";
		media_metadata = "    metadata: { instagram: { type: post, shouldShareToFeed: true } }\n";
	}
```

I'm sure more of these would be needed as you add more channels.

That's it. I draft my messages "saveToDraft: true" before sending it out.

### Future code adjustments

* To Publish Live Instantly: Change the parameter flag line from saveToDraft: true over to saveToDraft: false inside your query constructor.
* To Schedule for a Specific Time: Switch the schedulingType: automatic layout field to schedulingType: custom, and append a standard ISO timestamp argument row (e.g., scheduledAt: "2026-10-01T15:00:00Z").
* To Support Video or Reels: Simply switch the inner asset node array from nesting under image to standard multi-part payload syntax: assets: [{ video: { url: "..." } }].

## local webserver 

This took a couple of hours to write. I could make a simple local webserver with a form field to post one message to all the channels you have configured. Or you could do that. Good luck!