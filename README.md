# REST Assured Session 1 Playbook: Fundamentals

This project provides a hands-on introduction to REST Assured with a Behavior-Driven Development (BDD) style for testing REST APIs. It's designed to be a runnable project for training sessions and a reusable asset for API test automation.

**API Under Test:** [Swagger PetStore API](https://petstore.swagger.io/v2)

## Project Structure

```
restassured-session1-playbook/
├── pom.xml                 # Maven project configuration and dependencies
├── README.md               # This file
├── run_demo.bat            # Windows script to run the demo
├── run_demo.sh             # Linux/Mac script to run the demo
└── src
    └── test
        ├── java
        │   └── com/example/restassured
        │        ├── BaseTest.java   # Base class for common Rest Assured configurations
        │        └── PetApiTest.java # Contains the API test cases
        └── resources
            └── testdata
                └── pet_create.json # JSON payload template for creating a pet
```

## How to Run the Project

1.  **Prerequisites:**
    *   Java Development Kit (JDK) 11 or higher installed.
    *   Apache Maven installed and configured in your PATH.
    *   Internet connection (to reach the PetStore API).

2.  **Clone the Repository:**
    ```bash
    git clone <repository_url_here> # Replace with actual repo URL if hosted
    cd restassured-session1-playbook
    ```

3.  **Run the Demo:**

    *   **On Windows:**
        ```bash
        .\run_demo.bat
        ```
    *   **On Linux/Mac:**
        ```bash
        sh run_demo.sh
        ```

    The scripts will execute `mvn clean test`, which compiles the project, runs all tests in `PetApiTest.java`, and then prints a completion message.

## What Each Test Does

The `PetApiTest.java` class contains the following tests:

1.  **`testGetPetById()`**
    *   **Action:**
        *   Generates a random pet ID.
        *   Sends a `POST /pet` request to create a new pet.
        *   Sends a `GET /pet/{id}` request using the same ID.
    *   **Assertions:**
        *   Verifies both the `POST` and `GET` requests return `HTTP 200 OK`.
        *   Asserts that the `id`, `name`, and `status` fields in the `GET` response match the created pet.
    *   **Expected Output:**
        You will see logs for both the `POST` and `GET` requests. The final `GET` response will contain the pet that was just created.

2.  **`testCreatePetAndVerify()`**
    *   **Action:**
        *   Generates a unique pet id.
        *   Reads `pet_create.json` and dynamically replaces placeholders with the unique ID, a generated name, and a status.
        *   Makes a `POST` request to `/pet` with the crafted JSON body.
        *   If the creation is successful, it then makes a `GET` request to `/pet/{id}` using the newly created pet's ID.
    *   **Assertions:**
        *   Verifies the `POST` request returns `HTTP 200 OK`.
        *   Verifies the `GET` request returns `HTTP 200 OK`.
        *   Asserts that the `name` and `status` fields in the `GET` response match the values used during creation.
    *   **Expected Output:** You will see the request and response logs for both the `POST` and subsequent `GET` requests. Both should show `HTTP 200 OK` responses, and the final `GET` response body should reflect the pet details you created.

3.  **`testInvalidPetReturns404()`**
    *   **Action:**
        *   Generates a random pet ID.
        *   Sends a `DELETE /pet/{id}` request to ensure the pet does not exist.
        *   Sends a `GET /pet/{id}` request using the same ID.
    *   **Assertions:**
        *   Verifies the `GET` request returns `404 Not Found`.
    *   **Expected Output:**
        You will see a `DELETE` request followed by a `GET` request. The final response will show an `HTTP 404` status code.

## What the Learner is Supposed to See During the Demo

During the execution of `run_demo.bat` or `run_demo.sh`, learners should observe:

*   **Maven Build Output:** Standard Maven compilation and test execution logs.
*   **REST Assured Logging:** For each test, clear, human-readable logs of the outgoing HTTP request (method, URL, headers, body) and the incoming HTTP response (status, headers, body). This logging is crucial for understanding the API interaction.

*   **Test Results:** Clear indicators (typically green `[SUCCESS]` or similar) that each test method (`testGetPetById`, `testCreatePetAndVerify`, `testInvalidPetReturns404`) has passed successfully.
*   **Completion Message:** The final output "SESSION 1 DEMO COMPLETE" confirming the successful execution of the demo.

This visual feedback reinforces the concepts of request/response structure, HTTP status codes, and successful assertion of API behavior.
