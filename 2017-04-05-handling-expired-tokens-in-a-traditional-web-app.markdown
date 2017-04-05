---
layout: post
title: "Handling Expired Tokens in a Traditional Web App"
description: <A SHORT DESCRIPTION OF THE POST>
date: 2017-04-05 10:51
category: <FROM HERE: https://docs.google.com/spreadsheets/d/1e_RKzi8kVwzqPG8si8kyDOWPiBk9tI-XNGh0KgRIF7Q>
author: 
  name: <YOUR NAME>
  url: <YOUR URL>
  mail: <YOUR MAIL>
design: 
  bg_color: <A HEX BACKGROUND COLOR>
  image: <A PATH TO A 200x200 IMAGE>
tags: 
- foo
related:
- <ADD SOME RELATED POSTS FROM AUTH0'S BLOG>
---

# TL;DR
This is an awesome solution, deal with it!

# Scenario
Many traditional WebApps need to communicate with a set of resource servers.  Microservice approaches have been shown to be quite valuable, but can quickly lead to a complex set of authorization headaches.  We want a simple solution to avoid the headaches, but still leave us with a secure set of applications.

In this scenario, we are going to build a WebApp called c0der.io.  This application is a social site for developers.  It allows engineers to post information about themselves and the projects they are working on as well as escalating different levels based on their involvement and their peers votes.  To accomplish this c0der.io has the following components:

*  A website: this site has a homepage that has a way to send the user to their profile and lists their achievements and projects for review.  This homepage is a traditional webapp, server side rendering, but also has the ability to refresh the projects and achievements using AJAX calls to the server.
*  A users resource server: this resource server is an API that allows retrieval of user information, and patching of user profile information.
*  An achievements resource server: this resource server is a RESTful API that allows retrieval of achievement information and adding new achievements.
*  A projects resource server: this resource server is a RESTful API that allows retrieval of current projects, updating those projects, and adding new projects.

![Application Overview](./images/application_overview.png)

# Our Choices and Points of Interest

## Handling Expired Tokens

### Refresh tokens
This approach involves storing a refresh token on the server side.  When a token is expired, the refresh token is used to obtain a new set of tokens from the `/oath/token` endpoint.  Due to the nature of refresh tokens (they don't expire) these tokens must be protected on the server side and revoked when no longer needed.

<table>
  <tbody>
    <tr>
      <th align="center">Benefits</th>
      <th align="center">Trade-offs</th>
    </tr>
    <tr>
      <td>
        <ul>
          <li>No Client Side Code Needed</li>
          <li>Single Call to Refresh</li>
        </ul>
      </td>
      <td>
        <ul>
          <li>Refresh Tokens Do Not Expire</li>
          <li>Should Revoke them on Session Expiration</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

### SSO Cookie
This approach involves redirecting to the hosted login page when the token is expired.  If an SSO cookie is present, the redirect will automatically resolve and send the request to the callback, which can then redirect back to the originally requested page.

<table>
  <tbody>
    <tr>
      <th align="center">Benefits</th>
      <th align="center">Trade-offs</th>
    </tr>
    <tr>
      <td>
        <ul>
          <li>No Client Side Code Needed</li>
          <li>Reuses login code; only an addition if-check needed</li>
        </ul>
      </td>
      <td>
        <ul>
          <li>No Refresh Tokens to Maintain</li>
          <li>Refresh involves redirects, so complexity is slightly higher and potentially slower (though it re-uses existing code, so no additional flows are required).</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>


### Renew Auth
This technique involves pushing the expiry time out to the client.  That client can then set a timer or check on each request to the backend and initiate a background renewAuth request to refresh the tokens on the server's session.  This is similar to the SSO approach, but is initiated on the client side.  This is generally used in SPA and Hybrid SPA/WebApp applications.

<table>
  <tbody>
    <tr>
      <th align="center">Benefits</th>
      <th align="center">Trade-offs</th>
    </tr>
    <tr>
      <td>
        <ul>
          <li>Works well if making an Ajax call to the backend instead of refreshing the page.</li>
          <li>No Refresh tokens needed</li>
        </ul>
      </td>
      <td>
        <ul>
          <li>Need to create a separate callback from your login callback</li>
          <li>Needs client side code to trigger</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

## Handling Access Tokens for the Resource Servers

### Single Audience
<table>
  <tbody>
    <tr>
      <th align="center">Benefits</th>
      <th align="center">Trade-offs</th>
    </tr>
    <tr>
      <td>
        <ul>
          <li>???</li>
        </ul>
      </td>
      <td>
        <ul>
          <li>???</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

### Each Resource Server has its own Audience
<table>
  <tbody>
    <tr>
      <th align="center">Benefits</th>
      <th align="center">Trade-offs</th>
    </tr>
    <tr>
      <td>
        <ul>
          <li>???</li>
        </ul>
      </td>
      <td>
        <ul>
          <li>???</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

# Recommendation

## Choices Made

### Handling Expired Tokens: SSO Cookie
We chose SSO Cookie as the approach for this post because it is the simplest to implement and for a traditional web app that does not do any AJAX calls from the client side, this approach will satisfy all scenarios.  This will keep the authorization flow simple.

### Handling Access Tokens for Resource Servers: Single Audience
Since all of our API's are built in house, we consider them one audience.  We will use scopes to restrict access to the individual services as needed.

## Login Flow
![Login Flow](./images/login_flow.png)

1.  User enters https://c0der.io/dashboard
2.  Browser requests the page https://c0der.io/dashboard from the c0der.io server
3.  If there is a session and the access_token has not expired, skip to step 11; otherwise => c0der.io backend service redirects to https://c0der.auth0.com/authorize.  It passes the following information:

  *  client_id: client_id for the c0der.io main site
  *  scope: openid profile read:users write:users read:achievements write:achievements read:projects write:projects
  *  state: base64 encoded JSON string { "prompt": "none", returnTo": "/dashboard", "nonce": "a unique identifier stored in the session for checking later" }
  *  redirect_uri: https://c0der.io/auth/callback
  *  audience: https://c0der.io/api/v1/
  *  response_type: code
  *  prompt=none
  
4.  If there is an SSO cookie, skip to step 9; otherwise => c0der.auth0.com redirects to https://c0der.io/auth/callback with error=login_required
5.  c0der.io backend service redirects to https://c0der.auth0.com/authorize

  *  client_id: client_id for the c0der.io main site
  *  scope: openid profile read:users write:users read:achievements write:achievements read:projects write:projects
  *  state: base64 encoded JSON string { "returnTo": "/dashboard", "nonce": "a unique identifier stored in the session for checking later" }
  *  redirect_uri: https://c0der.io/auth/callback
  *  audience: https://c0der.io/api/v1/
  *  response_type: code
  
6.  auth0 presents the hosted login page to the browser
5.  Browser presents hosted login page to the user
6.  User enters their credentials and clicks "login"
7.  Browser redirects to auth0 to validate the credentials.  Rules are executed.  The check resource authorization rule will check the user's permissions against the requested scope and filter them out and just returns the scopes that the user has permission to.
8.  If rules all pass, Auth0 redirects to https://c0der.io/auth/callback#code=...&state=<original state>
9.  The /auth/callback validates the state.nonce and calls https://c0der.auth0.com/oath/token, it passes the client ID and secret for the c0der.io main site, grant_type=authorization_code, code=the code passed in through the hash, redirect_uri=https://c0der.io/auth/callback.  This will return the access_token, id_token, and expires_in values.  Those will be stored in local session and then the callback will redirect to /dashboard
10.  /dashboard will call GET https://c0der.io/api/v1/users, passing Authorization="Bearer <access_token>"
11.  /dashboard will call GET https://c0der.io/api/v1/achievements, passing Authorization="Bearer <access_token>"
12.  /dashboard will call GET https://c0der.io/api/v1/projects, passing Authorization="Bearer <access_token>"
13.  The /dashboard view will be rendered with that information and passed to the browser.
14.  The Browser presents the HTML to the user

# Example

