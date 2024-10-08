## Entra ID

### Step 1: Register Your Application in Azure AD
1. **Go to the Azure Portal** and navigate to "EntraID".
2. **Select "App registrations"** and click "New registration".
3. **Fill in the registration form**:
   - Name: Your application name.
   - Supported account types: Choose the appropriate one (e.g., "Accounts in this organizational directory only").
   - Redirect URI: Set it to `http://localhost:8080/login/oauth2/code/azure` for local development.
4. **Click "Register"**.

### Step 2: Configure Authentication
1. **After registration**, go to the "Authentication" section of your app.
2. **Add a redirect URI** if not already added: `http://localhost:8080/login/oauth2/code/azure`.
3. **Enable ID tokens** and save the changes.

### Step 3: Add Client Secret
1. **Go to the "Certificates & secrets"** section.
2. **Click "New client secret"**, add a description, and set an expiration period.
3. **Save the secret value**; you'll need it for your Spring Boot application.

### Step 4: Update Application Properties
Add the necessary Azure AD properties to your `application.yml` or `application.properties` file:

#### `application.yml`
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          azure:
            client-id: YOUR_CLIENT_ID
            client-secret: YOUR_CLIENT_SECRET
            scope: openid, profile, email
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
        provider:
          azure:
            authorization-uri: https://login.microsoftonline.com/YOUR_TENANT_ID/oauth2/v2.0/authorize
            token-uri: https://login.microsoftonline.com/YOUR_TENANT_ID/oauth2/v2.0/token
            user-info-uri: https://graph.microsoft.com/oidc/userinfo
            jwk-set-uri: https://login.microsoftonline.com/YOUR_TENANT_ID/discovery/v2.0/keys
            user-name-attribute: preferred_username
```

Replace `YOUR_CLIENT_ID`, `YOUR_CLIENT_SECRET`, and `YOUR_TENANT_ID` with the actual values from your Azure AD app registration.

#### `application.properties`
```properties
spring.security.oauth2.client.registration.azure.client-id=YOUR_CLIENT_ID
spring.security.oauth2.client.registration.azure.client-secret=YOUR_CLIENT_SECRET
spring.security.oauth2.client.registration.azure.scope=openid, profile, email
spring.security.oauth2.client.registration.azure.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.azure.redirect-uri={baseUrl}/login/oauth2/code/{registrationId}

spring.security.oauth2.client.provider.azure.authorization-uri=https://login.microsoftonline.com/YOUR_TENANT_ID/oauth2/v2.0/authorize
spring.security.oauth2.client.provider.azure.token-uri=https://login.microsoftonline.com/YOUR_TENANT_ID/oauth2/v2.0/token
spring.security.oauth2.client.provider.azure.user-info-uri=https://graph.microsoft.com/oidc/userinfo
spring.security.oauth2.client.provider.azure.jwk-set-uri=https://login.microsoftonline.com/YOUR_TENANT_ID/discovery/v2.0/keys
spring.security.oauth2.client.provider.azure.user-name-attribute=preferred_username
```

### Step 5: Configure Security in Your Spring Boot Application
Create a security configuration class to set up OAuth2 login:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorizeRequests ->
                authorizeRequests
                    .requestMatchers("/", "/login**").permitAll()
                    .anyRequest().authenticated()
            )
            .oauth2Login(withDefaults());
        return http.build();
    }
}
```

```gradle
implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
```



This should set up Azure AD authentication for your Spring Boot application.
