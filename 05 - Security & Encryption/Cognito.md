---
tags:
  - aws/cognito
  - security
  - identity
category: Security, Identity & Compliance
---

# 👤 Amazon Cognito

> [!INFO] Definition
> **Amazon Cognito** provides authentication, authorization, and user management for web and mobile apps. It acts as an identity broker between your application and various identity providers.

---

## 🏗️ The Two Pillars of Cognito

Cognito is split into two distinct components. Understanding the difference between them is critical for the exam.

### 1. User Pools (Authentication)
User Pools are all about **Identity and Sign-In**. Think of it as a managed user directory.

* **Core Function**: Users sign up, sign in, and receive a **JSON Web Token (JWT)** upon successful authentication.
* **Features**:
  * Customizable web UI for sign-up and sign-in.
  * User profile management.
  * Built-in security features like **MFA** (Multi-Factor Authentication) and password policies.
* **Limitations**: User Pools **DO NOT** grant direct access to AWS resources (like S3 or DynamoDB). Their sole job is to authenticate the user and provide the JWT.

### 2. Identity Pools (Authorization/Access)
Identity Pools (Federated Identities) are all about **Swapping Identities for AWS Credentials**.

* **Core Function**: Exchanges an external identity token for a set of **temporary AWS credentials** allowing direct access to AWS resources.
* **How it Works**: The Identity Pool assumes an **IAM Role** on behalf of the identity, and that assumption generates the temporary credentials.
* **Supported Identities**:
  * **Authenticated Identities (Federated)**: Swaps tokens from Google, Facebook, Twitter, Amazon, SAML 2.0, *OR a Cognito User Pool JWT*.
  * **Unauthenticated Identities**: Provides limited temporary credentials for **Guest Users**.

---

## 🤝 How They work Together

User Pools and Identity Pools are often used together to provide a seamless architecture:

1. A user authenticates against the **Cognito User Pool** (or a 3rd party like Google).
2. The User Pool (or 3rd party) provides a **JWT**.
3. The application passes this JWT to the **Cognito Identity Pool**.
4. The Identity Pool validates the token, assumes an IAM Role, and returns **Temporary AWS Credentials**.
5. The application uses those credentials to directly access AWS services (e.g., uploading a photo to S3).

> [!IMPORTANT] Exam PowerUP: User Pools vs. Identity Pools
> * If the question needs **Sign-up/Sign-in**, **JWTs**, or **User Profiles** -> **User Pools**.
> * If the question needs **AWS Credentials** to access services (S3, API Gateway) -> **Identity Pools**.
> * If the question needs **Guest Access** (unauthenticated users) -> **Identity Pools**.
