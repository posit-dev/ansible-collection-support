# Changelog

All notable changes to the semaphore-api-get role will be documented in this file.

## [3.0.0] - 2026-02-03

### Added
- **New Endpoints**:
  - `projects` - List all projects (global endpoint, no project_id required)
  - `users` - List all users (admin only, global endpoint)
  - `user` - Get current authenticated user info (global endpoint)
  - `events` - Project activity/event log
  - `views` - Project views for organizing templates
  - `task_output` - Get output/logs for a specific task (requires resource_id)

- **Global Endpoint Support**:
  - Added support for endpoints that don't require project_id
  - Automatic validation skips project_id check for global endpoints
  - Clear distinction between project-scoped and global endpoints in documentation

- **Enhanced Error Handling**:
  - Human-readable error messages for common HTTP status codes (400, 401, 403, 404, 405, 408, 429, 5xx)
  - Contextual troubleshooting suggestions based on error type
  - Improved error output with status-specific guidance

- **Response Metadata**:
  - New `semaphore_api_last_response` fact with request metadata
  - Includes: status code, elapsed time, URL, endpoint name, content-type
  - Configurable via `semaphore_api_store_metadata` (default: true)

- **New Configuration Options**:
  - `semaphore_api_expected_status_codes` - List of acceptable HTTP status codes
  - `semaphore_api_validate_json` - Toggle JSON validation (default: true)
  - `semaphore_api_store_metadata` - Toggle metadata storage (default: true)

- **Generic Endpoint Handler**:
  - New `generic_endpoint.yml` task using configuration-driven approach
  - Endpoint configuration defined in `vars/main.yml`
  - Reduces code duplication and simplifies adding new endpoints

### Changed
- **Architecture Overhaul**:
  - Replaced individual endpoint task files with generic handler
  - Endpoint configuration now centralized in `vars/main.yml`
  - Main task file reduced from 69 lines to 52 lines
  - Simplified routing using configuration lookup

- **Improved Validation**:
  - Project_id validation only runs for project-scoped endpoints
  - Resource_id validation for endpoints that require it (task_output)
  - Better handling of endpoint requirements

- **Enhanced Debug Output**:
  - Added HTTP status code to response summary
  - Added response time to debug output
  - Improved verbose debug with full URL display

- **Updated Documentation**:
  - Reorganized README with project-scoped vs global endpoint sections
  - Added 12 example playbooks covering all new features
  - Updated return values table with all 13 endpoints
  - Added response metadata documentation

### Technical Improvements
- **Code Reduction**: Main routing logic reduced by 75%
- **Configuration-Driven**: All endpoint paths and response variables defined in vars
- **Better Maintainability**: Adding new endpoints requires only vars update
- **HTTP Status Mapping**: Centralized status code to message mapping

## [2.0.0] - 2026-02-03

### Added
- **New Endpoints Support**:
  - Keys endpoint (`keys`) - Retrieve SSH keys and access keys
  - Templates endpoint (`templates`) - Retrieve task templates
  - Schedules endpoint (`schedules`) - Retrieve scheduled tasks

- **Error Handling and Retry Logic**:
  - Automatic retry mechanism for failed API calls (default: 3 retries)
  - Configurable retry delay between attempts (default: 5 seconds)
  - Comprehensive error messages with status codes and API error details
  - Clear validation failure messages

- **Query Parameters Support**:
  - `semaphore_api_query_params` variable to add custom query parameters to API requests
  - Support for sorting, filtering, and pagination
  - Automatic URL encoding of query parameters

- **Debug Output Control**:
  - `semaphore_api_debug` variable to enable/disable standard debug output (default: true)
  - `semaphore_api_verbose_debug` variable for detailed debug information including full API responses
  - Configurable output levels for different use cases

- **Response Validation**:
  - Automatic validation of HTTP status codes
  - JSON format validation for all API responses
  - Detection and reporting of API error messages

- **Common API Call Task**:
  - Created `common_api_call.yml` to reduce code duplication
  - Centralized authentication handling (token and basic auth)
  - Consistent error handling across all endpoints

### Changed
- **Refactored All Endpoint Tasks**:
  - Simplified individual endpoint files (inventory, task, variable_groups, repository)
  - All endpoints now use the common API call task
  - Reduced code from ~45 lines to ~8 lines per endpoint
  - Improved maintainability and consistency

- **Enhanced Validation**:
  - Added validation for authentication credentials
  - Improved error messages for missing or invalid required variables
  - Added support for `None` and empty string checks in resource IDs

- **Updated Documentation**:
  - Comprehensive README updates with 12 example playbooks
  - Added Advanced Features section with detailed explanations
  - Included example response structures for all endpoints
  - Updated Table of Contents with new sections

- **Updated Test Playbook**:
  - Extended `tests/test.yml` to cover all 7 endpoints
  - Added query parameter testing
  - Enhanced test summary output

### Fixed
- Fixed URL construction to properly handle empty and None resource IDs
- Improved query parameter handling and URL encoding
- Better handling of authentication methods (token vs basic auth)

### Technical Improvements
- **Code Quality**:
  - Reduced code duplication by 80%
  - Improved error messages with actionable information
  - Better separation of concerns

- **Performance**:
  - Retry logic with configurable delays
  - Timeout settings for API calls

- **Maintainability**:
  - Centralized API call logic
  - Easier to add new endpoints
  - Consistent error handling patterns

## [1.0.0] - Initial Release

### Added
- Initial release with support for 4 endpoints:
  - Inventory endpoint
  - Task endpoint
  - Variable groups endpoint
  - Repository endpoint
- Basic authentication support (token and basic auth)
- SSL certificate validation options
- HTTP request configuration (timeout, redirects, etc.)
- Example playbooks and test cases
- Comprehensive README documentation
