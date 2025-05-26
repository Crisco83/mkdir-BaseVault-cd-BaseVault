#!/bin/bash

set -e

echo "Scaffolding BaseVault project..."

# Root files
cat > .gitignore <<EOF
node_modules/
.env
.env.*
dist/
build/
artifacts/
cache/
coverage/
*.log
npm-debug.log*
yarn-debug.log*

# Hardhat
contracts/artifacts/
contracts/cache/
contracts/typechain/
contracts/coverage/
contracts/test-coverage/

# Frontend
frontend/build/
frontend/node_modules/
frontend/.env

# Backend
backend/node_modules/
backend/dist/
backend/.env
EOF

cat > LICENSE <<EOF
MIT License

Copyright (c) 2025 Crisco83

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF

cat > README.md <<EOF
# BaseVault

BaseVault is a decentralized staking platform built on the Base Coin Network. Users stake Base Ethereum (ETH) and earn rewards paid out in a special VaultToken.

## Features

- Stake Base ETH and earn yield
- Rewards distributed in VaultToken
- Transparent, secure smart contracts
- Modern React frontend (Vite + TypeScript)
- Backend scaffold for future API/services

## Project Structure

- \`contracts/\` – Solidity smart contracts (Hardhat)
- \`frontend/\` – Vite + React frontend
- \`backend/\` – Express-ready backend scaffold

## Setup

1. Clone the repo

2. Install dependencies for each part:

### Contracts (Hardhat + Solidity)

\`\`\`
cd contracts
npm install
npx hardhat test
\`\`\`

### Frontend (Vite + React + TypeScript)

\`\`\`
cd frontend
npm install
npm run dev
\`\`\`

### Backend (Express Scaffold)

\`\`\`
cd backend
npm install
npm run dev
\`\`\`

## License

MIT
EOF

# Contracts (Hardhat)
mkdir -p contracts/contracts
cat > contracts/package.json <<EOF
{
  "name": "basevault-contracts",
  "version": "1.0.0",
  "description": "BaseVault staking smart contracts (Hardhat, Solidity)",
  "scripts": {
    "test": "hardhat test",
    "compile": "hardhat compile"
  },
  "devDependencies": {
    "@nomicfoundation/hardhat-toolbox": "^3.0.0",
    "hardhat": "^3.0.0"
  }
}
EOF

cat > contracts/hardhat.config.js <<EOF
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.20",
  networks: {
    hardhat: {},
    // Add Base or Ethereum config here if needed
  },
};
EOF

cat > contracts/contracts/BaseVault.sol <<EOF
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BaseVault {
    // Basic structure for staking contract

    IERC20 public vaultToken;
    address public owner;

    mapping(address => uint256) public stakes;
    mapping(address => uint256) public rewards;

    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount);
    event RewardPaid(address indexed user, uint256 reward);

    constructor(address _vaultToken) {
        vaultToken = IERC20(_vaultToken);
        owner = msg.sender;
    }

    function stake() external payable {
        require(msg.value > 0, "Stake must be greater than 0");
        stakes[msg.sender] += msg.value;
        // Example logic: In production, add time-based/percentage accrual
        emit Staked(msg.sender, msg.value);
    }

    function unstake(uint256 amount) external {
        require(stakes[msg.sender] >= amount, "Not enough staked");
        stakes[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
        emit Unstaked(msg.sender, amount);
    }

    function claimReward() external {
        uint256 reward = calculateReward(msg.sender);
        require(reward > 0, "No rewards");
        rewards[msg.sender] = 0;
        vaultToken.transfer(msg.sender, reward);
        emit RewardPaid(msg.sender, reward);
    }

    function calculateReward(address staker) public view returns (uint256) {
        // Placeholder logic: real logic would use stake amount, duration, rate
        return stakes[staker] / 100; // 1% reward for demonstration
    }
}

interface IERC20 {
    function transfer(address to, uint256 value) external returns (bool);
}
EOF

# Frontend (Vite + React + TS)
mkdir -p frontend/src
cat > frontend/package.json <<EOF
{
  "name": "basevault-frontend",
  "version": "1.0.0",
  "description": "BaseVault frontend (Vite, React, TypeScript)",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.1.0",
    "typescript": "^5.0.0",
    "vite": "^4.0.0"
  }
}
EOF

cat > frontend/vite.config.ts <<EOF
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
EOF

cat > frontend/tsconfig.json <<EOF
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": false,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
EOF

cat > frontend/index.html <<EOF
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>BaseVault</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
EOF

cat > frontend/src/main.tsx <<EOF
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
EOF

cat > frontend/src/App.tsx <<EOF
import React from 'react';

function App() {
  return (
    <div style={{ maxWidth: 600, margin: '2rem auto', fontFamily: 'sans-serif' }}>
      <h1>BaseVault Staking DApp</h1>
      <p>
        Stake your Base ETH and earn VaultToken rewards!
      </p>
      {/* In production, connect wallet, show stake/unstake/claim UI */}
      <em>Frontend coming soon...</em>
    </div>
  );
}

export default App;
EOF

# Backend (Express)
mkdir -p backend
cat > backend/package.json <<EOF
{
  "name": "basevault-backend",
  "version": "1.0.0",
  "description": "Backend scaffold for BaseVault (Express-ready)",
  "scripts": {
    "dev": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

cat > backend/index.js <<EOF
const express = require('express');
const app = express();
const PORT = process.env.PORT || 4000;

app.get('/', (req, res) => {
  res.send('BaseVault backend API is running!');
});

app.listen(PORT, () => {
  console.log(\`BaseVault backend listening at http://localhost:\${PORT}\`);
});
EOF

echo "✅ BaseVault project scaffolded!"
echo "Next steps:"
echo "1. Run 'npm install' in contracts/, frontend/, and backend/"
echo "2. Start building your staking platform!"#!/bin/bash

set -e

echo "Scaffolding BaseVault project..."

# Root files
cat > .gitignore <<EOF
node_modules/
.env
.env.*
dist/
build/
artifacts/
cache/
coverage/
*.log
npm-debug.log*
yarn-debug.log*

# Hardhat
contracts/artifacts/
contracts/cache/
contracts/typechain/
contracts/coverage/
contracts/test-coverage/

# Frontend
frontend/build/
frontend/node_modules/
frontend/.env

# Backend
backend/node_modules/
backend/dist/
backend/.env
EOF

cat > LICENSE <<EOF
MIT License

Copyright (c) 2025 Crisco83

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF

cat > README.md <<EOF
# BaseVault

BaseVault is a decentralized staking platform built on the Base Coin Network. Users stake Base Ethereum (ETH) and earn rewards paid out in a special VaultToken.

## Features

- Stake Base ETH and earn yield
- Rewards distributed in VaultToken
- Transparent, secure smart contracts
- Modern React frontend (Vite + TypeScript)
- Backend scaffold for future API/services

## Project Structure

- \`contracts/\` – Solidity smart contracts (Hardhat)
- \`frontend/\` – Vite + React frontend
- \`backend/\` – Express-ready backend scaffold

## Setup

1. Clone the repo

2. Install dependencies for each part:

### Contracts (Hardhat + Solidity)

\`\`\`
cd contracts
npm install
npx hardhat test
\`\`\`

### Frontend (Vite + React + TypeScript)

\`\`\`
cd frontend
npm install
npm run dev
\`\`\`

### Backend (Express Scaffold)

\`\`\`
cd backend
npm install
npm run dev
\`\`\`

## License

MIT
EOF

# Contracts (Hardhat)
mkdir -p contracts/contracts
cat > contracts/package.json <<EOF
{
  "name": "basevault-contracts",
  "version": "1.0.0",
  "description": "BaseVault staking smart contracts (Hardhat, Solidity)",
  "scripts": {
    "test": "hardhat test",
    "compile": "hardhat compile"
  },
  "devDependencies": {
    "@nomicfoundation/hardhat-toolbox": "^3.0.0",
    "hardhat": "^3.0.0"
  }
}
EOF

cat > contracts/hardhat.config.js <<EOF
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.20",
  networks: {
    hardhat: {},
    // Add Base or Ethereum config here if needed
  },
};
EOF

cat > contracts/contracts/BaseVault.sol <<EOF
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BaseVault {
    // Basic structure for staking contract

    IERC20 public vaultToken;
    address public owner;

    mapping(address => uint256) public stakes;
    mapping(address => uint256) public rewards;

    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount);
    event RewardPaid(address indexed user, uint256 reward);

    constructor(address _vaultToken) {
        vaultToken = IERC20(_vaultToken);
        owner = msg.sender;
    }

    function stake() external payable {
        require(msg.value > 0, "Stake must be greater than 0");
        stakes[msg.sender] += msg.value;
        // Example logic: In production, add time-based/percentage accrual
        emit Staked(msg.sender, msg.value);
    }

    function unstake(uint256 amount) external {
        require(stakes[msg.sender] >= amount, "Not enough staked");
        stakes[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
        emit Unstaked(msg.sender, amount);
    }

    function claimReward() external {
        uint256 reward = calculateReward(msg.sender);
        require(reward > 0, "No rewards");
        rewards[msg.sender] = 0;
        vaultToken.transfer(msg.sender, reward);
        emit RewardPaid(msg.sender, reward);
    }

    function calculateReward(address staker) public view returns (uint256) {
        // Placeholder logic: real logic would use stake amount, duration, rate
        return stakes[staker] / 100; // 1% reward for demonstration
    }
}

interface IERC20 {
    function transfer(address to, uint256 value) external returns (bool);
}
EOF

# Frontend (Vite + React + TS)
mkdir -p frontend/src
cat > frontend/package.json <<EOF
{
  "name": "basevault-frontend",
  "version": "1.0.0",
  "description": "BaseVault frontend (Vite, React, TypeScript)",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.1.0",
    "typescript": "^5.0.0",
    "vite": "^4.0.0"
  }
}
EOF

cat > frontend/vite.config.ts <<EOF
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
EOF

cat > frontend/tsconfig.json <<EOF
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": false,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
EOF

cat > frontend/index.html <<EOF
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>BaseVault</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
EOF

cat > frontend/src/main.tsx <<EOF
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
EOF

cat > frontend/src/App.tsx <<EOF
import React from 'react';

function App() {
  return (
    <div style={{ maxWidth: 600, margin: '2rem auto', fontFamily: 'sans-serif' }}>
      <h1>BaseVault Staking DApp</h1>
      <p>
        Stake your Base ETH and earn VaultToken rewards!
      </p>
      {/* In production, connect wallet, show stake/unstake/claim UI */}
      <em>Frontend coming soon...</em>
    </div>
  );
}

export default App;
EOF

# Backend (Express)
mkdir -p backend
cat > backend/package.json <<EOF
{
  "name": "basevault-backend",
  "version": "1.0.0",
  "description": "Backend scaffold for BaseVault (Express-ready)",
  "scripts": {
    "dev": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

cat > backend/index.js <<EOF
const express = require('express');
const app = express();
const PORT = process.env.PORT || 4000;

app.get('/', (req, res) => {
  res.send('BaseVault backend API is running!');
});

app.listen(PORT, () => {
  console.log(\`BaseVault backend listening at http://localhost:\${PORT}\`);
});
EOF

echo "✅ BaseVault project scaffolded!"
echo "Next steps:"
echo "1. Run 'npm install' in contracts/, frontend/, and backend/"
echo "2. Start building your staking platform!"
