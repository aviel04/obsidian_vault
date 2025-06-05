# What is SSO and AUTH
> SSO Enables authentication using one set of credentials for multiple services using IdP connecting various - **service Providers** (SPs) 
- auth
- IDP: identity provider
- Credentials: digital docs associated with user like smart card
- OIDC : openID connect
- user/client: 
	- user have many attributes (usually a person) that can login,
	- client has 3 main attributes `id,secret,uri


# How do we use SSO and AUTH in GITLAB
#### Components
- omniauth (section) main config for IDP for identity provisioning, it can use SAML 
	- OmniAuth is a library that standardizes multi-provider authentication for web applications. It was created to be powerful, flexible, and do as little as possible. Any developer can create **strategies** for OmniAuth that can authenticate users via disparate systems.
- JIT: just in time account provisioning, user can sign up without having an account created by admin first

#### Gitlab Role
- **gitlab as a service provider** - the server acts as an `SP` while relying on an external `IdP` to get **identities**
**When a user tries to log into GitLab:**
1. GitLab redirects the user to the configured IdP.
2. The user authenticates with the IdP (if they haven't already in their current browser session).
3. The IdP confirms the user's identity and sends an authentication Assertion (a secure token) back to GitLab.
4. GitLab verifies this assertion-Token and grants the user access

#### Protocols for implementing SSO:
**SAML (Security Assertion Markup Language)**:
- **Description**: Widely adopted XML standard exchanges authentication data between an IdP and an SP. 
- **How it works with GitLab**: You configure GitLab with details about your SAML IdP (e.g., IdP's SSO URL, certificate). When a user attempts to log in via SAML, GitLab sends a SAML request to the IdP, the user authenticates, and the IdP sends a SAML response back to GitLab containing user information.

**OAuth 2.0 / OpenID Connect (OIDC)**:
- **Description**: Authorization framework, while OIDC is a thin identity layer built on top of OAuth 2.0. Together, they allow third-party applications (like GitLab) to authenticate users via an IdP.
- **How it works with GitLab**: You can configure GitLab to allow users to sign in using their accounts from providers like Google, or other OAuth2 providers. The user is redirected to the OAuth provider to authorize GitLab, and then redirected back with an access token/ID token.

**LDAP (Lightweight Directory Access Protocol) / Active Directory (AD)**:
# Openid/oAuth2


---
# CA in HTTPS (cannot recognize issuer cert)
## update CA gitlab
## gitlab bug certificate chains with multiple certificates
## Gitlab openssl
# Log-Rotate what is it and why use it in GITLAB
## save 20 rotating logs
## over 500m rotate

---

To integrate OpenID Connect (OIDC) with your GitLab instance on a Linux Omnibus setting, you will configure the `gitlab.rb` file. This file is located at `/etc/gitlab/gitlab.rb`.

Here is a comprehensive code sample for the `gitlab.rb` configuration, directly supporting OpenID Connect integration:
	
[`/etc/gitlab/gitlab.rb`](https://docs.gitlab.com/administration/auth/oidc/)
```ruby
gitlab_rails['omniauth_providers'] = [
  {
    name: "openid_connect", # do not change 
    label: "Provider name",  # optional label defaults to "Openid Connect"
    icon: "<custom_provider_icon>", # Optional custom icon for the login page
    args: {
      name: "openid_connect",
      scope: ["openid", "profile", "email"], # Defines the level of access requested
      response_type: "code", # Specifies: application is requesting an authorization code grant
      issuer: "<your_oidc_url>", 
      discovery: true, 
      client_auth_method: "query", 
      uid_field: "<uid_field>", 
      send_scope_to_token_endpoint: "false", 
      pkce: true, # Enable Proof Key for Code Exchange (available in GitLab 15.9).
      client_options: {
        identifier: "<your_oidc_client_id>", 
        secret: "<your_oidc_client_secret>", 
        redirect_uri: "<your_gitlab_url>/users/auth/openid_connect/callback" 
      }
    }
  }
]
```

### Key Configuration Details

- **`name: "openid_connect"`**: This parameter should **not be changed**.
- **`label`**: This is an optional label that appears on the GitLab login page for the OpenID Connect option.
- **`icon`**: An optional parameter for a custom icon on the login page. GitLab supports local paths or absolute URLs for this icon.
- **`scope`**: Defines the level of access requested from the OIDC provider. Common values include `"openid"`, `"profile"`, and `"email"`. The claims `email` and `email_verified` are included only if the application has access to the `email` scope and the user's public email address.
- **`response_type: "code"`**: This specifies that your application is requesting an authorization code grant. The Authorization Code grant type is commonly used for server-side applications where the client secret's confidentiality can be maintained, and it is a redirection-based flow.
- **`issuer`**: This is the URL that points to your OpenID Connect provider. For example, for Google, it would be `https://accounts.google.com`. If `discovery` is enabled, this URL's `.well-known/openid-configuration` endpoint will be used for auto-discovery of client options.
- **`discovery`**: When set to `true`, the OpenID Connect provider attempts to automatically discover client options using the issuer URL's `.well-known/openid-configuration` endpoint. If `discovery` is `false`, you must manually specify `authorization_endpoint`, `token_endpoint`, `userinfo_endpoint`, and `jwks_uri` within `client_options`.
- **`client_auth_method`**: This specifies how the client authenticates with the OIDC provider. Options include `basic` (HTTP Basic Authentication), `jwt_bearer` (JWT-based authentication), `mtls` (Mutual TLS), or other values that post the client ID and secret in the request body. If not specified, it defaults to `basic`. If you encounter 401 errors when retrieving the `userinfo` endpoint, check this setting and your OpenID web server configuration.
- **`uid_field`**: An optional field name from `user_info.raw_attributes` that defines the `uid` value. If not provided or the field is missing, the `sub` field is used.
- **`pkce`**: Proof Key for Code Exchange (PKCE) can be enabled, available in GitLab 15.9. The OAuth 2.1 draft recommends PKCE for all OAuth clients to prevent code injection attacks.
- **`client_options`**:
    - **`identifier`**: This is the **client ID** provided by your OpenID Connect service provider after you register your application.
    - **`secret`**: This is the **client secret** provided by your OpenID Connect service provider. It must be kept private.
    - **`redirect_uri`**: This is the **GitLab URL** to which the OIDC provider will redirect the user after a successful login. This URL typically follows the format `https://<your_gitlab_url>/users/auth/openid_connect/callback`.



After making these changes to `/etc/gitlab/gitlab.rb`, you must **save the file and reconfigure GitLab** for the changes to take effect by running `sudo gitlab-ctl reconfigure`. Once reconfigured, an "OpenID Connect" option will appear on the GitLab sign-in page, allowing users to authenticate through your OIDC provider.

For advanced configurations, such as supporting multiple OIDC providers, including one with 2FA, you can add additional provider blocks to `omniauth_providers` and explicitly set the `strategy_class`. You can also configure GitLab to manage user roles and group memberships based on OIDC group information passed from your Identity Provider. This involves specifying `groups_attribute`, and then `required_groups`, `external_groups`, `auditor_groups`, or `admin_groups` within a `gitlab` sub-block under `client_options`.

If you encounter authentication issues, common troubleshooting steps include ensuring `discovery` is set to `true` (unless you manually provide all endpoints), verifying your system clock synchronization, and checking that the `issuer` URL corresponds to the base URL of the Discovery URL.


---
> https://docs.gitlab.com/integration/oauth2_generic/
# Generic Connector OAuth

