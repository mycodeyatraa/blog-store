---
title: Smart Contract Validation: Ethers.js & Hardhat with Supertest
date: 06-Mar-2026
lastUpdated: 06-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, smart-contracts, ethersjs, hardhat, solidity]
category: API Supertest
categories: [API Supertest, Web3, Blockchain]
excerpt: >-
  Validate on-chain smart contract state! Combine Ethers.js and Hardhat with Supertest to test Web3 backend integrations.
readTime: 8 min read
---

# Smart Contract Validation: Ethers.js & Hardhat with Supertest

Testing Web3 applications requires asserting that off-chain Express backend services correctly react to on-chain Solidity smart contract events and state updates.

---

## 1. Testing Express API Against Hardhat Network

```typescript
import request from 'supertest';
import { ethers } from 'ethers';
import app from '../src/app';
 
describe('Smart Contract State Validation Test', () => {
 
  test('should query ERC-20 token balance via backend API', async () => {
    const mockWallet = '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266';
 
    const response = await request(app)
      .get(`/api/v1/tokens/balance/${mockWallet}`);
 
    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('balance');
  });
});
```
