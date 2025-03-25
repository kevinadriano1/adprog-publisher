# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model struct is enough?

In the Observer pattern, a Subscriber is usually defined as a trait that specifies how an object should receive updates from a Subject. This is useful when you have multiple types of subscribers that may behave differently when they get updates.

However, in BambangShop, the Subscriber is just a simple struct that holds data, and all subscribers behave the same way. There’s no custom behavior or need to treat different subscribers differently. So in this case, using a trait is unnecessary and would only add complexity without real benefit.

You would only need a trait if you plan to introduce different kinds of subscribers in the future (e.g., SMS notifications, email alerts, or admin logging). In that case, a trait would help you manage and reuse code across different subscriber types while following good design principles like Open-Closed and Dependency Inversion from SOLID.

id in Program and url in Subscriber is intended to be unique. Explain based on your understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently use is necessary for this case?

While it's technically possible to use a Vec (list) to store items with unique fields like id or url, it’s not efficient, especially as the data grows. With a Vec, you’d have to manually search through the entire list to check for duplicates or find a specific item. This means operations like checking for existence, insertion, and deletion can become slow and error-prone, because they rely on linear search.

On the other hand, DashMap is a much better fit for this use case. It works like a dictionary (HashMap) and stores data as key-value pairs, where the key (like url) is unique by default. 
In short, DashMap is faster, safer, and more scalable—making it the right tool for managing your list of unique subscribers in a multi-threaded context like Rust.

When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we used the DashMap external library for thread safe HashMap. Explain based on your understanding of design patterns, do we still need DashMap or we can implement Singleton pattern instead?

The Singleton pattern ensures that only one instance of a structure exists and provides global access to it. However, by itself, the Singleton pattern does not guarantee thread safety. This means if multiple threads try to access or modify shared data (like the SUBSCRIBERS list) at the same time, it could lead to data races or unexpected behavior.

In Rust, thread safety is strictly enforced by the compiler, so it's important to use data structures that are explicitly designed for concurrent access. That’s where DashMap comes in — it’s a thread-safe map that lets multiple threads read and write safely at the same time, without the need for manual locking (like Mutex or RwLock).

Even if you wrap your subscriber list in a Singleton, you’d still need to use something like DashMap inside it to ensure safe, concurrent access. So in this case, DashMap is still necessary, either on its own or as part of a thread-safe Singleton implementation.

#### Reflection Publisher-2

In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from a Model?

In traditional MVC, the Model handles both data access and business logic, but as applications grow in complexity, this all-in-one approach becomes difficult to maintain. By separating the Model into a Service layer (for business logic) and a Repository layer (for data access), we follow the Single Responsibility Principle, making each component focused and easier to manage. This separation improves testability, since services can be tested without needing a database, and enhances reusability and flexibility, allowing services to be reused across different parts of the system and repositories to be swapped out with minimal changes. Overall, it leads to cleaner, more modular, and maintainable code.

What happens if we only use the Model? Explain your imagination on how the interactions between each model (Program, Subscriber, Notification) affect the code complexity for each model?

If we rely solely on the Model to handle everything—data representation, business logic, and database access—each model becomes overloaded with responsibilities. This not only makes the code harder to read and maintain but also leads to tight coupling between models, where one model might rely heavily on the internal workings of another. As a result, even minor changes in one part of the code can cause unexpected issues elsewhere. This tangled structure makes the system fragile and difficult to scale. By separating concerns into dedicated Service and Repository layers, we keep our codebase cleaner, more organized, and much easier to test, maintain, and extend in the future.

Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects.

yes, postman is a very helpful tool for testing and checking if your APIs work correctly, especially when building the back-end. It lets you easily send requests like GET, POST, PUT, and DELETE without needing a front-end, and you can see the response clearly with status codes and data. You can save your requests in collections so you don’t have to rewrite them, and use variables to switch between different environments like development or production. Postman also lets you test APIs before the back-end is fully ready by using mock servers, and you can write small scripts to test things automatically. In group projects, it helps everyone test faster, stay organized, and work better together.

#### Reflection Publisher-3

Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and Pull model (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?

his tutorial uses the Push model because every time something changes in the Product Service—like when a product is added or removed—a notification is immediately sent (or "pushed") to all subscribers who follow that specific product type. This happens through the NotificationService's notify method that we implemented. The subscribers don’t ask for updates; instead, the system automatically pushes the update to them as soon as the change happens. This makes the process fast and automatic, without the subscribers needing to do anything to receive updates.

What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)

In the Pull model, the publisher only tells subscribers that something has changed, but it’s up to the subscribers to fetch the actual data themselves. This approach can be more flexible, as subscribers can decide what information they want, and it reduces unnecessary data transfer if not all details are needed. 

However, it also comes with downsides: it adds an extra step, making the process slower, and requires subscribers to know how to fetch the data from the publisher. This increases complexity and creates tighter coupling between the two. In the context of this tutorial, where all subscribers are meant to receive the full notification anyway, using the Pull model would make the system more complicated without offering real advantages.

Explain what will happen to the program if we decide to not use multi-threading in the notification process.

If we don’t use multi-threading, the program will send notifications to all subscribers one by one, in a blocking way. This means: If one subscriber is slow or stuck, the whole process is delayed. The system becomes less responsive, especially if there are many subscribers. The main thread may be blocked, which could freeze or slow down other parts of the app.
Using multi-threading allows notifications to be sent in parallel, improving performance and keeping the app fast and responsive.