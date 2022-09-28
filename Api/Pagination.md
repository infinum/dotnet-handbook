Here is an obvious sentence: one of the most important jobs of a backend service is fetching data from a database. When fetching a list of data, most of the time it’s not required to get all the items in the list - this is where pagination comes to play. Pagination is a process of dividing a list of items into pages in order to reduce the amount of data being transferred. The prerequisite for any pagination type is that the data is ordered in some way, after which we must determine how should we split it. There are multiple ways to do that, each with its own benefits and drawbacks, so let’s explore them.

## Offset pagination

This type of pagination requires two numbers to get the correct data: how many items to skip (offset) and how many items to return (size). The offset is usually calculated from the page number and page size because those numbers are much more intuitive to use.

Using the provided numbers, the implementation is quite simple:

```csharp
var resultList = await _dbSet
	.Where(condition)
	.OrderBy(orderByClause)
	.Skip((page - 1) * size)
	.Take(size)
	.ToListAsync();
```

This pagination type is the one that is used the most because of its simplicity and a few key advantages: the ability to jump to a specific page and the ability to see the total number of pages. In most use cases that is more than enough to cover the feature requirements, but there are some cases where this approach is not suitable.

This pagination type must fetch all the data that could potentially be returned, order it, and after that dispose of all the items that are not in the page that must be returned. This can become a problem when the data set becomes large because for a single page a lot more data is processed than it is returned. Another problem arises when a data set that is being queried changes frequently. We are trying to query based on an item’s position in the set, but when the set size changes, the item’s position also changes, so we might get duplicates or skip some items without knowing.

But what if, instead of using the item’s position in the whole set, we just remember what was the last item that we got and then use this data to filter the set and get the next page? This is where cursor and seek pagination come in handy.

## Cursor pagination

The idea behind cursor pagination (aka keyset pagination) is to have a pointer to a specific item in the data set which is later used to determine the item from which the pagination will continue. That way the pagination will continue from the correct item in the list, regardless if the list changed between the two paginated API calls. The client doesn’t need to understand what the cursor is pointing to, it just needs to store it temporarily and add it to the request for the next page.

For this to work, items in the set we’re querying must have a unique value (or values) that will be used both for ordering and for pointing. The pointer should be used as an instruction for filtering, rather than just a simple value (see Seek pagination for that option).

As an example, let’s say that we need to fetch a list of events from a calendar. The unique value that can be used in a pointer could be the start time of an event. The response for such a request could look something like this

```json
{
	"Events": [
		{
			"EventId": 1,
			"StartTime": "2022-14-03T08:00:00Z",
			"EndTime": "2022-14-03T08:15:00Z",
			"Name": "Minions team - Daily meeting"
		},
		...
		{
			"EventId": 42,
			"StartTime": "2022-14-03T20:00:00Z"
			"EndTime": "2022-14-03T21:00:00Z"
			"Name": "Rudolf's new headlamp tests"
		}
	],
	"Cursor": "gte2022-14-03T20:00:00Z"
}
```

The cursor in this example uses the start time of the last event in the fetched list to build the instruction for fetching the next page. Using that, the next client’s request would include the cursor:

```json
{
	"Limit": 15,
	"Cursor": "gte2022-14-03T20:00:00Z"
}
```

This example doesn’t have any encoding on the cursor, but since the cursor must be understood only by the backend server, the cursor is often base64 encoded in order to reduce the bandwidth and to enable changing the cursor without the need to change anything on the client side. As far as they know, the cursor is a magic string that they must pass back to the server to fetch the next page - and nothing more!

This approach addresses the drawbacks of offset pagination, but at a cost:

- There is no concept of the total number of pages or number of items in the set
- We can’t jump to a specific page because the pages are not enumerated
- Items in the set must have a unique, sequential column that can be used for the cursor

## Seek pagination

Seek pagination relies on the same base principle as Cursor pagination: instead of knowing the position in the set, we just know what comes immediately after and before the current page. The difference between them is that seek pagination references a certain value, rather than instructions for filtering. The data that can be used to determine the previous and next items must be unique and sequential.

This approach has more or less the same advantages and drawbacks as Cursor pagination (no pages concept, great for big and frequently changing data sets) - the difference is that it is less flexible because it only supports ordering and filtering by a single predetermined value, but that means also that it is much simpler to implement because the backend doesn’t need to parse the cursor in order to know how to get the required page.

As an example, we’ll build a request that fetches a list of emails from our server. Emails should be ordered by the time they arrive, but since we must give them unique IDs upon arrival, we could assign a sequential integer value to every one of them and use that to filter the pages:

```json
{
	"PreviousId": 42,
	"Items": [
		{
			"Id": 43,
			"From": "santaclause@northpole.com",
			"Subject": "Uber app for gift delivery?"
		},
		...
		{
			"Id": 75,
			"From": "easterbunny@magicalforest.org",
			"Subject": "Annual egg price negotiations with the Chicken alliance"
		}
	],
	"NextId": 76
}
```

The request example returns both a previous and next Id that could be used to fetch the previous or next page:

```json
{
	"Limit": 30,
	"FromId": 76
}
```
