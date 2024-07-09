# Guide to Running the Project

## Prerequisites
Make sure you have Docker and Docker Compose installed on your system.


## Steps to Run the Project

1. **Clone the Repository:**
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Navigate to the Project Directory:**
```bash
cd <project-directory>
```
Ensure you are in the directory where your docker-compose.yml file is located.

3. **Create Necessary Directories:**

Create the following directories if they do not exist already in your project root:
```bash
docker-volumes/postgres
docker-volumes/static
docker-volumes/media
```
These directories are used for persisting PostgreSQL data and storing static/media files.

4. **Set Environment Variables:**
Open the .env file in the project directory and set the necessary environment variables. For example:
```html
DEBUG=False
EMAIL_HOST_USER=****
EMAIL_HOST_PASSWORD=****
DATABASES_NAME=owltrace
DATABASES_USER=owltrace
DATABASES_PASSWORD=****
GOOGLE_CLIENT_ID=****
GOOGLE_SECRET=****
```
Adjust values as per your configuration.
5. **Run Docker Compose:**
Execute the following command to start the containers defined in docker-compose.yml:
```bash
docker-compose up -d
```
The -d flag runs containers in detached mode, meaning they will run in the background.

6. **Access the Application:**
Once Docker Compose has successfully started the containers, you can access the application:

    **Web Application:** Navigate to http://localhost in your web browser.
    **Admin Panel:** The admin panel is set up automatically. Use the superuser credentials defined in the owltrace service configuration.
    ```html
      email: admin@admin.com
      password: password
    ```
7. **Stopping the Application:**
To stop the application and shut down the containers, run:
```bash
docker-compose down
```
This command will stop and remove containers, networks, volumes, and other artifacts created by **docker-compose up**.
8. **Additional Notes:**
Ensure no other services are using ports 5432, 80, or 443 on your host machine to avoid conflicts with PostgreSQL, nginx, and the web application respectively.
Make sure your firewall allows traffic on these ports if you are running on a remote server or have strict firewall rules.
That's it! You should now have your Dockerized Django application running successfully.


Make sure to replace `<repository-url>`, `<repository-directory>`, and `<project-directory>` with the actual URLs and paths relevant to your project. This guide assumes basic familiarity with Docker and Docker Compose commands. Adjust the instructions according to any specific setup or requirements you might have.



# JWT Authentication Guide

This guide will help you understand how to authenticate using JWT (JSON Web Tokens) in your frontend application and interact with the Django backend API using the provided endpoints.

## API Endpoints

The following endpoints are available for obtaining and refreshing JWT tokens:

- **Obtain Token Pair**: `POST /api/token/`
- **Refresh Token**: `POST /api/token/refresh/`

### Obtain Token Pair

To obtain an access token and a refresh token, send a POST request to `/api/token/` with the user's credentials (username and password).

**Request Example**:

```http
POST /api/token/
Content-Type: application/json

{
  "username": "your_username",
  "password": "your_password"
}
```


**Response Example**:

```http
{
  "access": "your_access_token",
  "refresh": "your_refresh_token"
}
```

### Use Access Token to Access APIs
Once you have obtained the access token, you can use it to authenticate subsequent API requests. Include the access token in the Authorization header as a Bearer token.

**Request Example**:

```http
GET /your/api/endpoint/
Authorization: Bearer your_access_token
```

#### Refresh Token:
The access token has a limited lifespan. When it expires, you can obtain a new access token using the refresh token. Send a POST request to /api/token/refresh/ with the refresh token.
**Request Example**:

```http
POST /api/token/refresh/
Content-Type: application/json

{
  "refresh": "your_refresh_token"
}
```

**Request Example**:
```http
{
  "access": "your_new_access_token"
}
```

#### Handling Token Expiry:
To handle token expiry, your frontend application should:

    1- Store the tokens securely after obtaining them from the backend.
    2- Attach the access token to each API request in the Authorization header.
    3- Monitor the access token's validity and refresh it using the refresh token when it expires.

#### General Steps:
**1. Store Tokens:** Save the access and refresh tokens securely, for example in a session or local storage.
**2. Add Authorization Header:**  For each API request, add the Authorization: Bearer <access_token> header.
**3. Refresh Access Token:**

    When receiving a 401 Unauthorized response, indicating that the access token has expired, use the refresh token to request a new access token.
    Send a POST request to /api/token/refresh/ with the refresh token.
    Replace the old access token with the new one in your storage.

**4.Repeat Steps:** Continue to use the new access token for subsequent requests.
By following this guide, you will be able to authenticate users, handle token expiry, and make authenticated requests to your Django backend using JWT in your frontend application, regardless of the specific frontend technology used.




#  `/app/api/submit-analyze/` API

## Overview
The `/app/api/submit-analyze/` API endpoint is used to submit analysis data along with an image UUID, title, and type of analysis. The input data is validated against a JSON schema (`json_schema1`) to ensure correctness before processing.

## API Endpoint
- **Endpoint:** `/app/api/submit-analyze/`
- **Method:** POST
- **Authentication:** Required (Token-based authentication)
- **Permissions:** The user must have `is_ml` set to `true` 

## User Model Requirement
To access this API endpoint, the user must have the `is_ml` field set to `true` in their user model. This field indicates whether the user is an ML (Machine Learning) or Analyzer Machine user.


## Input Parameters
The API expects the following parameters in the request body:

- `data`: JSON object containing the analysis data. It must conform to the `json_schema1` schema defined in your project.
- `image_uuid`: UUID of the image associated with the analysis.
- `title`: Title of the analysis.
- `type`: Type of the analysis.

## JSON Schema Validation
The input data (`data` field) is validated against `json_schema1` using `jsonschema.validate()` function. If validation fails, a `ValidationError` is raised with details about the validation error.

## JSON Schema (`json_schema1`)
```json
 {
  "type": "object",
  "properties": {
    "title": {"type": "string"},
    "is_fault": {"type": "boolean"},
    "farm_id": {"type": "string","format": "uuid"},
    "overall_value": {
      "type": ["object", "null"],
      "properties": {
        "value": {"type": "number"},
        "color": {"type": ["string", "null"]},
        "max": {"type": ["number", "null"],"default": 1},
        "min": {"type": ["number", "null"],"default": 0}
      },
      "required": ["value"]
    },
    "details": {
      "type": "object",
      "properties": {
        "type": {"type": "string","enum": ["sparse", "continuous"]},
        "max": {"type": ["number", "null"],"default": 1},
        "min": {"type": ["number", "null"],"default": 0},
        "points": {"type": "array",
          "items": {
            "type": "object",
            "properties": {
              "value": {"type": "number"},
              "color": {"type": ["string", "null"]},
              "location": {"type": "object",
                "properties": {
                  "lat": {"type": "number"},
                  "long": {"type": "number"}
                },
                "required": ["lat", "long"]
              }
            },
            "required": ["value", "location"]
          }
        }
      },
      "required": ["type", "points"]
    }
  },
  "required": ["title", "farm_id", "details", "is_fault"]
}
```

## Request Example
```json
{
  "data": {
    "title": "Sample Analysis",
    "is_fault": true,
    "farm_id": "123e4567-e89b-12d3-a456-426614174000",
    "overall_value": {
      "value": 0.8,
      "color": "#FFA500",
      "max": 1.0,
      "min": 0.0
    },
    "details": {
      "type": "sparse",
      "max": 1.0,
      "min": 0.0,
      "points": [
        {
          "value": 0.6,
          "color": "#00FF00",
          "location": {
            "lat": 45.123456,
            "long": -75.123456
          }
        },
        {
          "value": 0.4,
          "color": null,
          "location": {
            "lat": 46.123456,
            "long": -76.123456
          }
        }
      ]
    }
  },
  "image_uuid": "123e4567-e89b-12d3-a456-426614174001",
  "title": "Sample Analysis",
  "type": "Agricultural Analysis"
}
```
##### Description of "is_fault": true in Analysis

In the context of the Django application, the `"is_fault": true` attribute is used to determine whether an analysis should be displayed in an "EZ view". This feature is designed to highlight analyses that are flagged as having potential faults or issues that require attention.

###### Functionality
- When `"is_fault": true` is set for an analysis, it indicates that the analysis is considered to have a fault or issue.
- Analyses marked with `"is_fault": true` are specifically filtered and shown in the EZ view interface.


###### Usage
- **Backend Logic:** Within the backend of the Django application, filtering or querying analyses with `"is_fault": true` ensures that only relevant analyses are fetched for display in the EZ view.


**Response**:
Upon successful submission and processing of the analysis data, the API responds with:
```http
{
  "status": 201
}
```
If the title provided does not match any valid analysis type (AnalyzeType), the API responds with:
```html
{
  "error": "'{title}' is not a valid Analyze title."
}
```
**Error Handling**:
**400 Bad Request:** If the request does not conform to the expected format or schema.
**404 Not Found:** If the image_uuid does not correspond to any existing FieldImages.

# `/app/api/pending-analysis/` API

## Overview
The `/app/api/pending-analysis/` API endpoint retrieves pending analysis records from your Django application. It requires authentication and specific permissions to access.

## API Endpoint
- **Endpoint:** `/app/api/pending-analysis/`
- **Method:** GET
- **Authentication:** Required (Token-based authentication)
- **Permissions:** Requires authentication with `IsAuthenticated` and either `SuperUserOr_ML_Permission` permissions.

## Response
The API endpoint returns a list of pending analysis records in JSON format. Each record includes:
- `uuid`: Unique identifier of the analysis.
- `field_image.file`: URL to the associated image file (if available).
- `field_image.file_uuid`: UUID of the associated image file.
- `title`: Title of the analysis.
- `status`: Current status of the analysis (e.g., "Pending").
- `created_at`: Timestamp when the analysis record was created.
- `updated_at`: Timestamp when the analysis record was last updated.

## Example Response
```json
[
  {
    "uuid": "123e4567-e89b-12d3-a456-426614174001",
    "field_image": {
      "file": "http://example.com/media/zip_files/123e4567-e89b-12d3-a456-426614174002.zip",
      "file_uuid": "123e4567-e89b-12d3-a456-426614174002"
    },
    "title": "Sample Analysis",
    "status": "Pending",
    "created_at": "2024-06-27T12:00:00Z",
    "updated_at": "2024-06-27T12:30:00Z"
  },
  {
    "uuid": "123e4567-e89b-12d3-a456-426614174003",
    "field_image": {
      "file": "http://example.com/media/zip_files/123e4567-e89b-12d3-a456-426614174004.zip",
      "file_uuid": "123e4567-e89b-12d3-a456-426614174004"
    },
    "title": "Another Analysis",
    "status": "Pending",
    "created_at": "2024-06-26T14:00:00Z",
    "updated_at": "2024-06-26T14:30:00Z"
  }
]
```
**Error Handling**:
**401 Unauthorized:**  If the request lacks proper authentication credentials.
**403 Forbidden:** If the authenticated user does not have the required permissions **(SuperUserOr_ML_Permission)**.

