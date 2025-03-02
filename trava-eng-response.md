# Trava Engineering Assessment – [Task Link](https://gist.github.com/mhoc/c4bfd297d14a8aeb108bfcbc2a346ee5)

### Candidate Name : [David Ajmed Hernandez](https://www.linkedin.com/in/david-ajmed-hernandez/)  

<br>

## Have a quick conversation with the CEO via Slack

> "Thanks for reaching out! I totally understand how frustrating those long URLs can be—especially when you're just trying to share a quick link with others. I'm glad you brought it up; I'd love to help make that experience better."

> "Right now, the URLs get very long because every filter (like `first_name`, `last_name`, and `age`) appears as query parameters. This is great for deep linking, but not so great for readability."

> "One approach is to build a feature that stores the filter parameters on our server, and in return we provide a short link that references that stored data. So instead of sending a huge URL, you can share something like `app.coolcompany.com/users/***`. When someone visits that link, our server will look up those parameters and load the same filters and sorting rules."

---

## Next Steps from an Engineering Process Perspective

### 1. Planning
Discuss with the CEO to validate the solution's requirements and overall approach (eg. security considerations, performance constraints, and system integration needs are fully addressed.)

### 2. Prioritization
Coordinate with product and engineering teams to assess where this feature fits in the current sprint or backlog, especially if it's urgent. Determine the MVP features versus enhancements that can be deferred.

### 3. Scheduling

**Milestones:**
- Implement **backend storage and short link generation**.  
- Update the **frontend** to generate and display short links.  
- Add **testing, security, and monitoring**.

**Tasks:**
- Create API endpoints for short URL generation and retrieval.
- Implement frontend integration for generating and sharing short links.
- Conduct testing, review, and deploy.


## Technical specs or proposals

### Overview
We want to improve the experience of sharing URLs from our user list page. Currently, every time a user applies filters (like first name, last name, age, etc.), the URL becomes very long and hard to share. Our goal is to create a short URL that still captures the filter state.

### Requirements
- Store filter parameters securely in the backend.
- Generate a short URL (`app.coolcompany.com/users/8**`) that references stored filters.
- Retrieve and apply filters when the short URL is accessed.
- Ensure **performance** is optimized and **security** measures prevent abuse.

### Proposed Approach
#### 1. Backend Service
- Create an API (`POST /shortenFilters`) to store filter data and return a short URL.
- Store filter data using a **unique ID** (e.g., shortened UUID).
- Retrieve filter data when a user visits the short URL (`GET /users/:shortId`).

#### 2. Frontend Integration
- When filters are applied, the frontend calls the API to generate a short link.
- Display the short URL and provide a "Copy Link" button.

#### 3. Retrieval Process
- When a user visits `app.coolcompany.com/users/***`, the backend fetches the filter data.
- The frontend applies the retrieved filters automatically.


#### Timeline and Milestones
- **MVP (Minimum Viable Product):**  
  - Build the backend endpoint to store and retrieve filter states.  
  - Generate a short URL.

- **Integration:**  
  - Connect the frontend to the backend service to create and display the short URL.

- **Refinements:**  
  - Add tests (unit and integration), enhance security, and set up monitoring.

#### Risks and Considerations
- **Security Risks:**  
  - Ensure the short URL does not expose sensitive information.
  - Implement measures to prevent URL tampering and brute-force ID guessing.
- **Performance Constraints:**  
  - Evaluate and optimize the performance of the filter state retrieval.
  - Monitor the load on the storage system if usage increases.
- **Integration Points:**  
  - Confirm that the new endpoints work well with the existing user list page.
  - Plan for any necessary changes to the UI for a smooth user experience.

#### Future Enhancements
- **Expiration policy:** Remove unused short URLs after a set time.
- **Usage analytics:** Track how often short URLs are created and accessed.
- **User authentication:** Restrict access to specific short URLs.
- **Saved filters feature:** Allow users to save and name filter presets.


### Backend Service

API Routes

POST /shortenFilters
```json
{
    "first_name": "John",
    "first_name_op": "eq",
    "last_name": "Snow",
    "last_name_op": "neq",
    "age": 30,
    "age_op": "gte",
    "sort": "age",
    "sort_dir": "asc"
}
```
response

```json
{
  "shortUrl": "https://app.coolcompany.com/users/***"
}
```

GET /users/:shortId
```bash
GET /u/abc123
```
resopnse
```json
search result
{
    "first_name": "John",
    "first_name_op": "eq",
    "last_name": "Snow",
    "last_name_op": "neq",
    "age": 30,
    "age_op": "gte",
    "sort": "age",
    "sort_dir": "asc"
}
```

simple implementation
```js
app.post('/shortenFilters', (req, res) => {
  const filterData = req.body;
  const shortId = uuidv4().slice(0, 8);
  store[shortId] = filterData;
  const shortUrl = `https://app.coolcompany.com/u/${shortId}`;
  res.json({ shortUrl });
});

app.get('/u/:id', (req, res) => {
  const shortId = req.params.id;
  const filterData = store[shortId];
  if (filterData) {
    res.json(filterData);
  } else {
    res.status(404).json({ error: 'Filter state not found' });
  }
});
```


## Ongoing Maintenance and Monitoring

### 1. Maintenance Requirements

- **Storage Management:** If using a database or cache (e.g., Redis), we need a cleanup mechanism to remove stale or unused short URLs over time.
- **Security Updates:** Regularly update dependencies and libraries to address potential vulnerabilities.
- **Performance Optimization:** Monitor query performance and optimize storage and retrieval mechanisms as usage scales.
- **Bug Fixes & Enhancements:** Address any reported issues and continuously improve based on user feedback.

### 2. Monitoring & Metrics

#### Business-Facing Metrics
- **Short URL Usage:** Track how often users generate short URLs and how frequently they are accessed.
- **Filter Popularity:** Log which filters are most commonly applied to provide insights into user behavior.
- **Retention & Engagement:** Analyze how often users revisit shared links to understand the impact on collaboration.

#### Engineering/Operational Metrics
- **Error Tracking:** Set up logging and error monitoring (e.g., Sentry, Datadog) to detect and resolve issues quickly.
- **Response Time:** Monitor API latency to ensure fast retrieval of filter states.
- **Storage Utilization:** Track database/cache usage to prevent excessive storage growth.
- **Rate Limiting & Abuse Prevention:** Monitor for potential misuse (e.g., automated requests trying to generate or access thousands of short URLs).

### 3. Automation & Alerts
- **Health Checks:** Implement automated tests to verify the service is functioning correctly.
- **Alerts for Failures:** Set up alerts if API failures exceed a threshold, storage reaches critical levels, or response times degrade.
- **Logging & Auditing:** Maintain logs of short URL creation and access events for debugging and security purposes.


## automate testing

### JavaScript/Node.js - Using Jest & Supertest
For a **Node.js backend with Express**, we can use **Jest** (for unit testing) and **Supertest** (for API testing).

### Automate Testing with GitHub Actions
```yaml
name: Run Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v14
      - run: npm install
      - run: npm test
```

### Python (Django/Flask) – Using Pytest
If the backend is in Python (Django or Flask), we can use Pytest for testing.

### Java (Spring Boot) – Using JUnit & RestAssured
If using Java with Spring Boot, we can automate tests using JUnit and RestAssured.

### Frontend (React, Angular, Vue) – Using Cypress
If the short URL functionality is integrated into a React, Angular, or Vue frontend, Cypress can be used for end-to-end testing.

### API Testing with Postman/Newman
If the project involves RESTful APIs, we can use Postman collections and automate them using Newman.

```bash
newman run ShortUrlCollection.json --reporters cli
```

```yaml
name: API Testing
on: [push]
jobs:
  test-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: matt-ball/newman-action@v1
        with:
          collection: ShortUrlCollection.json
```

### Performance & Load Testing – Using k6

```bash
#windows
choco install k6
#mac
brew install k6
```

# Improving the App Without Adding New Features

If improving the app is more suitable for this proejct, then I can **make the existing URLs more compact and readable** instead of creating a short URL service.
- **Use shorter parameter names** (e.g., `fn=John` instead of `first_name=John`).
- **Remove redundant operators** (e.g., `age>=30` can be `a=30g`).
- **Apply URL encoding** only when necessary to handle special characters properly.

For example
```code
https://app.coolcompany.com/users?fn=John&ln=Snow&a=30&s=a_asc
```

## **Implement "Saved Filters" Instead of Short URLs**

- Users apply filters and **save them with a name** (e.g., "Marketing View").
- The system stores a **saved filter ID** in the database.
- The URL **only references the saved filter ID**, keeping it short.

for example
```code
https://app.coolcompany.com/users?saved_filter=123
```

## Improve Frontend State Management Instead of Using URLs**
If URL length is a concern, I can **store filters in session storage** instead of appending them to the URL.

- Use **browser history (pushState API)** to update filters without modifying the URL.
- Store filters in **local storage or session storage** for quick access.
- Provide a **“Copy Link” button** only when users want to share a filter state.


