# Session 1 — REST Assured with BDD: Fundamentals

## Session Objective
This session provides a foundational understanding of REST Assured, how to use it to test REST APIs, and introduces the Behavior-Driven Development (BDD) style for writing clear, readable API automation tests in Java.

## What is REST Assured and Why it Matters in QA Automation

**REST Assured** is a powerful Java library designed to simplify the testing and validation of RESTful web services. It provides a domain-specific language (DSL) that makes writing API tests highly readable and maintainable, especially when compared to using lower-level HTTP client libraries.

**Why it matters in QA Automation:**
*   **Ease of Use:** REST Assured abstracts away much of the boilerplate code required for HTTP communication, allowing testers to focus on the API logic rather than the underlying HTTP complexities.
*   **BDD Support:** Its syntax naturally aligns with Behavior-Driven Development (BDD) principles, making tests self-documenting and understandable by technical and non-technical stakeholders alike.
*   **Robust Assertions:** It offers comprehensive capabilities for asserting various aspects of API responses, including status codes, headers, and complex JSON/XML body structures using JSONPath and XPath.
*   **Integration:** Seamlessly integrates with popular testing frameworks like JUnit 5 and TestNG, as well as build tools like Maven and Gradle.
*   **Debugging:** Excellent logging capabilities for requests and responses aid significantly in debugging API issues and understanding test failures.

## Explanation of Key Concepts

### Given / When / Then (BDD Style)

The BDD (Behavior-Driven Development) style is a way of writing tests that focuses on the behavior of the system from the perspective of its users. REST Assured beautifully implements this pattern using the keywords `given()`, `when()`, and `then()`.

*   **`given()` (Preconditions/Setup):**
    *   This part defines the *state* of the system before the action takes place.
    *   It's where you configure your request: set headers, path parameters, query parameters, authentication, and the request body.
    *   Think of it as setting up the "context" for your API call.
    *   In the code, this often involves methods like `contentType()`, `header()`, `body()`, `param()`, etc.

*   **`when()` (Action/Event):**
    *   This part describes the *action* or *event* that triggers the system's behavior.
    *   It's where you specify the HTTP method (GET, POST, PUT, DELETE) and the endpoint path.
    *   This is the actual API call being made.
    *   In the code, this is typically represented by methods like `get()`, `post()`, `put()`, `delete()`, etc.

*   **`then()` (Assertions/Outcomes):**
    *   This part specifies the *expected outcome* or *result* of the action.
    *   It's where you validate the API's response: status code, response headers, and the response body content.
    *   Think of it as asserting the "result" of your API call.
    *   In the code, this involves methods like `statusCode()`, `body()`, `header()`, `contentType()`, etc.

**Example (from `PetApiTest.java`):**
```java
given() // Preconditions: Log all request details
    .log().all()
.when()  // Action: Make a GET request to /pet/{id}
    .get("/pet/{id}", petId)
.then()  // Assertions: Log response, check status code, and JSON body fields
    .log().all()
    .statusCode(200)
    .body("id", equalTo((int) petId));
```

### RequestSpecification

While not always explicitly instantiated as an object in basic tests, `RequestSpecification` is the underlying interface that `given()` returns. It allows you to define and configure all aspects of an HTTP request before it is sent. When you chain methods like `contentType()`, `header()`, `body()`, `param()` after `given()`, you are building up the `RequestSpecification`. This object is then used by the `when()` part to execute the actual HTTP call.

### ResponseSpecification

Similar to `RequestSpecification`, `ResponseSpecification` is the interface returned by `then()`. It allows you to define expectations and perform assertions on the HTTP response received after the request is executed. Chaining methods like `statusCode()`, `body()`, `header()` after `then()` means you are building up the `ResponseSpecification` to validate the API's behavior.

### JSON Path Assertions

JSONPath is a query language for JSON, similar to XPath for XML. REST Assured integrates JSONPath to allow easy navigation and extraction of data from JSON responses. This is incredibly useful for making precise assertions on specific values within a complex JSON structure.

**Syntax Examples (from `PetApiTest.java`):**
*   `body("id", equalTo((int) petId))`: Asserts the value of the `id` field at the root of the JSON response.
*   `body("name", equalTo("doggie"))`: Asserts the value of the `name` field at the root.
*   `body("category.name", equalTo("Dogs"))`: If your JSON had a nested `category` object with a `name` field, this would assert its value.

## Walk-through of `BaseTest.java`

Open `restassured-session1-playbook/src/test/java/com/example/restassured/BaseTest.java`

```java
package com.example.restassured;

import io.restassured.RestAssured;
import org.junit.jupiter.api.BeforeAll;

public class BaseTest {

    // Base URI for the PetStore API
    protected static final String BASE_URI = "https://petstore.swagger.io/v2";

    @BeforeAll
    static void setup() {
        // Set the base URI for all REST Assured requests
        RestAssured.baseURI = BASE_URI;
        
        // This setup will apply to all tests inheriting from BaseTest.
        // For demonstration, we'll enable logging directly in the tests
        // using .log().all() to show specific request/response details per test.
        // For global logging across all tests, one could use filters like:
        // RestAssured.filters(new RequestLoggingFilter(), new ResponseLoggingFilter());
        // but for a training session, explicit logging per test is clearer.
    }
}
```

*   **`package com.example.restassured;`**: Defines the package for our test classes.
*   **`import io.restassured.RestAssured;`**: Imports the main class for REST Assured.
*   **`import org.junit.jupiter.api.BeforeAll;`**: Imports the JUnit 5 annotation for a method that runs once before all tests in the class.
*   **`public class BaseTest { ... }`**: This is our base class. Any test class that extends `BaseTest` will inherit its configurations. This promotes reusability.
*   **`protected static final String BASE_URI = "https://petstore.swagger.io/v2";`**: Defines a constant for the base URL of the API we are testing. Making it `static final` means it's available to all instances and cannot be changed.
*   **`@BeforeAll static void setup() { ... }`**: This method is annotated with `@BeforeAll`, meaning it will be executed *once* before *all* test methods in `BaseTest` or any class extending it.
*   **`RestAssured.baseURI = BASE_URI;`**: This is a crucial line. It sets the default base URI for all subsequent REST Assured requests. This means in our `PetApiTest.java`, we don't have to specify `https://petstore.swagger.io/v2` in every `get()`, `post()`, etc., call; we can just use the path like `/pet/{id}`.
*   **Logging Comment**: The comment explains that while global logging can be set up here using filters (e.g., `RequestLoggingFilter`, `ResponseLoggingFilter`), for this introductory session, we will enable logging directly within each test method using `.log().all()` for clearer demonstration and visibility of each API interaction.

## Walk-through of Each Test in `PetApiTest.java`

Open
`restassured-session1-playbook/src/test/java/com/example/restassured/PetApiTest.java`

### `testGetPetById()`

```java
@Test
@DisplayName("Test GET /pet/{id} - create then retrieve pet by ID")
void testGetPetById() {

    int petId = ThreadLocalRandom.current().nextInt(1_000_000, Integer.MAX_VALUE / 1000);

    String createBody = String.format(
        "{\"id\":%d,\"name\":\"doggie\",\"status\":\"available\"}", petId);

    // Create the pet
    given()
        .log().all()
        .contentType(ContentType.JSON)
        .body(createBody)
    .when()
        .post("/pet")
    .then()
        .log().all()
        .statusCode(200)
        .body("id", equalTo(petId))
        .body("name", equalTo("doggie"))
        .body("status", equalTo("available"));

    // Retrieve the pet
    given()
        .log().all()
    .when()
        .get("/pet/{id}", petId)
    .then()
        .log().all()
        .statusCode(200)
        .body("id", equalTo(petId))
        .body("name", equalTo("doggie"))
        .body("status", equalTo("available"));
}
```

* **`@Test`**: Marks this method as a JUnit test.
* **`@DisplayName(...)`**: Provides a readable test name in reports.
* **`int petId = ThreadLocalRandom.current().nextInt(...)`**: Generates a random integer ID for the test pet.
* **`String createBody = String.format(...)`**: Builds a JSON request body with the generated ID, name, and status.
* **First `given()...post("/pet")` block**: Sends a `POST /pet` request to create a new pet and asserts that the response contains the expected `id`, `name`, and `status`.
* **Second `given()...get("/pet/{id}", petId)` block**: Sends a `GET` request for the same pet ID and verifies that the pet can be retrieved with the same data.


### `testCreatePetAndVerify()`

```java
@Test
@DisplayName("Test POST /pet and GET /pet/{id} - Create a new pet and then verify its details")
void testCreatePetAndVerify() throws IOException {

    int uniquePetId = ThreadLocalRandom.current().nextInt(1_000_000, Integer.MAX_VALUE / 1000);
    String petName = "test-pet-" + uniquePetId;
    String petStatus = "pending";

    String requestBody = new String(Files.readAllBytes(Paths.get(PET_CREATE_JSON_PATH)));

    requestBody = requestBody.replace("{{id}}", String.valueOf(uniquePetId));
    requestBody = requestBody.replace("{{name}}", petName);
    requestBody = requestBody.replace("{{status}}", petStatus);

    // Create pet
    given()
        .log().all()
        .contentType(ContentType.JSON)
        .body(requestBody)
    .when()
        .post("/pet")
    .then()
        .log().all()
        .statusCode(200)
        .body("id", equalTo(uniquePetId))
        .body("name", equalTo(petName))
        .body("status", equalTo(petStatus));

    // Verify via GET
    given()
        .log().all()
    .when()
        .get("/pet/{id}", uniquePetId)
    .then()
        .log().all()
        .statusCode(200)
        .body("id", equalTo(uniquePetId))
        .body("name", equalTo(petName))
        .body("status", equalTo(petStatus));
}
```

* **`uniquePetId`**: A randomly generated integer used as the new pet’s ID.
* **`petName` and `petStatus`**: Dynamic values for the test pet.
* **`Files.readAllBytes(...)`**: Loads the JSON template from `pet_create.json`.
* **`requestBody.replace(...)`**: Replaces `{{id}}`, `{{name}}`, and `{{status}}` in the JSON template with the dynamic test values.
* **First `given()...post("/pet")` block**: Creates a new pet using the JSON request body and verifies the response fields.
* **Second `given()...get("/pet/{id}", uniquePetId)` block**: Retrieves the newly created pet and checks that the returned data matches what was created.

### `testInvalidPetReturns404()`

```java
@Test
@DisplayName("Test GET /pet/{id} - Verify 404 for a non-existent pet")
void testInvalidPetReturns404() {

    int nonExistentPetId = ThreadLocalRandom.current().nextInt(1_000_000, 2_000_000_000);

    // Ensure the pet does not exist
    given()
        .log().all()
    .when()
        .delete("/pet/{id}", nonExistentPetId)
    .then()
        .log().all()
        .statusCode(anyOf(equalTo(200), equalTo(404)));

    // Verify 404 on GET
    given()
        .log().all()
    .when()
        .get("/pet/{id}", nonExistentPetId)
    .then()
        .log().all()
        .statusCode(404);
}
```

* **`nonExistentPetId`**: A randomly generated integer ID used for a pet that should not exist.
* **`delete("/pet/{id}", nonExistentPetId)`**: Attempts to delete the pet first, ensuring that the ID does not exist.
* **`statusCode(anyOf(equalTo(200), equalTo(404)))`**: Accepts either response, since the pet may or may not have existed before deletion.
* **Final `get("/pet/{id}", nonExistentPetId)`**: Verifies that requesting this ID returns `404 Not Found`, confirming correct error handling.

## Lab (35 minutes)

**Objective:** Enhance an existing test with additional JSON Path assertions to deepen understanding of response validation.

**Task:**
Modify the `testCreatePetAndVerify()` method in `restassured-session1-playbook/src/test/java/com/example/restassured/PetApiTest.java`.

Add **one more JSON Path assertion** to the **second `then()` block** (the `GET` verification part) that validates a field from the `category` or `tags` object.

**Steps:**
1.  Open `PetApiTest.java`.
2.  Locate the `testCreatePetAndVerify()` method.
3.  Find the second `.then()` block (the one immediately after the `GET` request).
4.  Add a new `.body()` assertion.

**Hint:**
Refer to `pet_create.json` to see the structure of the `category` and `tags` objects. For example, if you wanted to assert the `name` within `category`, the JSON Path would be `"category.name"`. If you wanted to assert the `name` of the *first* tag, it would be `"tags[0].name"`.

**Acceptance Criteria:**
*   The `testCreatePetAndVerify()` method still passes successfully when `run_demo.bat` or `run_demo.sh` is executed.
*   The added assertion correctly validates a field (e.g., `category.name` or `tags[0].name`) from the `GET` response of the newly created pet.
*   The test demonstrates proper use of JSON Path for nested objects/arrays.

## Quiz (Minimum 5 questions with answers)

1.  **Question:** What are the three main keywords used in REST Assured's BDD style, and what does each represent?
    **Answer:** `given()` (Preconditions/Setup), `when()` (Action/Event), `then()` (Assertions/Outcomes).

2.  **Question:** How do you set the base URI for all your REST Assured tests in a reusable way?
    **Answer:** By setting `RestAssured.baseURI` in a `@BeforeAll` or `@BeforeEach` method in a base test class (e.g., `BaseTest.java`).

3.  **Question:** What method would you use in REST Assured to assert that an API call returned an HTTP 201 Created status code?
    **Answer:** `.statusCode(201)` within the `then()` block.

4.  **Question:** Explain the purpose of `.log().all()` in a REST Assured test. Where can it be used, and why is it beneficial?
    **Answer:** `.log().all()` is used to print all details of the HTTP request (when used after `given()`) or response (when used after `then()`) to the console. It's beneficial for debugging, understanding the exact data sent and received, and verifying headers, body, and status during test execution.

5.  **Question:** If a JSON response contains a field `{"user": {"address": {"city": "New York"}}}` and you want to assert the city, what would be the correct JSON Path assertion in REST Assured?
    **Answer:** `.body("user.address.city", equalTo("New York"))`

## Homework

1.  **Fork the Repository:** Create your own copy of the `restassured-session1-playbook` repository on GitHub (or equivalent).
2.  **Add One Negative Test:**
    *   Create a new test method in `PetApiTest.java` (e.g., `testDeleteNonExistentPetReturns404()`).
    *   This test should attempt to delete a pet that does not exist (using `DELETE /pet/{id}` with a clearly non-existent ID).
    *   Assert that the API returns the appropriate HTTP status code for such an action (e.g., 404 Not Found).
    *   Ensure request and response logging is enabled for this test.
3.  **Run `mvn test`:** Execute `mvn test` from your project's root directory to ensure all tests, including your new one, pass.

## How This Session Fits into a Larger REST Assured + BDD Learning Path

"Session 1 — REST Assured with BDD: Fundamentals" is the crucial entry point into efficient API test automation.

**Following Sessions could cover:**

*   **Session 2: Advanced JSONPath & Hamcrest Matchers:** Deep dive into more complex JSONPath expressions (e.g., filtering arrays, extracting multiple values) and a wider range of Hamcrest matchers for robust assertions.
*   **Session 3: Parameterization & Data-Driven Testing:** Techniques for running the same test with multiple sets of data using CSV, Excel, or external data sources, integrating with JUnit's parameterized tests.
*   **Session 4: Authentication & Authorization:** Handling different API security mechanisms (e.g., API Keys, Basic Auth, OAuth 2.0, Bearer Tokens) with REST Assured.
*   **Session 5: Custom Filters & Interceptors:** Implementing custom logic to modify requests/responses, such as logging to a file, adding custom headers dynamically, or handling error responses globally.
*   **Session 6: Integration with Reporting Tools:** Setting up and configuring test reporting (e.g., Allure Reports) to generate visually appealing and informative test execution results.
*   **Session 7: Building a Full BDD Framework (Cucumber/Serenity):** Integrating REST Assured with higher-level BDD frameworks like Cucumber or Serenity BDD to write Gherkin feature files, step definitions, and generate living documentation.
*   **Session 8: CI/CD Integration:** Discussing how to integrate API automation tests into a Continuous Integration/Continuous Deployment pipeline using tools like Jenkins, GitLab CI, or GitHub Actions.

This structured approach ensures learners build a strong foundation before progressively tackling more advanced and enterprise-grade API automation concepts.
