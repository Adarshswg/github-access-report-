# GitHub Access Report Service

A Spring Boot service that connects to GitHub and generates comprehensive reports showing which users have access to which repositories within a given organization.

## Features

- **Secure Authentication**: Uses GitHub Personal Access Tokens for secure API access
- **Scalable Design**: Efficiently handles organizations with 100+ repositories and 1000+ users
- **Parallel Processing**: Built with Spring Web MVC for concurrent API calls
- **Rate Limiting**: Intelligent rate limiting to respect GitHub API limits
- **Comprehensive Reporting**: Detailed access information including permission levels
- **Error Handling**: Robust error handling with meaningful error messages
- **RESTful API**: Clean REST API endpoints

## Architecture

The service is built using:
- **Spring Boot 2.7.18** with Spring Web MVC for traditional programming
- **OkHttp** for efficient HTTP client communication with GitHub API
- **Lombok** for reducing boilerplate code
- **Jackson** for JSON serialization/deserialization

## Prerequisites

- Java 8 or higher
- Maven 3.6 or higher
- GitHub Personal Access Token with appropriate permissions

## Setup Instructions

### 1. Generate GitHub Personal Access Token

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Give it a descriptive name (e.g., "Access Report Service")
4. Select the following scopes:
   - `repo` (Full control of private repositories)
   - `read:org` (Read org and team membership)
   - `read:user` (Read user profile data)
5. Click "Generate token" and copy the token (you won't see it again)

### 2. Configure the Application

#### Option 1: Environment Variable (Recommended)
```bash
export GITHUB_TOKEN=your_github_token_here
```
**On Windows:**
```cmd
set GITHUB_TOKEN=your_github_token_here
```

#### Option 2: application.yml
Edit `src/main/resources/application.yml`:
```yaml
github:
  token: your_github_token_here
  api-url: https://api.github.com
  connect-timeout: 30000
  read-timeout: 30000
  write-timeout: 30000
  max-requests: 100
  request-window-ms: 60000
```

### 3. Build and Run the Application

```bash
# Build the application
mvn clean compile

# Run the application
mvn spring-boot:run
```

**On Windows (using Maven wrapper):**
```cmd
set JAVA_HOME=C:\path\to\java8
set PATH=%JAVA_HOME%\bin;%PATH%
mvnw.cmd spring-boot:run
```

Alternatively, you can run the JAR directly:
```bash
java -jar target/github-access-report-0.0.1-SNAPSHOT.jar
```

The application will start on port 8080 by default.

## API Usage

### Generate Access Report

**Endpoint:** `GET /api/v1/access-report/{organization}`

**Parameters:**
- `organization` (path): The GitHub organization name
- `includeInactive` (query, optional): Include inactive users (default: false)

**Example Request:**
```bash
curl -X GET "http://localhost:8080/api/v1/access-report/my-org"
```

**Response Format:**
```json
{
  "organization": "my-org",
  "generatedAt": "2024-01-15T10:30:00",
  "totalRepositories": 150,
  "totalUsers": 1250,
  "userAccessList": [
    {
      "user": {
        "id": 123456,
        "login": "john.doe",
        "name": "John Doe",
        "email": "john.doe@example.com",
        "type": "User",
        "htmlUrl": "https://github.com/john.doe",
        "avatarUrl": "https://avatars.githubusercontent.com/u/123456?v=4",
        "siteAdmin": false
      },
      "repositories": [
        {
          "repositoryName": "awesome-project",
          "repositoryFullName": "my-org/awesome-project",
          "repositoryUrl": "https://github.com/my-org/awesome-project",
          "privateRepo": true,
          "permissions": {
            "admin": false,
            "push": true,
            "pull": true,
            "triage": false,
            "maintain": false
          },
          "accessLevel": "write"
        }
      ]
    }
  ]
}
```

### Health Check

**Endpoint:** `GET /api/v1/health`

**Example Request:**
```bash
curl -X GET "http://localhost:8080/api/v1/health"
```

**Response:** `Service is healthy`

## Design Decisions

### 1. Java 8 Compatibility
- **Why**: Ensures broader compatibility and wider adoption
- **Benefit**: Can run on older systems and enterprise environments

### 2. Spring Web MVC over WebFlux
- **Why**: Simpler programming model, better community support
- **Benefit**: Easier to understand and maintain for most developers

### 3. Parallel Processing
- **Implementation**: Uses `parallelStream()` for concurrent API calls
- **Benefit**: Significantly reduces total API call time for large organizations

### 4. Rate Limiting
- **Implementation**: Custom rate limiting service to respect GitHub's API limits
- **Benefit**: Prevents API abuse and ensures reliable operation

### 5. Pagination Handling
- **Implementation**: Automatic pagination for all GitHub API endpoints
- **Benefit**: Handles organizations with any number of repositories/users

### 6. Error Handling
- **Implementation**: Global exception handler with detailed error responses
- **Benefit**: Clear error messages and proper HTTP status codes

## Performance Considerations

### For Large Organizations (100+ repos, 1000+ users):

1. **Concurrent API Calls**: The service makes parallel calls to fetch repository data
2. **Rate Limiting**: Automatically respects GitHub's 5000 requests/hour limit for authenticated requests
3. **Memory Efficiency**: Uses streaming to handle large datasets without loading everything into memory
4. **Timeout Configuration**: Configurable timeouts to prevent hanging requests

### Estimated Performance:
- **Small org** (< 50 repos, < 200 users): ~30-60 seconds
- **Medium org** (50-100 repos, 200-1000 users): ~2-5 minutes  
- **Large org** (100+ repos, 1000+ users): ~5-15 minutes

## Security Considerations

1. **Token Security**: Never commit GitHub tokens to version control
2. **HTTPS**: Always use HTTPS for API calls
3. **Least Privilege**: Use tokens with minimal required permissions
4. **Token Rotation**: Regularly rotate your GitHub tokens

## Troubleshooting

### Common Issues:

1. **401 Unauthorized**
   - Check that your GitHub token is valid and has the required permissions
   - Ensure the token hasn't expired

2. **403 Rate Limit Exceeded**
   - The service includes rate limiting, but you may need to adjust the configuration
   - Consider using GitHub App authentication for higher limits

3. **404 Not Found**
   - Verify the organization name is correct
   - Ensure your token has access to the organization

4. **Empty Results**
   - Some organizations (like Microsoft) restrict API access
   - Try with smaller, public organizations first

5. **Timeout Errors**
   - Increase timeout values in configuration for large organizations
   - Check network connectivity to api.github.com

### Debug Logging

Enable debug logging by updating `application.yml`:
```yaml
logging:
  level:
    com.example.github_access_report: DEBUG
    okhttp3: DEBUG
```

## Development

### Running Tests
```bash
mvn test
```

### Building for Production
```bash
mvn clean package -DskipTests
```

### Docker Support (Optional)

Create a `Dockerfile`:
```dockerfile
FROM openjdk:8-jdk-alpine
COPY target/github-access-report-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

Build and run:
```bash
docker build -t github-access-report .
docker run -p 8080:8080 -e GITHUB_TOKEN=your_token_here github-access-report
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues and questions:
1. Check the troubleshooting section above
2. Review the GitHub API documentation: https://docs.github.com/en/rest
3. Create an issue in the repository with detailed information about your problem
