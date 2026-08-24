# What is an API: Starting from Zero

## 1. The hook: the problem an API solves

Imagine you have a weather app on your phone. When you open it, it instantly tells you "It's 18°C in Amsterdam right now, with light rain." Stop and think for a second, your phone itself isn't connected to any weather station. It has to "ask" for this data from somewhere else.

So how does the phone do that? It "asks" another system (one that holds weather data), using a fixed, defined way of asking. This way of asking is exactly what an API is.

## 2. A simple definition of an API

An **API (Application Programming Interface)** is a fixed set of rules that describes how one piece of software can "talk" to another, without needing to know what's happening internally inside that other software.

The weather app doesn't know (and doesn't need to know) how weather data is collected, which satellites are used, or what calculations are involved. All it knows is: "if I ask in this specific way (the API), I'll get an answer back in this specific format."

## 3. A restaurant analogy, we'll use this throughout the guide

Imagine you've gone to a restaurant:

- **You (the customer)** ask for what you want
- **The waiter** is the middleman (this is the API)
- **The kitchen** is where the food actually gets made

You don't need to go into the kitchen and cook the food yourself. You just look at the menu (the available options), tell the waiter "I want this," and the waiter brings it to you from the kitchen. You don't need to know the kitchen's internal process, how the chef works, or which order gets made first.

An API plays exactly this role: a middleman layer between two pieces of software.

## 4. Client and Server: two essential names

There are two "characters" in this whole concept, and their names are worth remembering:

- **Client**: the one that sends the request (asks for something), like our weather app, or the customer at the restaurant
- **Server**: the one that gives the response (answers), like the system holding the weather data, or the kitchen

## 5. The request-response cycle

Every API interaction happens in two steps:

1. **Request**: the client "asks" for something. For example: "give me the weather for Amsterdam"
2. **Response**: the server answers. For example: "18°C, light rain"

This cycle usually completes in a very short time (milliseconds).

## 6. Let's try it ourselves: a real API call

Let's not keep this purely theoretical, let's see it in action. **Open-Meteo** is a free weather API that anyone can use, no login, no password, no "key" required.

Run this command in your terminal:

```bash
curl "https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&current_weather=true"
```

Here's what we sent (this was our **request**):

- **Endpoint**: `https://api.open-meteo.com/v1/forecast` (this is the "address" where we're sending the request, just like a restaurant's address)
- **Parameters**: `latitude=52.52&longitude=13.41` (these are extra details that specify which location's weather we want, here we gave the coordinates for Berlin)
- **`current_weather=true`**: this tells the API "I want the current weather" (not a 7-day forecast)

And the response that comes back looks something like this (this is the server's **response**):

```json
{
  "latitude": 52.52,
  "longitude": 13.42,
  "current_weather": {
    "temperature": 15.3,
    "windspeed": 12.4,
    "winddirection": 210,
    "weathercode": 3,
    "is_day": 1,
    "time": "2026-03-26T10:00"
  }
}
```

See that, without any login, you got the data directly, in a specific, defined format (this is called **JSON**, a common way of writing data that a lot of APIs use).

## 7. What else is in a response

Every response also comes with a **status**, which tells you whether the request succeeded or not. For example, if you send an incorrect latitude/longitude, you'll get an error instead of a successful result. (These are called "status codes", for example `200` means "everything's fine," `404` means "couldn't find that." These are very common, if you've ever done any web development, you've likely seen them.)

## 8. REST: a brief mention

A lot of modern APIs (including the one we just used) follow a pattern called **REST**. It's enough to understand this much: REST has a fixed set of "actions" you can perform, **show something (GET)**, **create something new (POST)**, **change something (PUT)**, **delete something (DELETE)**. The `curl` command we just ran was a **GET** request, meaning "show me the data," we didn't create or change anything.

## 9. Bridge: this concept will come in handy next

Now, these same concepts (client, server, request, response, and REST actions like GET/POST) apply to Kubernetes as well.

When you run `kubectl apply -f file.yaml`, you're actually sending a **request** to Kubernetes' **API server**, exactly like what we just did with `curl` against the weather API. The only difference is: with the weather API, we did a **GET** ("show me the data"), but here we're doing a **POST**-like action ("create this new resource"). Kubernetes' API server (this is the **server**) processes it and sends back a response, "created successfully" or "here's an error."

So the question is: **what** exactly is being "created"? Just like with the weather API, where we could ask for specific things (like "current weather," or "a 7-day forecast"), the Kubernetes API also has specific "things" you can manage, called **resources** (like `Pod`, `Service`).

One of these resources is called `Ingress`, whose job is to describe how traffic coming from outside should be routed to a service inside the cluster. Over time, the limitations of `Ingress` became apparent, and Kubernetes designed a new, better resource called `Gateway API`.

In the next guide, we'll look at exactly what `Ingress`'s limitations were, and how `Gateway API` solved them.
