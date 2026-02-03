# Changelog

All notable changes to the semaphore-api-get role will be documented in this file.

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
