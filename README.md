OWASP Juice Shop — SQL Injection Investigation

Author: Ologan Oluwadamola Andrew
Project Type: Application Security / Vulnerability Assessment
Environment: Kali Linux, Docker, OWASP Juice Shop
Tools: Burp Suite Community Edition, curl, Node.js
Target: Local OWASP Juice Shop training application
Endpoint: GET /rest/products/search

⸻

1. Project Overview

This project documents an authorized cybersecurity lab investigation of the product-search functionality in OWASP Juice Shop, an intentionally vulnerable web application used for security training.

The objective was to investigate how user-controlled search input is processed, inspect the application’s SQL query construction, observe HTTP responses, identify potential security weaknesses, and document appropriate remediation.

The investigation focused on SQL injection risk and database-error disclosure in a controlled local environment.

2. Objectives

* Understand how web application search requests interact with a database.
* Inspect application source code for unsafe SQL query construction.
* Test normal and specially crafted search inputs against the local lab.
* Observe HTTP responses and database errors.
* Document evidence and potential security impact.
* Recommend secure coding practices to prevent SQL injection.

3. Lab Environment

The investigation used the following environment and tools:

* Kali Linux: Security testing operating system.
* Oracle VirtualBox: Virtual machine environment.
* Docker: Containerized deployment of OWASP Juice Shop.
* OWASP Juice Shop: Intentionally vulnerable web application.
* Burp Suite Community Edition: Web request inspection and manual testing.
* curl: Command-line HTTP requests.
* Node.js: Application source-code inspection.

The local application was accessed at:

http://127.0.0.1:3000

All testing was intended for the local training application.

4. Vulnerability Investigation

Finding: Unsafe SQL query construction / SQL injection risk

Affected endpoint: GET /rest/products/search

User-controlled input: q query parameter

During source-code inspection, the search criteria was found to be inserted directly into a SQL query string rather than consistently handled as a bound parameter.

This pattern can allow specially crafted input to alter the structure of a database query, depending on the application’s behavior and database configuration.

Source-code location

/juice-shop/build/routes/search.js

Evidence collected

The investigation recorded the following observations:

1. A normal product search returned HTTP 200.
2. A search containing a single quotation mark also returned HTTP 200. This result alone does not establish whether SQL injection is possible.
3. A previously tested crafted search input produced an SQLite SQLITE_ERROR: incomplete input message and an HTTP 500 response.
4. Source inspection confirmed unsafe SQL string construction.

Validation status: Unsafe SQL construction was identified in the inspected source. Successful SQL injection and sensitive-data extraction were not confirmed by the recorded tests.

5. Reproduction in the Local Lab

First, confirm that the local OWASP Juice Shop container is running and that the application is reachable.

Normal search request:

curl -sG "http://127.0.0.1:3000/rest/products/search" \
  --data-urlencode "q=apple"

Previously observed error-producing request:

curl -sG "http://127.0.0.1:3000/rest/products/search" \
  --data-urlencode "q= ' OR 1=1-- "

The exact response may vary depending on the application version and input handling. These commands are intended only for the local training instance or another explicitly authorized environment.

6. Potential Security Impact

Unsafe SQL query construction may expose an application to SQL injection if user input can change the intended query structure.

Depending on exploitability, database permissions, and application behavior, potential consequences may include:

* Manipulation of search results.
* Unauthorized access to database records.
* Exposure of information available to the database account.
* Application errors that reveal internal database details.

These are potential impacts, not confirmed outcomes of this investigation.

7. Recommended Remediation

Use parameterized queries

Pass user input as bound parameters instead of concatenating it directly into SQL statements.

Illustrative Sequelize-style pattern:

models.sequelize.query(
  "SELECT * FROM Products WHERE " +
  "(name LIKE :pattern OR description LIKE :pattern) " +
  "AND deletedAt IS NULL ORDER BY name",
  {
    replacements: {
      pattern: `%${criteria}%`
    }
  }
);

The example should be validated against the installed application’s ORM version and intended search behavior before use.

Improve error handling

* Avoid returning detailed database errors to clients.
* Log diagnostic information securely on the server.
* Return appropriate generic error responses.

Perform regression testing

After implementing a fix:

* Confirm normal product searches still work.
* Verify that quotation marks and SQL metacharacters are treated as input data.
* Test that user input cannot change the intended SQL query structure.
* Record the results before declaring the vulnerability fixed.

8. Key Learning Outcomes

This project provided practical experience with:

* Web application security testing.
* HTTP request and response inspection.
* SQL injection concepts and risk identification.
* Source-code review.
* Docker-based security labs.
* Vulnerability documentation.
* Risk assessment and secure coding recommendations.
* Communicating findings without overstating the available evidence.

9. Project Status

* [x]	Local application and endpoint identified.
* [x]	Relevant application source inspected.
* [x]	Normal search behavior recorded.
* [x]	Database error response documented.
* [x]	Security risk and remediation recommendations documented.
* [ ]	Add sanitized screenshots from the actual lab session.
* [ ]	Implement a fix in a controlled copy, if required.
* [ ]	Perform and document post-fix regression testing.

10. Conclusion

This investigation demonstrates the importance of reviewing how applications handle user-controlled input before passing it to database queries.

The source-code evidence identified unsafe SQL query construction, and testing recorded a database error response. Further repeatable testing and post-remediation verification are required before claiming confirmed exploitation or a successfully fixed vulnerability.

Author: Ologan Oluwadamola Andrew

Authorized-use notice: This project documents an educational security investigation of an intentionally vulnerable local application. Do not test systems without explicit authorization.
