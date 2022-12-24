## Introduction

Imagine your friend runs a database that stores every single photo on the internet. Your friend wants to give you access to the database through an endpoint, but he can’t send you the whole list (since there’s too much data). 

So instead, your friend decides to use pagination. First, he creates a base endpoint that looks like this:

```
mysite/images
```

This base endpoint will send the first 100 images along with a response header that contains a link to the next endpoint:

```
mysite/images/2
```

When someone queries that second endpoint, they will get the next 100 images along with a response header that links to the next endpoint

```
mysite/images/3
```

## The Ideal API

Now that we have our endpoint, our manager asks us to build a simple API for someone else on our team so that they can get access to all of these images without thinking too much about the logic.

There are many ways we could construct this API, but the simplest would be to just give them an array of all the images. That way our teammate doesn’t need to think about any logic, he can just loop through our array and consume as many images as he wants.

Conceptually, here’s what this array would look like:
```javascript
allImages = [img1, img2 ..... img∞]
```

But how would we do this? Obviously its infeasible to fetch every image and store it in an array.

## Using Generators

This is a great use case for asynchronous generators. Our goal is to create generator that our teammate can use in the following manner:
```javascript
// Conceptually this gives us an array of all of the images in the world
let allImages = generator(
  "https://mysite/images"
);
```

This generator (like all generators) will return an iterator. This iterator will loop through every single image in the world, and our teammate should be able to do the following:
```javascript
 for await (const image of allImages) {
  // Do Something
 }
```
So how would we implement a generator like this? Take a look at this code:
```javascript
async function* generator(URL) {
  // Get the response from the first page
  let response = await axios.get(URL);
  let data = response.data;

  while (true) {
    // Loop through 100 images contained on each page
    for (let image of data) {
      yield image;
    }
    // Once we're done with the 100 images, get the next URL
    URL = data.headers.nextPage

    if (!URL) {
      break;
    }
  }
}
```
This generator gives us a really simple way to implement an API that will “appear” as if it’s an infinite array to our teammate. Obviously we don’t have to fetch all of the elements. We just fetch the next element each time. 

## Consuming the Generator

So how would our teammate use this? If our teammate wanted to get the first 1000 images he could write a function that looks like this:
```javascript
// Get an iterator
let allImages = generator(
  "https://mysite/images"
);

function getThousandImages() {
 let idx = 0
 for await (const image of allImages) {
   console.log(image)
   if (idx++ >= 1000) break
 }
}
```
As you can see, our teammate is able to loop through this function as if it’s an infinite array even though it’s not. The usefulness of using a generator in this case is that we’re able to simulate an infinite array of data that we’re fetching asynchronously without having to fetch the data all at once.
