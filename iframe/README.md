# iFrame

This module shows how to use the iframe to export a key using the provided components.

## Configuring Location Block Routing

The template (`iframe_nginx.conf.template`) includes a basic location block that serves static files. To enable production-ready routing with API key extraction, origin validation, and platform-based CSP policies, follow these steps:

### Overview

The advanced routing system validates requests, extracts API keys from URLs, detects platform type (web vs native), and applies appropriate security policies.

### Step-by-Step Configuration

1. **Replace the simple `location /` block** (lines 90-120) with a regex-based location that matches your API key pattern:
   ```nginx
   location ~ "^/iframe/(pk_(?:test|live)_[a-zA-Z0-9\-]+)(/.*)?$" {
       set $api_key $1;  # Extract API key from URL
   ```

2. **Enable authentication** by uncommenting and configuring:
   ```nginx
   auth_request /auth-origin;
   auth_request_set $platform_type $upstream_http_x_platform_type;
   auth_request_set $allowed_origins $upstream_http_x_allowed_origins;
   ```

3. **Update `/auth-origin` location** (lines 51-70) to:
   - Uncomment `proxy_set_header Authorization "Bearer $api_key";`
   - Add `proxy_ssl_server_name on;` and `proxy_ssl_name` if using HTTPS
   - Add `X-Forwarded-For` and `X-Native-App-Identifier` headers if needed

4. **Add platform-based routing** after the main location block:
   ```nginx
   location @check_platform {
       error_page 418 = @serve_web_with_csp;
       error_page 419 = @serve_native_no_csp;
       if ($platform_type = "native") {
           return 419;
       }
       return 418;
   }
   ```

5. **Create platform-specific serving locations**:
   - `@serve_web_with_csp`: Full CSP with `frame-ancestors $allowed_origins`
   - `@serve_native_no_csp`: No CSP header (native apps don't need it)

6. **Update CSP policy** to include:
   - `script-src 'unsafe-eval'` if needed for your application
   - Sentry DSN or other monitoring endpoints in `connect-src`
   - Use `$request_id` instead of `$request_nonce` for nonce values

7. **Change file serving** from `try_files $uri $uri/ /index.html` to `try_files /index.html` to always serve the SPA.

### Key Differences

- **URL Pattern**: Regex captures API key from `/iframe/pk_*` URLs
- **Authentication**: Active origin validation via backend
- **Platform Detection**: Routes to different handlers based on platform type
- **CSP Policy**: Platform-specific (full CSP for web, none for native)

See the production configuration for a complete reference implementation.
