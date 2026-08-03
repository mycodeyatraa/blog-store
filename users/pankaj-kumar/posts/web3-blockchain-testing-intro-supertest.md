---
title: Web3 & Blockchain Testing Intro: Testing Decentralized Apps with Supertest
date: 05-Mar-2026
lastUpdated: 05-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, web3, blockchain, ethereum, dapps]
category: API Supertest
categories: [API Supertest, Web3, Blockchain]
excerpt: >-
  Bridge Web2 and Web3 testing! Learn how to validate JSON-RPC Ethereum endpoints and smart contract gateway APIs using Supertest.
readTime: 8 min read
---

# Web3 & Blockchain Testing Intro: Testing Decentralized Apps with Supertest

Decentralized Applications (**dApps**) rely on Web2 gateway APIs to index blockchain state, handle off-chain metadata, and route transactions to Ethereum / EVM nodes via **JSON-RPC**.

**Supertest** is ideal for testing Web3 node APIs and smart contract indexers by executing JSON-RPC payload calls programmatically.

---

## 1. JSON-RPC API Test Example

**tests/web3-jsonrpc.spec.ts**

```typescript
import request from 'supertest';
import app from '../src/app';
 
describe('Web3 JSON-RPC Node Endpoint Testing', () => {
 
  test('should return current block number from Ethereum node RPC', async () => {
    const jsonRpcPayload = {
      jsonrpc: '2.0',
      method: 'eth_blockNumber',
      params: [],
      id: 1
    };
 
    const response = await request(app)
      .post('/api/v1/rpc')
      .send(jsonRpcPayload);
 
    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('result');
    expect(response.body.result).toMatch(/^0x[0-9a-fA-F]+$/); // Valid hex string
  });
});
```
