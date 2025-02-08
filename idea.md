# High-Level Design (HLD)

## Overview

This document provides the High-Level Design (HLD) and Low-Level Design (LLD) for the Salesforce Testing Web App SaaS. The application enables users to create and execute automated tests for Salesforce using BDD (Behavior-Driven Development) principles. It integrates with Salesforce Metadata API for fetching org structure and WebDriverIO for test execution.

## Key Features

1. **User Authentication**
   - Free 7-day trial
   - Per-user signup
   - Organization signup (100+ users get full access)
2. **Salesforce Metadata Fetching**
   - Uses Salesforce Metadata API to retrieve org details
3. **BDD Test Editor**
   - Uses Cucumber BDD framework
   - CodeMirror-based UI for editing test cases
4. **Test Execution Engine**
   - WebDriverIO for running UI automation tests
   - Executes within browser using WebContainer
5. **Project Management**
   - Users can save test cases as projects

## Architecture

- **Frontend:** React, CodeMirror (for BDD Editor)
- **Backend:** Node.js, Express, jsforce (Salesforce API)
- **Database:** PostgreSQL (for storing user details, test cases)
- **Test Execution:** Cucumber.js + WebDriverIO

---

# Low-Level Design (LLD)

## 1. User Authentication

- **Signup/Login APIs**
- **Token-based authentication (JWT)**
- **Roles:** Trial User, Paid User, Organization Admin

## 2. Salesforce Metadata Fetching

- **API Call:** Uses `jsforce` to fetch metadata
- **Stored in DB for caching**

## 3. BDD Test Editor

- **React component with CodeMirror**
- **Supports writing/editing feature files**

## 4. Test Execution Engine

- **Parses feature files using Cucumber.js**
- **Runs WebDriverIO automation inside a sandbox**
- **Reports test results**

## 5. Project Management

- **CRUD APIs for test cases**
- **Database Schema:** Users, Projects, Test Cases

---

# Sample Implementation

## 1. Salesforce Metadata API Fetching (Node.js with jsforce)

```javascript
const jsforce = require("jsforce");

async function fetchSalesforceMetadata(accessToken, instanceUrl) {
  const conn = new jsforce.Connection({ accessToken, instanceUrl });
  try {
    const metadata = await conn.metadata.read("CustomObject", "Lead");
    console.log("Fetched Metadata:", metadata);
    return metadata;
  } catch (error) {
    console.error("Error fetching metadata:", error);
    throw error;
  }
}
module.exports = { fetchSalesforceMetadata };
```

## 2. Test Execution Engine (Cucumber.js + WebDriverIO)

```javascript
const { Given, When, Then } = require("@cucumber/cucumber");
const { remote } = require("webdriverio");

let browser;

Given("I open Salesforce login page", async () => {
  browser = await remote({ capabilities: { browserName: "chrome" } });
  await browser.url("https://login.salesforce.com");
});

When("I enter username and password", async () => {
  await browser.$("#username").setValue("testuser");
  await browser.$("#password").setValue("password123");
});

Then("I should see the Salesforce dashboard", async () => {
  await browser.$("#Login").click();
  await browser.pause(5000);
  const title = await browser.getTitle();
  console.log("Page Title:", title);
  await browser.deleteSession();
});
```

## 3. Frontend UI (React + CodeMirror for BDD Editor)

```javascript
import React, { useState } from "react";
import CodeMirror from "@uiw/react-codemirror";
import { javascript } from "@codemirror/lang-javascript";

const BDDTestEditor = () => {
  const [code, setCode] = useState(
    "Feature: Salesforce Login\n  Scenario: User logs into Salesforce\n    Given I open Salesforce login page\n    When I enter username and password\n    Then I should see the Salesforce dashboard"
  );

  return (
    <div>
      <h2>BDD Test Editor</h2>
      <CodeMirror
        value={code}
        height="400px"
        extensions={[javascript()]}
        onChange={(value) => setCode(value)}
      />
    </div>
  );
};

export default BDDTestEditor;
```
