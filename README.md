# User Management Backend

Node.js backend service.

## Local Development
```bash
npm install
npm start
```

## CI/CD Flow
Push code → Jenkins builds image → pushes to DockerHub → updates gitops-environments → ArgoCD deploys
