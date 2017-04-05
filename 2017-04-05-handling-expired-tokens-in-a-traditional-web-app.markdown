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

# Example

