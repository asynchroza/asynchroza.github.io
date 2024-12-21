---
title: Understanding The Flyweight Pattern 
date: 2024-09-03
categories: [Architecture]
tags: [design patterns, architecture, javascript]
pin: false
math: false
mermaid: false
---

I started reading about the release of the [new unique package](https://tip.golang.org/blog/unique) in Go and ended up on Wikipedia, exploring the Flyweight design pattern.

After some browsing, I came across this odd [page](https://www.patterns.dev/vanilla/flyweight-pattern), which is apparently part of a book about design patterns in web applications.

What's odd about this page is that it talks about how JavaScript supports flyweight out of the box with its prototype-based inheritance.

Ok, so far so good... but then they proceed to give an example of how to implement the pattern:

```js
const createBook = (title, author, isbn) => {
  const existingBook = books.has(isbn);

  if (existingBook) {
    return books.get(isbn);
  }

  const book = new Book(title, author, isbn);
  books.set(isbn, book);

  return book;
};

// ...
const bookList = [];

const addBook = (title, author, isbn, availability, sales) => {
  const book = {
    ...createBook(title, author, isbn),
    sales,
    availability,
    isbn,
  };

  bookList.push(book);
  return book;
};
```

\
The `createBook` function creates a new book object if the ISBN hasn't been stored before, and returns the existing object if it has. So far, this works as intended.

The issue arises in the following steps. The Flyweight pattern is a design strategy aimed at saving memory by sharing objects (interning). The goal is for distinct but logically identical objects to share the same instance. However, in this implementation, the book objects are not shared because they are destructured.

Additionally, when we destructure primitive values, such as strings, we end up creating new instances of those values. This behavior contradicts the purpose of the Flyweight pattern, which is meant to avoid duplicating objects (assuming that title and ISBN are strings).

Considering the above points, this implementation doesn't actually share anything. It also poses a risk to data integrity, as it doesn’t ensure data consistency across different instances of a book.
If there are already 10 book instances derived from the parent book, and the parent reference’s title is changed, all subsequent objects will be created with the new title. However, the initial 10 instances will retain the original data, leading to inconsistency.

Here's a quick PoC:

```js
class Book {
  constructor(title, author, isbn) {
    this.title = title;
    this.author = author;
    this.isbn = isbn;
  }
}

const books = [];

const book = new Book("Book Title", "Author Name", "1234567890");

for (let i = 0; i < 1000000; i++) {
  books.push({
    ...book,
    someArbitraryData: "Some arbitrary data",
  });
}
```

\
This code uses ~326330768 bytes or **~326 MBs** of heap memory.

```js
for (let i = 0; i < 1000000; i++) {
  books.push({
    __proto__: book,
    someArbitraryData: "Some arbitrary data",
  });
}
```

\
The correct flyweight implementation uses ~62106400 bytes or **~62 MBs** of heap memory.

The second example uses significantly less memory because it shares the prototype of the book object, rather than duplicating it.
To be exact, that's a difference of 264224368 bytes or ~251 MBs.

By utilizing the reserved `__proto__` property, we can ensure that all instances of the book object share the same prototype (base object).

```js
// Will change the title of all logically equivalent book instances
this.title = "New Title";
```

\
This way, we can achieve the desired memory savings and data consistency. If you don't know what `__proto__` is, I would highly suggest you read through Mozilla's docs about the [prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain).
