# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia
---
### Rakabima Ghaniendra Rusdianto - Advanced Programming A - 2306228472
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
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
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
##### 1. Observer Pattern: Subscriber as Interface vs. Model Struct
In the Observer pattern (as presented in Head First Design Patterns), the Subscriber is defined as an interface so that different concrete implementations can subscribe and react differently to notifications. In this BambangShop case, a single `Subscriber` model struct is defined without a dedicated trait.
- **Why a single model can be enough:**  
  This application currently requires only one kind of subscriber behavior – storing the subscriber’s URL and name – and there is no need for multiple polymorphic subscriber types.
- **When to consider an interface (trait in Rust):**  
  If in the future the app need to support various kinds of subscribers (e.g., email subscribers, SMS subscribers, etc.), then defining a `Subscriber` trait would be beneficial. For now, a single model struct meets the requirements.

##### 2. Data Structure for Unique Identifiers: Vec vs. DashMap
In this application, the `id` in `Product` and the `url` in `Subscriber` are intended to be unique identifiers.
- **Using a Vec (list):**  
  While a Vec can store a list of items, checking for uniqueness or performing lookups requires iterating over the entire list (O(n) complexity). This becomes inefficient as the data grows.
- **Using DashMap (map/dictionary):**  
  DashMap provides constant-time (O(1)) lookup, insertion, and deletion. This makes it much more suitable for enforcing uniqueness and handling concurrency. The choice of using DashMap ensures fast access and better scalability, especially when uniqueness is crucial.

##### 3. Thread Safety: DashMap vs. Singleton Pattern
Rust’s compiler enforces strict thread-safety guarantees, but when dealing with mutable global state (such as this app's static list of subscribers), it needs a thread-safe container.
- **DashMap:**  
  DashMap is a concurrent hash map that provides built-in synchronization, allowing multiple threads to safely read and write without manual locking.
- **Singleton Pattern:**  
  The Singleton pattern can ensure that only one instance of a structure exists (e.g., our global database of subscribers). However, it does not inherently guarantee thread safety for concurrent modifications.
- **Conclusion:**  
  The app still need DashMap (or another thread-safe collection) even if the Singleton pattern is implemented, because thread safety is about protecting concurrent access. The use of DashMap in the `SubscriberRepository` ensures that the global state is safely accessible across threads.

#### Reflection Publisher-2
##### 1. Why Separate “Service” and “Repository” from the Model?
In the traditional MVC pattern, the Model often encompasses both data storage and business logic. However, separating the Service and Repository layers provides several benefits:
- **Separation of Concerns:**
    - The **Repository** handles all data access and persistence logic (e.g., our `ProductRepository` and `SubscriberRepository` use DashMap for thread-safe storage).
    - The **Service** layer encapsulates business logic and workflows (e.g., our `NotificationService` and `PaymentServiceImpl` coordinate data manipulation and enforce business rules).
- **Improved Maintainability:**
    - Changes to data access (such as moving from an in‑memory store to a database) affect only the Repository without impacting business logic.
    - Business rules and interactions (like cascading status updates from Payment to Order) are maintained in the Service layer, making the system easier to evolve and test.
- **Enhanced Testability:**
    - With distinct layers, you can test business logic in the Service layer independently from persistence logic. This results in cleaner, more focused unit tests.

##### 2. Consequences of Using Only the Model
If all the responsibilities are combined into a single Model:
- **Increased Complexity:**
    - A model that handles data storage, business logic, and even presentation concerns becomes bloated and harder to maintain. For example, a `Product` model that includes persistence methods (like `save()` or `findAll()`) and business rules (like calculating discounts) can quickly become entangled.
- **Tight Coupling:**
    - Without separation, changes in how data is stored or how business logic is processed would force modifications in the model itself. For instance, if both `Subscriber` and `Notification` models contain logic to manage subscription status, changes in subscription rules could affect both models, leading to code duplication and bugs.
- **Difficult Testing and Reuse:**
    - When a single model handles multiple responsibilities, writing isolated tests becomes challenging. The interactions between, for example, the `Program`, `Subscriber`, and `Notification` models can result in complex interdependencies, increasing the chance of regression errors and making future modifications risky.

##### 3. Exploring Postman for Testing
I have explored Postman extensively, and it has proven to be an invaluable tool for testing my application. In this BambangShop project:
- **HTTP Request Simulation:**
    - Postman allows me to simulate various HTTP requests (GET, POST, DELETE, etc.) and easily verify that endpoints behave as expected. For example, I can quickly test subscription and unsubscription routes defined in our `notification` controller.
- **Response Validation:**
    - With Postman, I can inspect response status codes, headers, and JSON bodies, which is critical for validating API behavior.
- **Collection Management and Automation:**
    - I can group API requests into collections (as seen in the provided Postman collection) and even automate tests with scripts. This organization makes it easier to share API test cases with my group and integrate them into CI/CD pipelines.
- **Feature Interest:**
    - I find features like automated testing, environment variables, and monitors particularly helpful. These allow for consistent testing across different environments (development, staging, production) and support future projects that require robust API testing.

Overall, Postman not only speeds up the API testing process but also ensures that our endpoints work as intended before deploying changes.
#### Reflection Publisher-3
