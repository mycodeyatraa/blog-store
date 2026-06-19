---
title: Schema-Less Assertions: MongoDB Validation
date: 20-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, mongodb, mongoose, nosql, database-testing, enterprise-validation]
category: Selenium TypeScript
categories: [Selenium TypeScript, Enterprise Validation]
excerpt: >-
  Leave the relational world behind. Learn how to connect to MongoDB using Mongoose, and validate highly nested, schema-less BSON documents directly from your TypeScript automation framework.
readTime: 4 min read
---

# Schema-Less Assertions: MongoDB Validation

We have conquered relational databases (MySQL) and hybrid databases (PostgreSQL). Now, it is time to venture into the world of **NoSQL**.

Unlike SQL databases, which strictly enforce schemas (columns and types), NoSQL document databases like **MongoDB** are completely schema-less. Data is stored as BSON (Binary JSON) documents. This provides incredible flexibility for developers, but it presents a unique challenge for Automation Engineers.

How do you validate data when the structure of that data is constantly changing?

Today, we will connect our Selenium framework to MongoDB and write assertions against schema-less documents.

---

## 1. Installing Mongoose

To interact with MongoDB in a Node.js/TypeScript environment, the industry standard is **Mongoose**. Mongoose provides a straight-forward, schema-based solution to model your application data, even in a NoSQL environment.

Install the necessary packages:

```bash
npm install mongoose
npm install --save-dev @types/mongoose
```

---

## 2. Establishing the MongoDB Connection

As always, we handle our connections securely via environment variables.

In your `.env` file:

```env
MONGO_URI=mongodb://qa_user:qa_pass@localhost:27017/enterprise_qa_db
```

Create `MongoManager.ts`:

```typescript
import mongoose from 'mongoose';
import * as dotenv from 'dotenv';
dotenv.config();
export class MongoManager {
  private static isConnected = false;
  static async connect() {
    if (!this.isConnected) {
      const uri = process.env.MONGO_URI;
      if (!uri) throw new Error('MONGO_URI is not defined in .env');
      await mongoose.connect(uri);
      this.isConnected = true;
      console.log('[DB] Connected to MongoDB successfully.');
    }
  }
  static async disconnect() {
    if (this.isConnected) {
      await mongoose.disconnect();
      this.isConnected = false;
      console.log('[DB] MongoDB connection closed.');
    }
  }
}
```

---

## 3. Defining the Mongoose Model and DAO

Even though MongoDB is schema-less, Mongoose allows us to define an interface and a schema to provide strict TypeScript typing in our test code!

Let's imagine we are testing a User Profile application.

Create `UserDocument.ts` (The Mongoose Schema):

```typescript
import mongoose, { Document, Schema } from 'mongoose';
// 1. Define the TypeScript Interface
export interface IUser extends Document {
  email: string;
  preferences: {
    theme: string;
    notificationsEnabled: boolean;
  };
  tags?: string[]; // Optional, schema-less array!
}
// 2. Define the Mongoose Schema
const UserSchema: Schema = new Schema({
  email: { type: String, required: true, unique: true },
  preferences: {
    theme: { type: String, default: 'light' },
    notificationsEnabled: { type: Boolean, default: true }
  },
  tags: [String]
});
// 3. Export the Model
export const UserModel = mongoose.model<IUser>('User', UserSchema);
```

Now, create the DAO `MongoUserDAO.ts`:

```typescript
import { MongoManager } from '../utils/MongoManager';
import { UserModel, IUser } from './UserDocument';
export class MongoUserDAO {
  static async getUserByEmail(email: string): Promise<IUser | null> {
    await MongoManager.connect();
    // Mongoose makes querying incredibly easy!
    return await UserModel.findOne({ email }).exec();
  }
  static async deleteUserByEmail(email: string) {
    await MongoManager.connect();
    await UserModel.deleteOne({ email }).exec();
  }
}
```

---

## 4. The NoSQL Assertion Test

Let's write a test that updates a user's preferences via the UI and validates that the nested JSON document in MongoDB reflects the change.

```typescript
import { Builder, WebDriver } from 'selenium-webdriver';
import { expect } from 'chai';
import { ProfileSettingsPage } from '../pages/ProfileSettingsPage';
import { MongoUserDAO } from '../db/MongoUserDAO';
import { MongoManager } from '../utils/MongoManager';
describe('MongoDB Profile Update Validation', function () {
  let driver: WebDriver;
  const testEmail = 'nosql_user@mycodeyatra.com';
  before(async function () {
    driver = await new Builder().forBrowser('chrome').build();
  });
  after(async function () {
    await driver.quit();
    // Teardown: Remove the test user document
    await MongoUserDAO.deleteUserByEmail(testEmail);
    await MongoManager.disconnect();
  });
  it('should verify nested NoSQL document updates', async function () {
    const profilePage = new ProfileSettingsPage(driver);
    await profilePage.navigate();
    // 1. UI Layer Action: Toggle Dark Mode and Disable Notifications
    await profilePage.login(testEmail, 'password123');
    await profilePage.enableDarkMode();
    await profilePage.disableNotifications();
    await profilePage.saveSettings();
    // 2. UI Layer Assertion
    const toast = await profilePage.getToastMessage();
    expect(toast).to.equal('Settings Saved!');
    // 3. Database Layer Assertion
    const mongoDoc = await MongoUserDAO.getUserByEmail(testEmail);
    expect(mongoDoc).to.not.be.null;
    // Validate the deeply nested BSON structure
    expect(mongoDoc!.preferences.theme).to.equal('dark');
    expect(mongoDoc!.preferences.notificationsEnabled).to.be.false;
  });
});
```

## Conclusion

Testing NoSQL databases requires a different mindset. By leveraging Mongoose, we can bring TypeScript's strict typing to MongoDB's schema-less nature, ensuring that our Automation Framework remains highly reliable.

We have now conquered databases. But what if the data isn't in a database? What if it's currently flying through the air in an asynchronous event queue?

In the next tutorial, we will learn how to connect our Selenium tests directly to **Kafka** to validate Enterprise microservice event streams!
