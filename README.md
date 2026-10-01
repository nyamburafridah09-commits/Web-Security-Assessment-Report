# Web-Security-Assessment-Report
## SQL Injection Testing – OWASP Juice Shop

**Prepared by:** Fridah Nyambura  
**Project:** Web Application Security Testing  
**Application:** OWASP Juice Shop  
**Testing Environment:** Local Docker Lab  
**Testing Tool:** Burp Suite Community Edition  

---

## 1. Executive Summary

This project involved a controlled security assessment of the OWASP Juice Shop web application running locally in a Docker-based laboratory environment.

The assessment focused on the product search functionality and examined how the application handled user-supplied input sent through the `q` search parameter.

Using Burp Suite, HTTP requests were intercepted and analyzed through the Proxy HTTP history and Repeater functionality. Normal search requests were compared with requests containing specially encoded characters.

The testing demonstrated that supplying a URL-encoded single quote to the search parameter caused the application to return an HTTP 500 Internal Server Error containing a SQLite database syntax error.

This behavior indicates that user-controlled input is reaching a database query and is not being handled safely. The exercise provided practical experience with HTTP request interception, parameter manipulation, response analysis, and identification of database-related error behavior.

---

# 2. Objectives

The objectives of this assessment were to:

- Understand how web applications process HTTP GET requests.
- Capture application traffic using Burp Suite.
- Identify and examine the product search endpoint.
- Analyze the `q` parameter.
- Compare normal and unexpected input.
- Observe application responses and HTTP status codes.
- Identify database-related error messages.
- Document the security finding professionally.
- Practice producing evidence suitable for a cybersecurity portfolio.

---

# 3. Scope

Testing was limited to the locally hosted OWASP Juice Shop application.

### In Scope

- Product search functionality
- `/rest/products/search` endpoint
- `q` search parameter
- HTTP request and response behavior
- Input handling
- Database error behavior

### Out of Scope

- External websites
- Real-world production systems
- Other users' data
- Infrastructure outside the local laboratory
- Destructive testing

---

# 4. Laboratory Environment

The application was hosted locally using Docker.

### Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux |
| Application | OWASP Juice Shop |
| Deployment | Docker |
| Application Port | 3000 |
| Testing Tool | Burp Suite |
| Browser | Web browser |
| Database behavior observed | SQLite |

The Juice Shop application was accessed through:

```text
http://localhost:3000
```

---

# 5. Tools Used

## Burp Suite

Burp Suite was used to:

- Intercept HTTP traffic
- View HTTP history
- Inspect requests
- Send requests to Repeater
- Modify request parameters
- Compare server responses

## Docker

Docker was used to run the OWASP Juice Shop application locally in an isolated testing environment.

## Browser

The browser was used to interact with the Juice Shop application and generate normal product-search requests.

---

# 6. Methodology

The assessment followed a controlled process:

1. Confirm that the Juice Shop application was running.
2. Perform a product search through the browser.
3. Capture the resulting HTTP request in Burp Suite.
4. Identify the product search endpoint.
5. Send the request to Burp Repeater.
6. Establish a normal request baseline.
7. Modify the `q` parameter.
8. Observe changes in HTTP status codes and response content.
9. Test unexpected characters.
10. Document the resulting database error.
11. Compare normal and abnormal responses.
12. Record the finding and recommended remediation.

---

# 7. Endpoint Discovery

A normal product search was performed using the browser.

Burp Suite HTTP history captured the following request:

```http
GET /rest/products/search?q=banana HTTP/1.1
Host: localhost:3000
```

The request was then sent to Burp Repeater for controlled testing.

### Screenshot 1 – HTTP History

<img width="2736" height="1824" alt="image" src="https://github.com/user-attachments/assets/22fc07ca-fdf4-4e62-9f40-aa31be009edd" />


*Figure 1: Burp Suite HTTP History showing the product search request.*

---

# 8. Baseline Testing

The original request used the search term `banana`:

```http
GET /rest/products/search?q=banana HTTP/1.1
```

The application successfully processed the request and returned product search data.

The normal response established a baseline for comparison with subsequent tests.

### Screenshot 2 – Normal Request

<img width="2736" height="1824" alt="image" src="https://github.com/user-attachments/assets/46fdcc7a-1afb-4265-8fed-719aa9406ef1" />

*Figure 2: Normal product search request and successful response.*

---

# 9. Empty Parameter Testing

The search parameter was then tested without a value:

```http
GET /rest/products/search?q=
```

The application returned a successful response containing a `data` field.

This demonstrated that the endpoint accepts an empty search parameter.

### Screenshot 3 – Empty Search

<img width="2736" height="1824" alt="image" src="https://github.com/user-attachments/assets/22e739a0-d70b-4a5a-8720-caa14be9bcf4" />


*Figure 3: Response to an empty product-search parameter.*

---

# 10. Unexpected Input Testing

The search parameter was then modified to include a URL-encoded single quote.

The following request was tested:

```http
GET /rest/products/search?q=banana%27 HTTP/1.1
Host: localhost:3000
```

The `%27` encoding represents a single quote character.

Instead of returning a normal search response, the application returned:

```text
HTTP/1.1 500 Internal Server Error
```

The response also exposed the following database error:

```text
SQLITE_ERROR: near "'%'": syntax error
```

### Screenshot 4 – SQLite Error

<img width="2736" height="1824" alt="image" src="https://github.com/user-attachments/assets/7a0b4a9a-10aa-4179-b537-f8b3328172d8" />


*Figure 4: Burp Repeater showing the HTTP 500 response and SQLite syntax error.*

---

# 11. Finding

## Finding: SQL Injection Indication in Product Search

**Endpoint:**

```text
/rest/products/search
```

**Parameter:**

```text
q
```

**Observed behavior:**

A normal search request was successfully processed, while adding a URL-encoded single quote caused the application to return an HTTP 500 Internal Server Error containing a SQLite syntax error.

This demonstrates that specially crafted input can influence the database-processing path and cause the underlying SQL operation to fail.

---

# 12. Evidence

The following evidence was collected during testing:

### Normal request

```http
GET /rest/products/search?q=banana
```

Result:

```text
Successful response
```

### Modified request

```http
GET /rest/products/search?q=banana%27
```

Result:

```text
HTTP/1.1 500 Internal Server Error
```

Database error:

```text
SQLITE_ERROR
```

The difference between the normal and modified requests demonstrates that the application's response changes significantly when unexpected input is introduced into the search parameter.

---

# 13. Security Impact

The observed behavior has several security implications.

### Database Error Disclosure

The application returned a database-specific SQLite error to the client.

Exposing internal database errors can provide attackers with information about the application's backend implementation.

### Input Handling Weakness

The behavior indicates that user-controlled input reaches database-processing logic in a way that can cause SQL syntax errors.

### Potential SQL Injection Risk

The observed behavior is consistent with an SQL injection weakness and warrants secure query handling.

Further exploitation was not performed as part of this portfolio exercise.

---

# 14. Recommended Remediation

The application should use **parameterized queries / prepared statements** when interacting with the database.

Instead of constructing SQL queries by concatenating user-supplied input, application developers should bind user input as query parameters.

Additional recommendations include:

- Validate and constrain user input where appropriate.
- Use parameterized database queries.
- Avoid dynamically constructing SQL statements from raw user input.
- Prevent database error details from being returned to users.
- Implement centralized error handling.
- Log detailed errors securely on the server side.
- Return generic error messages to clients.
- Conduct security testing on all endpoints accepting user-controlled input.

---

# 15. Lessons Learned

This project provided practical experience in several areas of web application security.

### HTTP Request Analysis

I learned how browser actions generate HTTP requests and how parameters are transmitted to web applications.

### Burp Suite

I practiced using:

- Proxy
- HTTP History
- Request inspection
- Send to Repeater
- Request modification
- Response analysis

### Parameter Testing

I learned how changing a single request parameter can produce significantly different application behavior.

### Error Analysis

The SQLite error demonstrated why server responses should be carefully examined during security testing.

### Security Documentation

I practiced documenting:

- Scope
- Methodology
- Evidence
- Findings
- Impact
- Remediation

---

# 16. Conclusion

The assessment successfully demonstrated a database-related input-handling weakness in the OWASP Juice Shop product-search functionality.

The testing began with a normal product search and established a successful baseline. Controlled modification of the `q` parameter then caused the application to return a SQLite syntax error and HTTP 500 response.

The exercise strengthened practical skills in web application testing, Burp Suite usage, HTTP analysis, input validation testing, and professional security reporting.

All testing was performed against a deliberately vulnerable application running locally in an authorized laboratory environment.

---

# 17. Portfolio Evidence

The following screenshots can be included with the final project:

1. **Juice Shop running locally**
2. **Burp Suite HTTP History**
3. **Normal product search request**
4. **Burp Repeater request**
5. **Successful normal response**
6. **Modified request containing `%27`**
7. **SQLite error response**
8. **Comparison of normal and abnormal responses**

These screenshots provide visual evidence that the testing was performed practically rather than being purely theoretical.

---

## Project Summary for GitHub

**Project:** Web Application Security Testing – OWASP Juice Shop

**Summary:**  
Performed a controlled web security assessment of a locally hosted OWASP Juice Shop application. Used Burp Suite to intercept and analyze HTTP requests, identify the product search endpoint, manipulate the `q` parameter, and investigate database-related error behavior. Documented the observed SQL injection indication, security impact, evidence, and recommended remediation.

**Skills Demonstrated:**

- Web Application Security
- HTTP/HTTPS
- Burp Suite
- HTTP Request Analysis
- Parameter Manipulation
- SQL Injection Testing
- SQLite Error Analysis
- Vulnerability Documentation
- Security Reporting
- Docker
- Kali Linux
- OWASP Juice Shop
