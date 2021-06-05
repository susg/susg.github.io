+++
categories = ["Hugo", "Jekyll"]
date = "2024-03-10"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
slug = "Migrer vers Hugo depuis Jekyll"
title = "Migrer vers Hugo depuis Jekyll"
tags = ["posts","hugo"]
draft = true
[ author ]
  name = "Hugo Authors"
+++

Lets design a platform which allows users to reserve a seat in a restaurant online. The platofrm allows users to browse through a list a restaurants and reserve seats, anytime anywhere.

__Similar Service:__ Zomato

**Dificulty:** Medium

### Requirements and Goals of the System

__Functional__:

1. Display the restaurants of the user's city.
2. On restaurant page - let the user select the slot and number of seats to reserve.
3. If number of seats is available for the given slot - reserve and notify user
4. Else show an error message and return to step 2.


__Non Functional__:

1. Handle multiple requests gracefully.

### Design Considerations

1. Seat gets free when the time slot ends.
2. System will not handle Partial Reservation.


### Capacity Estimation

__Traffic Estimates__: Let's assume we have 1 billion views per month and do 15 million reservations a month.

__Storage Estimates__: Let's assume we run in 500 cities and on average each city has 20 restaurants. If there are 10 slots in each restaurant

### API's
- get_restaurants(auth_token, page_number, page_size)

Response: 
```JSON
{
    "restaurants": [{
        "id": 1,
        "address": "abcd efg"
        "rating": 4
    }, {
        "id": 2,
        "address": "abcd efg"
        "rating": 4
    }]
}
```
- get_slots(auth_token, restaurant_id, page_number, page_size)

Response:
```
{
    "slots": [{
        "id": 1,
        "start_time": "10:30 am"
    }, {
        "id": 2,
        "start_time": "11:30 am"
    }]
}
```
- reserve_seats(auth_token, slot_id, seats)

Response:
```
SUCCESS:
{
    "code": 200
    "message": "Successfully reserved"
}

FAILURE:
{
    "code": 400
    "message": "Seat not available"
}
```

### Database Design
![dbschema](restaurant_db.png)

### High Level Design


### Detailed Component Design

Lets start with the flow itself:

- get list of all restaurants in a city: 
Query over the `Restaurant` table to get all the restaurants in a city. We can cache this response in redis as this data is not going to change very frequently. 

- get list of all slots of a restaurant:
This can be fetched by querying over the `Slot` table by filtering on `restaurant_id` and `start_time`. We can further filter slots which have `seats_available` > 0.

- reserve seats:
We need to perform the following sequentially:
    
    1. Check in `Slot` table if the corresponding slot has the requested seats available.
    2. If yes, then subtract that number from seats available and create a row in `Reservation` table.
    3. Notify the user about successful reservation.
    4. If no in step 1, show the user that only `x` seats are available. 

Though this looks simple but we are missing a very important point here, thats, Concurrency. How are we going to multiple requests coming such that no two users book the same seat. We need to take a lock the rows before we update it. We can utilize the [Transaction Isolation Level](https://docs.microsoft.com/en-us/sql/odbc/reference/develop-app/transaction-isolation-levels?view=sql-server-ver15) provided by the SQL server. This isolation level gurantees safety from Dirty, Nonrepeatable and Phantom read. 

```SQL
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

BEGIN TRANSACTION;

SELECT seats_available FROM slot WHERE id = 123;

/* if seats_available >= required seats */
UPDATE slot SET seats_available = .....
INSERT INTO reservation .......

COMMIT TRANSACTION;
```

### DATABASE PARTITIONING

To scale our DB, we need to [partition](https://en.wikipedia.org/wiki/Partition_(database)) it so that it can store information about millions of slots / bookings. We need to come up with a partiotioning scheme that would divide and store data to different DB servers. 

We can do range based partitioning based on `slot_id`. So what will happen is, slot_ids 1 - 1000 will be stored in server 1, 1000- 2000 will be stored in server 2 and so on.

We can also partition it based on `restaurant_id` such that slots of one restaurant are stored in one server. But it won't be a good idea because a very running restaurant could cause load on one server.